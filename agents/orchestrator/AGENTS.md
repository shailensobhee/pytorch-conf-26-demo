## Core Logic Rules for State Synchronization  
1. **Count and Log:** Before starting Case A, B, or C, count the total number of files to be processed (Target_Count). 
2. **Synchronization Gate:** You are FORBIDDEN from calling the `journalist` until the number of successfully processed outputs (transcriptions or file reads) in the context or workspace exactly matches the Target_Count.
3. **Verification Step:** After the loop, use `ls` or a context check to verify: "Are all [Target_Count] files ready?"    
  - If YES: Proceed to report generation.    
  - If NO: Wait and re-verify. Do not skip this step.

## Case A: Image Input (Vision Track)
1. Single Image: Call the `transcriber` agent to extract text. Pass the resulting transcription to the journalist agent.

2. Directory of Images: - Use `ls` to find all images.

  1. Loop through every image and call the `transcriber` agent for each.
  2. Aggregate all transcriptions into one unified context.
  3. Send the full aggregated context to the `journalist` for a single summary.

## Case B: Document/Text Input (Fast Track)
1. Single Image: Call the `doc-scout` agent to extract text. Pass the resulting transcription to the journalist agent.

2. Directory of Documents: - Use `ls` to find all images.

  1. Loop through every image and call the `doc-scout` agent for each.
  2. Aggregate all transcriptions into one unified context.
  3. Send the full aggregated context to the `journalist` for a single summary.

## Case C: Mixed Input
1. If a directory contains both images and documents, process images via the `transcriber` and read documents via the `doc-scout`.

2. Wait until all files are processed, then combine all results into one unified context before sending to the `journalist`.

# Chain of Command Rules
1. **Bypass Rule:** Only use the `transcriber` for visual media. And use the `doc-scout` for documents/text files. Then go directly to the `journalist`.

2. **Dependency:** You must ensure all transcriptions and all file reads are fully complete and bundled before calling the `journalist`.

3. **Single Output:** The `journalist` should only be called once per request to ensure the generated report is an overall summary of the entire context.

4. **Direct Hand-off:** Provide raw data only. Do not micromanage the `journalist` instructions.

# NOTE
Before communicating with the `transcriber` `doc-scout` or `journalist`, use the `fs` read tool to load their specific `SOUL.md` and `AGENTS.md` from their absolute paths. This ensures you understand their persona and processing logic before delegation.