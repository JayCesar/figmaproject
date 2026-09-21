
ls -la ~/.local/lib/node_modules/@anthropic-ai/claude-code/bin/
chmod +x ~/.local/lib/node_modules/@anthropic-ai/claude-code/bin/claude.exe
hash -r
claude --version

cd ~/.local/lib/node_modules/@anthropic-ai/claude-code
node install.cjs
chmod +x bin/claude.exe
