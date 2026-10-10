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

São duas barras, cada uma configurada num lugar: a barra de título dos painéis (do Terminator) e a moldura da janela (do tema GTK).

## 1. Barra de título dos painéis

Fechar o Terminator antes de editar, senão ele sobrescreve o arquivo ao sair:

```zsh
pkill -f terminator
```

Fazer backup da configuração:

```zsh
cp ~/.config/terminator/config ~/.config/terminator/config.bak
```

Adicionar as cores na seção `[global_config]`:

```zsh
sed -i '/^\[global_config\]/a\  title_transmit_bg_color = "#282a36"\n  title_transmit_fg_color = "#f8f8f2"\n  title_inactive_bg_color = "#1e1f29"\n  title_inactive_fg_color = "#6272a4"' ~/.config/terminator/config
```

Conferir se as quatro linhas entraram:

```zsh
grep title_ ~/.config/terminator/config
```

`transmit` é o painel em foco e `inactive` são os outros. `bg` é o fundo e `fg` é o texto. Para mudar uma cor depois, editar o valor hexadecimal:

```zsh
nano ~/.config/terminator/config
```

## 2. Moldura da janela

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

## 3. Desfazer

Voltar as cores da barra de título:

```zsh
pkill -f terminator
```

```zsh
cp ~/.config/terminator/config.bak ~/.config/terminator/config
```

Voltar a moldura ao tema padrão:

```zsh
rm ~/.local/share/applications/terminator.desktop
```
