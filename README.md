## Comando para p zsh

```sh
LD_LIBRARY_PATH=/home/jccbzlf/settings/softwares/libs/usr/lib64 /home/jccbzlf/settings/softwares/anki/anki-26.9.3-linux-x86_64.tar/anki

## Anki
anki() {
  LD_LIBRARY_PATH=/home/jccbzlf/settings/softwares/libs/usr/lib64 nohup /home/jccbzlf/settings/softwares/anki/anki-26.9.3-linux-x86_64.tar/anki "$@" >/dev/null 2>&1 &!
}

```


autoload -Uz compinit && compinit
zstyle ':completion:*' menu select
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'
zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
bindkey '^[[Z' reverse-menu-complete    # Shift+TAB volta no menu
