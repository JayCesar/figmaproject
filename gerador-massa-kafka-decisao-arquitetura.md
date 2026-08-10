# Gerador de Massa — Decisão de Arquitetura

> Ferramenta interna para criação de pedidos nas jornadas Kafka, substituindo o uso de Insomnia e scripts locais.

---

## 1. Contexto

### Situação atual

- Existe uma API que, no POST, converte um corpo JSON específico e publica em tópico Kafka.
- O contrato do Kafka vive no **RDS Aurora**, consultado pela aplicação `core-v2` ao receber as mensagens.
- Vários times/negócios usam o Kafka; a API acabou virando o caminho de teste e geração de massa.
- Cada negócio exige um **application id** que autoriza escrita na jornada daquele negócio.
- O appid é obtido indo na base ou batendo numa API que retorna os ids cadastrados por ambiente e **tipo de leitura (1, 2, 3)** — o tipo de escrita é o **3**.
- Geração de massa migrou do Insomnia para scripts Python locais, porque cada negócio tem dados específicos.

### Aplicações envolvidas

| Aplicação | Endpoints usados |
|---|---|
| **App A** — criação de pedidos | `GET /appids` (resolve appid) + `POST /pedidos` (cria) |
| **App B** — consulta de pedidos | `GET /pedidos/{id}` (confirma criação) |

São **2 hosts, 3 chamadas**. O appid estar na mesma aplicação do POST simplifica: mesmo host, mesma autenticação, mesmo caminho de rede, mesmo cliente HTTP.

### Objetivo

Front onde o usuário seleciona a jornada/negócio, vê os campos necessários listados, preenche e envia — sem depender de Insomnia.

**Público inclui pessoas não técnicas.** Isso elimina a opção CLI e torna autenticação e auditoria obrigatórias.

---

## 2. Decisão

### ✅ ECS Fargate

Um serviço só, Kotlin, servindo API + estático do front.

**Os três fatores que decidem:**

1. **Cold start × uso esporádico.** Público não-técnico, poucas gerações por dia. Em Lambda, uso esporádico significa que quase toda invocação é fria — o pior caso vira o caso normal.
2. **O cache do appid.** Em Lambda o cache vive no escopo global do container e só sobrevive enquanto ele está quente. Container quase sempre frio = cache quase nunca acerta. Em ECS, processo longo, `Caffeine` com TTL funciona sem infra extra.
3. **Caminho pavimentado.** O que a squad já roda define o prazo de entrega mais que qualquer trade-off teórico.

O custo mensal **não decide nada** — a diferença de ~$38 não paga uma hora de trabalho.

### Onde Lambda ganharia

Se você desistisse do cache (defensável — o appid vem do mesmo host do POST, custa milissegundos), aceitasse 1–2s de cold start com SnapStart, e a squad já tivesse API Gateway + Lambda pavimentado. Aí a entrega é mais rápida e o custo é quase zero.

### Descartados

**App Runner / EKS** — complexidade sem retorno nesse escopo.

**Enfiar no serviço de pedidos existente** — acopla geração de massa de teste a um serviço produtivo, e o appid de escrita passa a viver num lugar com blast radius muito maior.

**CLI** — resolveria se o público fosse só a squad. Não resolve para gente não-técnica.

---

## 3. Prós e Contras Completos

### ECS Fargate

**Prós**

| Ponto | Detalhe |
|---|---|
| Sem cold start | Latência constante — importa com público que não entende "às vezes demora" |
| Cache do appid funciona | Processo longo, `Caffeine` com TTL em memória, zero infra extra |
| Keep-alive | Appid e criação são o mesmo host: conexão aberta serve as duas chamadas seguidas |
| Um artefato só | API + estático no mesmo container, um deploy, sem CORS |
| Timeout sob controle | ALB configurável, sem teto de 29s do API Gateway |
| Cache do OpenAPI | Busca o spec uma vez no startup em vez de a cada requisição |
| Paridade local total | Mesmo container roda na máquina — relevante no RHEL corporativo sem sudo |
| Rede por cópia | Herda VPC, SG, IAM, pipeline e observabilidade de outro serviço da squad |
| Sessão trivial | Autenticação com estado em memória, sem store externo |
| Escopo pode crescer | Lote, SSE de progresso, geração longa — sem travar |
| Métricas contínuas | Dashboard e alerta com processo vivo, sem correlacionar invocações soltas |

**Contras**

| Ponto | Detalhe |
|---|---|
| Custo idle real | ~$10/task + ~$18 ALB. Com 2 tasks pra HA, ~$38/mês |
| Superdimensionado | Capacidade parada 24/7 pra atender picos de minutos |
| Manutenção | Imagem base, patch de CVE, health check, autoscaling, task definition |
| Deploy/rollback | Mais lentos que alias de Lambda |
| HA custa dobrado | Uma task só = indisponibilidade em cada deploy e em cada falha |
| Vazamento de memória | Processo longo acumula; Lambda mata o container e limpa sozinho |
| Escala por task | Pico simultâneo depende do autoscaling reagir |

