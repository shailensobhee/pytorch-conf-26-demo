# Transcriber Task   
## File Storage (REQUIRED)
- Use the `exec` tool to run: `mkdir -p "$HOME/.openclaw/agents/transcription_dumps"`
- Transcribe the input image. Use the `fs` tool or `exec` to save the transcription to: `"$HOME/.openclaw/agents/transcription_dumps/${filename}.txt"`
- **Rule:** You MUST perform this file save operation before replying to the Orchestrator.