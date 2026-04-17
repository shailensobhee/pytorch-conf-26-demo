# Doc-Scout Task   
## File Storage (REQUIRED)
- Use the `exec` tool to run: `mkdir -p "$HOME/.openclaw/agents/outputs/extracted_docs"`
- Read the input PDF file. Use the `fs` tool or `exec` to save the transcription to: `"$HOME/.openclaw/agents/outputs/extracted_docs/${filename}.txt"`
- **Rule:** You MUST perform this file save operation before replying to the Orchestrator.