### Lambda

**Prós**

| Ponto | Detalhe |
|---|---|
| Custo idle ~zero | Casa perfeitamente com uso esporádico |
| Zero capacidade | Sem imagem, sem autoscaling, sem health check |
| Absorve pico | Vários times disparando junto não exige nada de você |
| Rollback imediato | Por versão/alias |
| Formato nativo | Orquestrador stateless de HTTP é literalmente o caso de uso |
| Isolamento por invocação | Bug de estado compartilhado não existe |
| Nunca toca no Aurora | O problema clássico de pool com RDS não se aplica ao desenho |
| Blast radius pequeno | Função com permissão mínima, fácil auditar quem chama o quê |

**Contras**

| Ponto | Detalhe |
|---|---|
| **Cold start** | Kotlin/JVM: 3–8s, ou 1–2s com SnapStart. Uso esporádico = quase toda invocação fria |
| **Cache não funciona** | Escopo global do container; container frio = cache não acerta. Cache real exige DynamoDB/ElastiCache — e some a simplicidade |
| Armadilha do SnapStart | O que for inicializado antes do snapshot (conexão, credencial, seed aleatório) é restaurado idêntico em todos os containers |
| Sem keep-alive | Cada container novo refaz TLS com a App A — que é chamada duas vezes seguidas |
| Teto de 29s | API Gateway. Contornável com ALB, mas aí paga o ALB e perde a vantagem de custo |
| Front separado | S3 + CloudFront: dois artefatos, dois deploys, CORS pra configurar |
| VPC obrigatória | Subnets, SG, ENI pra alcançar APIs internas — some parte da simplicidade prometida |
| OpenAPI recarregado | Duas chamadas extras antes de fazer qualquer coisa útil, toda vez fria |
| Debug mais chato | Sem execução local fiel; correlacionar 3 chamadas entre invocações dá trabalho |
| Sessão externa | Autenticação obrigatória exige store adicional |

---

## 4. Arquitetura

```
[React SPA] ──► [ALB] ──► [ECS Fargate: Kotlin]
                              │
                              ├──► App A: GET /appids      (resolve, filtra tipo=3)
                              ├──► App A: POST /pedidos    (cria)
                              └──► App B: GET /pedidos/{id} (confirma)

GET  /negocios                        → lista negócios
GET  /negocios/{id}/schema            → campos + defaults + labels
POST /negocios/{id}/pedidos           → {ambiente, campos, extras}
GET  /negocios/{id}/pedidos/{pedidoId} → PENDING | CONFIRMED | NOT_FOUND
```

O front vê **2 endpoints seus** e nunca sabe que existem 3 chamadas por baixo.

### Fluxo do envio

```
Front                    Seu backend                  Apps da squad
  │
  │ GET /negocios/{id}/schema
  ├────────────────────────►  (OpenAPI + overlay)
  │ ◄── campos + labels + defaults
  │
  │ POST /negocios/{id}/pedidos
  │   {ambiente, campos, extras}
  ├────────────────────────►  1. GET appids?ambiente=X ──► App A
  │                            2. filtra tipo=3 (falha se ≠1)
  │                            3. gera chave do pedido
  │                            4. POST payload + appid   ──► App A
  │ ◄── {pedidoId, status:SENT}
  │
  │ GET /negocios/{id}/pedidos/{pedidoId}
  ├────────────────────────►  GET consulta ──────────────► App B
  │ ◄── PENDING | CONFIRMED | NOT_FOUND
```

**O serviço nunca toca no Aurora** — nem agora, nem quando a API de negócio existir. É orquestrador HTTP puro. Isso mantém a porta aberta pra migrar pra Lambda depois, se o padrão de uso mudar.

---

## 5. Armadilhas do Desenho

### 5.1 O GET vai falhar na primeira tentativa

O caminho é `POST → Kafka → core-v2 → Aurora`. É assíncrono. Se o usuário verificar 200ms depois do POST, recebe "não existe" e conclui que quebrou.

Precisa de **três estados, não dois**:

- `AINDA_PROCESSANDO`
- `CONFIRMADO`
- `NÃO_ENCONTRADO_APÓS_TIMEOUT`

Sem isso a ferramenta gera falso negativo e as pessoas voltam pro Insomnia.

### 5.2 Filtrar por tipo 3 com `.first()` é bomba-relógio

Se a API devolver zero do tipo 3 pra aquele negócio? Se devolver dois? Escolher silenciosamente significa escrever na jornada errada.

**Falha explícita > escolha implícita.**

