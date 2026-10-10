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

# Terminator: trocar as cores

São três ajustes, cada um num lugar: a paleta de cores do texto (igual à do terminal nativo), a barra de título dos painéis e a moldura da janela.

Todos os comandos que editam a configuração devem ser rodados pelo terminal nativo, com o Terminator fechado, senão ele sobrescreve o arquivo ao sair.

## 1. Paleta de cores (igual ao terminal nativo)

Conferir qual paleta o terminal nativo usa (rodar nele):

```zsh
uuid=$(gsettings get org.gnome.Ptyxis default-profile-uuid | tr -d "'")
gsettings get org.gnome.Ptyxis.Profile:/org/gnome/Ptyxis/Profiles/$uuid/ palette
```

Fechar o Terminator:

```zsh
pkill -f terminator
```

Fazer backup da configuração:

```zsh
cp ~/.config/terminator/config ~/.config/terminator/config.bak
```

Aplicar a paleta Dracula (pastas em lilás, caminho em ciano claro):

```zsh
sed -i 's|^\( *palette = \).*|\1"#21222c:#ff5555:#50fa7b:#f1fa8c:#bd93f9:#ff79c6:#8be9fd:#f8f8f2:#6272a4:#ff6e6e:#69ff94:#ffffa5:#d6acff:#ff92df:#a4ffff:#ffffff"|' ~/.config/terminator/config
```

Conferir:

```zsh
grep palette ~/.config/terminator/config
```

A paleta tem 16 cores separadas por `:`, na ordem preto, vermelho, verde, amarelo, azul, magenta, ciano, branco, e depois as mesmas oito em versão clara. O `ls` pinta as pastas com o "azul" (5ª cor) e o prompt usa o "ciano" (7ª cor).

## 2. Remover cores forçadas do `.zshrc`

Com a paleta certa, as cores de pasta definidas à mão deixam de ser necessárias. Fazer backup e apagar as linhas:

```zsh
cp ~/.zshrc ~/.zshrc.bak
```

```zsh
sed -i '/COR_PASTA\|EZA_COLORS\|LS_COLORS\|list-colors/d' ~/.zshrc
```

Abrir uma janela nova do Terminator e testar:

```zsh
ls
```

Precisa ser uma janela nova, porque as variáveis antigas continuam valendo nos terminais já abertos.

## 3. Barra de título dos painéis

Adicionar as cores na seção `[global_config]`:

```zsh
sed -i '/^\[global_config\]/a\  title_transmit_bg_color = "#282a36"\n  title_transmit_fg_color = "#f8f8f2"\n  title_inactive_bg_color = "#1e1f29"\n  title_inactive_fg_color = "#6272a4"' ~/.config/terminator/config
```

Conferir se as quatro linhas entraram (rodar o `sed` só uma vez, senão elas duplicam):

```zsh
grep title_ ~/.config/terminator/config
```

`transmit` é o painel em foco e `inactive` são os outros. `bg` é o fundo e `fg` é o texto.

## 4. Moldura da janela

Testar o tema escuro só no Terminator:

```zsh
GTK_THEME=Adwaita:dark terminator
```

Fixar para quando abrir pelo menu de aplicativos:

```zsh
mkdir -p ~/.local/share/applications
```

```zsh
cp /usr/share/applications/terminator.desktop ~/.local/share/applications/
```

```zsh
sed -i 's|^Exec=terminator|Exec=env GTK_THEME=Adwaita:dark terminator|' ~/.local/share/applications/terminator.desktop
```

Conferir (deve mostrar `Exec=env GTK_THEME=Adwaita:dark terminator`):

```zsh
grep '^Exec=' ~/.local/share/applications/terminator.desktop
```

Fixar para quando abrir pelo terminal, adicionando esta linha no `~/.zshrc`:

```zsh
alias terminator='GTK_THEME=Adwaita:dark terminator'
```

```zsh
source ~/.zshrc
```

## 5. Cor do caminho no prompt (Starship)

Opcional: fixa o ciano claro do caminho em qualquer terminal, independente da paleta. Adicionar no `~/.config/starship.toml`:

```toml
[directory]
style = "bold #8be9fd"
```

## 6. Desfazer

Voltar a paleta e as cores da barra de título:

```zsh
pkill -f terminator
```

```zsh
cp ~/.config/terminator/config.bak ~/.config/terminator/config
```

Voltar o `.zshrc`:

```zsh
cp ~/.zshrc.bak ~/.zshrc
```

Voltar a moldura ao tema padrão:

```zsh
rm ~/.local/share/applications/terminator.desktop
```
