#!/bin/bash
#
# listar.sh - Lista CSVs de negócio (agro.csv, seguros.csv, etc) em duas tabelas:
#   1) Dados do grupo/proposta (grupo_proprietario, id_grupo, template, id_template)
#   2) Etapas (ordem, nome_etapa, id_etapa)
#
# Uso direto:
#   ./listar.sh agro                -> mostra as duas tabelas do agro
#   ./listar.sh agro -f "Agro"      -> filtra por texto no grupo_proprietario
#   ./listar.sh all                 -> junta todos os negócios (com coluna "negocio")
#   ./listar.sh listar              -> lista negócios disponíveis
#
# Uso interativo (sem argumentos):
#   ./listar.sh
#   -> mostra um menu pra escolher o negócio
#
# Requisitos: csvkit (pip install --user csvkit)

set -e

DATA_DIR="./dados"

GREEN='\033[0;32m'
CYAN='\033[0;36m'
YELLOW='\033[1;33m'
BOLD='\033[1m'
RESET='\033[0m'

# ------------------------------------------------------------------
# Funções
# ------------------------------------------------------------------

listar_negocios() {
    echo -e "${CYAN}Negócios disponíveis em ${DATA_DIR}:${RESET}"
    local i=1
    ARQS=()
    for f in "$DATA_DIR"/*.csv; do
        [ -e "$f" ] || continue
        base=$(basename "$f" .csv)
        n=$(($(wc -l < "$f") - 1))
        echo -e "  ${GREEN}${i})${RESET} ${base}  (${n} etapas)"
        ARQS+=("$base")
        i=$((i+1))
    done
}

# Mostra a tabela 1: dados do grupo/proposta (uma linha por negócio)
tabela_grupo() {
    local arquivo="$1"
    local extra_col="$2"   # "negocio," se modo all, senão vazio
    echo -e "${BOLD}${YELLOW}▸ Dados do Grupo / Proposta${RESET}"
    csvcut -c "${extra_col}grupo_proprietario,id_grupo,template,id_template" "$arquivo" \
        | (IFS= read -r h; echo "$h"; sort -u) \
        | csvlook
    echo ""
}

# Mostra a tabela 2: etapas
tabela_etapas() {
    local arquivo="$1"
    local extra_col="$2"
    echo -e "${BOLD}${YELLOW}▸ Etapas${RESET}"
    csvcut -c "${extra_col}ordem,nome_etapa,id_etapa" "$arquivo" | csvsort -c ordem | csvlook
    echo ""
}

processar() {
    local nome="$1"
    local filtro="$2"
    local arquivo
    local extra_col=""

    if [ "$nome" == "all" ]; then
        TMP=$(mktemp)
        primeiro=1
        for f in "$DATA_DIR"/*.csv; do
            [ -e "$f" ] || continue
            negocio=$(basename "$f" .csv)
            if [ "$primeiro" -eq 1 ]; then
                header=$(head -n1 "$f")
                echo "negocio,${header}" > "$TMP"
                primeiro=0
            fi
            tail -n +2 "$f" | sed "s/^/${negocio},/" >> "$TMP"
        done
        arquivo="$TMP"
        extra_col="negocio,"
    else
        if [ -f "${DATA_DIR}/${nome}.csv" ]; then
            arquivo="${DATA_DIR}/${nome}.csv"
        elif [ -f "${nome}.csv" ]; then
            arquivo="${nome}.csv"
        else
            echo "Arquivo não encontrado: ${nome}.csv"
            echo "Use '$0 listar' para ver os negócios disponíveis."
            exit 1
        fi
    fi

    if [ -n "$filtro" ]; then
        FTMP=$(mktemp)
        csvgrep -c grupo_proprietario -m "$filtro" "$arquivo" > "$FTMP"
        arquivo="$FTMP"
    fi

    echo -e "${CYAN}Fonte:${RESET} ${nome}"
    [ -n "$filtro" ] && echo -e "${CYAN}Filtro:${RESET} grupo_proprietario contém \"$filtro\""
    echo ""

    { tabela_grupo "$arquivo" "$extra_col"; tabela_etapas "$arquivo" "$extra_col"; } \
        | (command -v less >/dev/null && less -RS || cat)

    [ "$nome" == "all" ] && rm -f "$TMP"
    [ -n "$filtro" ] && rm -f "$FTMP"
}

# ------------------------------------------------------------------
# Modo interativo (sem argumentos)
# ------------------------------------------------------------------
if [ -z "$1" ]; then
    listar_negocios
    echo ""
    echo -e "${CYAN}Digite o número, o nome do negócio, ou 'all' para ver todos:${RESET}"
    read -r ESCOLHA

    if [[ "$ESCOLHA" =~ ^[0-9]+$ ]]; then
        idx=$((ESCOLHA-1))
        NOME="${ARQS[$idx]}"
    else
        NOME="$ESCOLHA"
    fi

    echo -e "${CYAN}Filtrar por texto no grupo_proprietario? (Enter pra pular):${RESET}"
    read -r FILTRO

    processar "$NOME" "$FILTRO"
    exit 0
fi

# ------------------------------------------------------------------
# Modo direto (com argumentos)
# ------------------------------------------------------------------
NOME="$1"
shift

if [ "$NOME" == "listar" ]; then
    listar_negocios
    exit 0
fi

FILTRO=""
while [ "$1" != "" ]; do
    case "$1" in
        -f|--filter)
            FILTRO="$2"
            shift 2
            ;;
        *)
            echo "Opção desconhecida: $1"
            exit 1
            ;;
    esac
done

processar "$NOME" "$FILTRO"
