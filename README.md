## Comando para p zsh

```sh
LD_LIBRARY_PATH=/home/jccbzlf/settings/softwares/libs/usr/lib64 /home/jccbzlf/settings/softwares/anki/anki-26.9.3-linux-x86_64.tar/anki

## Anki
anki() {
  LD_LIBRARY_PATH=/home/jccbzlf/settings/softwares/libs/usr/lib64 nohup /home/jccbzlf/settings/softwares/anki/anki-26.9.3-linux-x86_64.tar/anki "$@" >/dev/null 2>&1 &!
}

watch -n 1 -d 'uptime; echo; free -h; echo; df -h / /home'

javascript:(function(){var s=document.documentElement.style;s.filter=s.filter?'':'invert(1) hue-rotate(180deg)';})()

```


## Mudar opactidade:

```sh
base=org.gnome.settings-daemon.plugins.media-keys
caminho=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/flameshot/

gsettings set $base custom-keybindings "['$caminho']"
gsettings set $base.custom-keybinding:$caminho name 'Flameshot'
gsettings set $base.custom-keybinding:$caminho command 'flameshot gui'
gsettings set $base.custom-keybinding:$caminho binding '<Super><Shift>s'
```

## Flaemshot (para conficuar o ctrl +c)

```sh
   mkdir -p ~/.config/flameshot
   print -l '[Shortcuts]' 'TYPE_ACCEPT=Ctrl+C' 'TYPE_COPY=Ctrl+Shift+C' >> ~/.config/flameshot/flameshot.ini
```

## Mudar pasta de Downloads
```
brave://settings/downloads
```
# Terminator: documentação

Guia do Terminator no Fedora (GNOME, zsh): instalação, uso, configuração, cores e solução de problemas.

## 1. Instalação

```zsh
sudo dnf install terminator
```

Abrir:

```zsh
terminator
```

Ver a versão:

```zsh
terminator --version
```

## 2. Atalhos de teclado

### Dividir e navegar

| Ação | Atalho |
|---|---|
| Dividir em cima/embaixo | `Ctrl+Shift+O` |
| Dividir lado a lado | `Ctrl+Shift+E` |
| Mudar de painel | `Alt+setas` |
| Próximo / painel anterior | `Ctrl+Shift+N` / `Ctrl+Shift+P` |
| Redimensionar o painel | `Ctrl+Shift+setas` |
| Maximizar o painel atual (alterna) | `Ctrl+Shift+X` |
| Maximizar e aumentar a fonte (alterna) | `Ctrl+Shift+Z` |
| Fechar o painel | `Ctrl+Shift+W` |

### Abas e janelas

| Ação | Atalho |
|---|---|
| Nova aba | `Ctrl+Shift+T` |
| Aba seguinte / anterior | `Ctrl+PageDown` / `Ctrl+PageUp` |
| Nova janela | `Ctrl+Shift+I` |
| Tela cheia | `F11` |
| Fechar tudo | `Ctrl+Shift+Q` |

### Texto

| Ação | Atalho |
|---|---|
| Copiar / colar | `Ctrl+Shift+C` / `Ctrl+Shift+V` |
| Buscar no histórico da tela | `Ctrl+Shift+F` |
| Aumentar / diminuir / restaurar a fonte | `Ctrl++` / `Ctrl+-` / `Ctrl+0` |
| Limpar a tela e o histórico | `Ctrl+Shift+G` |

### Digitar em vários painéis ao mesmo tempo

| Ação | Atalho |
|---|---|
| Enviar o que digitar para todos os painéis | `Alt+A` |
| Desligar o envio | `Alt+O` |

## 3. Opções de linha de comando

Abrir maximizado:

```zsh
terminator -m
```

Abrir em tela cheia:

```zsh
terminator -f
```

Abrir sem a moldura da janela:

```zsh
terminator -b
```

Abrir numa pasta específica:

```zsh
terminator --working-directory=~/development
```

Abrir um layout salvo:

```zsh
terminator -l nome-do-layout
```

## 4. Preferências

Botão direito dentro do terminal e depois "Preferências".

- **Global**: comportamento da janela e cores da barra de título dos painéis.
- **Perfis**: fonte, cores, transparência, rolagem e cursor.
- **Layouts**: arranjos de painéis salvos.
- **Atalhos**: todas as combinações de teclas.
- **Plugins**: extras opcionais.

## 5. Arquivo de configuração

Local:

```zsh
cat ~/.config/terminator/config
```

Regra principal: o Terminator regrava esse arquivo ao fechar as Preferências e ao sair. Para editar à mão, fechar todas as janelas antes e usar outro terminal.

```zsh
pkill -f terminator
```

```zsh
cp ~/.config/terminator/config ~/.config/terminator/config.bak
```

```zsh
nano ~/.config/terminator/config
```

### Configuração completa de referência

```
[global_config]
  title_transmit_bg_color = "#282a36"
  title_transmit_fg_color = "#f8f8f2"
  title_inactive_bg_color = "#1e1f29"
  title_inactive_fg_color = "#6272a4"
[keybindings]
[profiles]
  [[default]]
    background_darkness = 0.83
    background_type = image
    scroll_on_output = True
    scrollback_infinite = True
    palette = "#21222c:#ff5555:#50fa7b:#f1fa8c:#bd93f9:#ff79c6:#8be9fd:#f8f8f2:#6272a4:#ff6e6e:#69ff94:#ffffa5:#d6acff:#ff92df:#a4ffff:#ffffff"
[layouts]
  [[default]]
    [[[window0]]]
      type = Window
      parent = ""
    [[[child1]]]
      type = Terminal
      parent = window0
[plugins]
```

O que cada seção guarda:

