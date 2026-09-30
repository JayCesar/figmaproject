type -a claude
ls -l "$(command -v claude)"
readlink -f "$(command -v claude)"
mount | grep -E "$(df --output=target "$(readlink -f "$(command -v claude)")" | tail -1)"



