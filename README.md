ls ~/Android/Sdk/system-images/ 2>/dev/null && echo "=== TEM IMAGEM ==="
ls ~/Android/Sdk/emulator/ 2>/dev/null && echo "=== TEM EMULATOR DO SDK ==="
~/Android/Sdk/emulator/emulator -list-avds 2>/dev/null
