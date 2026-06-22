
chmod +x ~/.local/share/applications/flameshot/Flameshot-12.1.0.x86_64.AppImage




mkdir -p ~/.local/bin
ln -sf ~/.local/share/applications/flameshot/Flameshot-12.1.0.x86_64.AppImage ~/.local/bin/flameshot

which flameshot      # esperado: /home/jccbzlf/.local/bin/flameshot
flameshot gui        # esperado: abre o overlay de captura

gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings \
  "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/flameshot/']"

BASE="org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/flameshot/"

gsettings set "$BASE" name 'Flameshot'
gsettings set "$BASE" command 'flameshot gui'
gsettings set "$BASE" binding 'Print'

