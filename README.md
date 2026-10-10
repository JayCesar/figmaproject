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


