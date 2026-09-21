mv ~/.local/bin/claude.bak ~/.local/bin/claude
ls -l ~/.local/bin/claude
readlink -f ~/.local/bin/claude
ls -la ~/.local/lib/node_modules/@anthropic-ai/claude-code/


chmod +x "$(readlink -f ~/.local/bin/claude)"
hash -r
claude --version