- **`[global_config]`**: opções gerais e cores da barra de título.
- **`[keybindings]`**: atalhos alterados (vazio = todos no padrão).
- **`[profiles]`**: aparência do terminal. O perfil `default` é o usado ao abrir.
- **`[layouts]`**: arranjos de painéis salvos.
- **`[plugins]`**: configuração dos plugins.

## 6. Cores

### Paleta (igual ao terminal nativo)

Conferir a paleta do terminal nativo (rodar nele):

```zsh
uuid=$(gsettings get org.gnome.Ptyxis default-profile-uuid | tr -d "'")
gsettings get org.gnome.Ptyxis.Profile:/org/gnome/Ptyxis/Profiles/$uuid/ palette
```

Aplicar a paleta Dracula no Terminator:

```zsh
sed -i 's|^\( *palette = \).*|\1"#21222c:#ff5555:#50fa7b:#f1fa8c:#bd93f9:#ff79c6:#8be9fd:#f8f8f2:#6272a4:#ff6e6e:#69ff94:#ffffa5:#d6acff:#ff92df:#a4ffff:#ffffff"|' ~/.config/terminator/config
```

A paleta tem 16 cores separadas por `:`, na ordem preto, vermelho, verde, amarelo, azul, magenta, ciano, branco, e depois as mesmas oito em versão clara. O `ls` pinta as pastas com o "azul" (5ª cor) e o prompt usa o "ciano" (7ª cor).

Com a paleta certa, remover as cores de pasta forçadas no `.zshrc`:

```zsh
cp ~/.zshrc ~/.zshrc.bak
```

```zsh
sed -i '/COR_PASTA\|EZA_COLORS\|LS_COLORS\|list-colors/d' ~/.zshrc
```

### Barra de título dos painéis

Rodar uma vez só, senão as linhas duplicam:

```zsh
sed -i '/^\[global_config\]/a\  title_transmit_bg_color = "#282a36"\n  title_transmit_fg_color = "#f8f8f2"\n  title_inactive_bg_color = "#1e1f29"\n  title_inactive_fg_color = "#6272a4"' ~/.config/terminator/config
```

```zsh
grep title_ ~/.config/terminator/config
```

`transmit` é o painel em foco e `inactive` são os outros. `bg` é o fundo e `fg` é o texto.

Para esconder a barra, adicionar no perfil `[[default]]`:

```
    show_titlebar = False
```

### Moldura da janela (tema escuro)

Testar:

```zsh
GTK_THEME=Adwaita:dark terminator
```

Fixar para o menu de aplicativos:

```zsh
mkdir -p ~/.local/share/applications
```

```zsh
cp /usr/share/applications/terminator.desktop ~/.local/share/applications/
```

```zsh
sed -i 's|^Exec=terminator|Exec=env GTK_THEME=Adwaita:dark terminator|' ~/.local/share/applications/terminator.desktop
```

```zsh
grep '^Exec=' ~/.local/share/applications/terminator.desktop
```

Fixar para o terminal, com esta linha no `~/.zshrc`:

```zsh
alias terminator='GTK_THEME=Adwaita:dark terminator'
```

## 7. Transparência e fonte

Ficam no perfil `[[default]]`.

Transparência (`1.0` é opaco, valores menores são mais transparentes):

```
    background_darkness = 0.83
    background_type = transparent
```

A configuração atual usa `background_type = image` sem imagem definida e o fundo aparece translúcido. Se a transparência sumir, trocar para `transparent`.

Fonte própria em vez da fonte do sistema:

```
    use_system_font = False
    font = JetBrains Mono 12
```

Listar as fontes monoespaçadas instaladas:

```zsh
fc-list :spacing=mono family | sort -u
```

## 8. Atalhos personalizados

Pelas Preferências, na aba de atalhos: clicar na ação e apertar a nova combinação.

Por arquivo, na seção `[keybindings]`. Exemplo com `Alt+D` e `Alt+O` para dividir:

```
[keybindings]
  split_vert = <Alt>d
  split_horiz = <Alt>o
```

`split_vert` divide lado a lado e `split_horiz` divide em cima/embaixo. Evitar `Ctrl+D` (fecha o shell) e `Ctrl+O` (salvar do nano).

## 9. Layouts

Salvam um arranjo de painéis para abrir pronto.

1. Dividir a janela do jeito desejado.
2. Abrir as Preferências, aba Layouts, clicar em adicionar e dar um nome.
3. Abrir com o layout:

```zsh
terminator -l nome-do-layout
```

## 10. Solução de problemas

**As mudanças no arquivo somem.** O Terminator regravou o arquivo ao sair. Fechar tudo antes de editar:

```zsh
pkill -f terminator
```

**Erro ao abrir depois de editar.** Costuma ser chave duplicada na mesma seção. Procurar repetições:

```zsh
sort ~/.config/terminator/config | uniq -d
```

**Voltar ao backup:**

```zsh
cp ~/.config/terminator/config.bak ~/.config/terminator/config
```

**Voltar à configuração de fábrica** (o Terminator recria o arquivo ao abrir):

```zsh
mv ~/.config/terminator/config ~/.config/terminator/config.velho
```

**Moldura da janela continua branca.** Conferir se o atalho do menu tem o tema:

```zsh
grep '^Exec=' ~/.local/share/applications/terminator.desktop
```

**Cores de pasta diferentes das esperadas.** Conferir se sobrou cor forçada no shell (deve vir vazio):

```zsh
echo "$EZA_COLORS $LS_COLORS"
```

**Ícones do `ls` como quadradinhos.** A fonte não tem os símbolos. Instalar uma Nerd Font e selecioná-la no perfil, ou tirar o `--icons` dos aliases do `eza`.