### 5.3 Hardcoded ≠ `when (negocio)`

Hardcode como **dado** (JSON no resources), no mesmo formato que a API futura vai devolver. Trocar depois vira substituir uma implementação de `SchemaRepository`. Se hardcodar como `if/else` no código, migrar depois é reescrever.

### 5.4 A chave do pedido quem gera é o backend

Não o usuário, não a App A. Você devolve no POST e o GET fica determinístico. Se depender de um id que a App A devolve, fica refém do contrato dela.

### 5.5 Idempotência

O usuário vai clicar duas vezes em "Enviar". Aceite um `Idempotency-Key` do front (uuid gerado no clique) e guarde `key → pedidoId` por alguns minutos. Sem isso, massa duplicada e uma hora de debug.

### 5.6 Autenticação e auditoria (público não-técnico)

- **Auth corporativo obrigatório**, com controle de qual ambiente cada pessoa pode disparar. Sem isso, você criou um caminho de escrita em jornada de negócio sem dono.
- **O appid nunca trafega pro browser.** Front manda `{negocio, ambiente}`; backend resolve server-side.
- **Auditoria:** registre quem gerou, qual negócio, qual ambiente, quantos pedidos. Quando perguntarem "de onde veio essa massa", você responde em vez de investigar.

---

## 6. Schema dos Campos

### Estratégia recomendada: híbrida

| Fonte | O que fornece |
|---|---|
| **OpenAPI/Swagger das APIs** | Estrutura, tipos, campos obrigatórios — automático, acompanha mudança de contrato |
| **Overlay por negócio** (JSON ~20 linhas) | Labels em português, valores default, quais campos esconder |

OpenAPI dá a estrutura técnica, não a semântica de negócio. Público não-técnico não sabe o que preencher em `cdIdentPedido` — daí o overlay.

**O overlay é o que vira a chamada na base no futuro.** Mesmo formato, fonte diferente.

Isso é melhor que hardcode puro: quando o contrato mudar, o front acompanha sozinho em vez de você descobrir pelo 400 de alguém.

---

## 7. Código

```kotlin
// Schema como dado, não como branch
interface SchemaRepository { fun byNegocio(id: String): NegocioSchema }

class LocalSchemaRepository : SchemaRepository {   // MVP
    private val cache = loadFromResources("/schemas")
    override fun byNegocio(id: String) = cache[id] ?: throw NegocioNotFound(id)
}
// depois: class ApiSchemaRepository(...) : SchemaRepository — só troca o bind
```

```kotlin
// Resolução do appid com falha explícita
class AppIdResolver(private val client: AppIdClient) {
    private val cache = Caffeine.newBuilder()
        .expireAfterWrite(Duration.ofMinutes(10)).build<Key, String>()

    fun resolve(negocio: String, ambiente: String): String =
        cache.get(Key(negocio, ambiente)) {
            val candidatos = client.list(negocio, ambiente)
                .filter { it.tipo == TIPO_ESCRITA }

            when (candidatos.size) {
                1    -> candidatos.single().appId
                0    -> throw NoWriteAppId(negocio, ambiente)
                else -> throw AmbiguousAppId(negocio, ambiente, candidatos.map { it.appId })
            }
        }
}
```

`TIPO_ESCRITA = 3` vem de **config**, não de literal no meio do fluxo — se algum negócio usar outro tipo, sem redeploy.

> **Sobre o cache:** no MVP, considere **cortar**. O appid vem do mesmo host do POST — uma chamada a mais numa conexão já aberta custa milissegundos, e você elimina uma classe inteira de bug de invalidação. Adicione quando medir que dói.

---

## 8. Próximos Passos

### Perguntas que mudam o desenho

1. **Quanto tempo entre o POST e o pedido aparecer no GET?**
   Meça manualmente 5 vezes.
   - `< 2s` → polling de 500ms por 15s + spinner
   - `30s+` → botão "verificar" manual; polling seria over-engineering

2. **As APIs da squad expõem OpenAPI?**
   - Sim → gere o formulário do spec, escreva só o overlay
   - Não → JSON de schema por negócio, começando por um piloto

3. **A squad já tem serviço Kotlin em ECS com pipeline pronto?**
   - Sim → copie o esqueleto: rede, IAM e observabilidade vêm resolvidos
   - Não → reavalie Lambda + SnapStart; o custo de setup passa a dominar

### Ordem de implementação

1. `SchemaRepository` local + `GET /negocios/{id}/schema`
2. `POST /negocios/{id}/pedidos` com o resolver de appid
3. `GET` de confirmação

> **Os dois primeiros já substituem os scripts Python. O terceiro é conveniência.**

### Não faça agora

- SQS + worker → só quando lote virar dor real
- Cache do appid → só quando medir latência
- Autoscaling elaborado → uma task resolve o volume inicial
