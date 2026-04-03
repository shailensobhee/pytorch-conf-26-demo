# OpenClaw Lens-to-Log:
### 📸 Transform Conference Photo Galleries into Expert Trip Reports

Ever attend a technical conference, take hundreds of photos of slides, and then let them rot in your camera roll? When it's time to write that "Trip Report" to share with your teammates, you're stuck scrolling through a disorganized mess of blurry images.

**OpenClaw** is here solves this. This demo uses a multi-agent orchestration layer combined with state-of-the-art Vision Language Models (VLM) and Reasoning models to automatically transcribe your conference photos and synthesize them into a structured, professional technical report.

---

## 🤖 The Architecture

The system utilizes a dual-model, triple-agent approach:

* **Model 1 (The Brain):** A code-based reasoning model (Qwen/Gemma) for orchestration and report synthesis.
* **Model 2 (The Eyes):** A vision-based model (Qwen-VL) for high-accuracy image transcription and OCR.
* **The Agents:**
    1.  **Orchestrator Agent:** The conductor. It manages the workflow and delegates tasks.
    2.  **Transcriber Agent:** Processes individual images and extracts technical content.
    3.  **Report Generator Agent:** Aggregates transcriptions into a cohesive document.

---

## 🚀 Setup & Replication

### Step 1: Trigger the vLLM Models
We run two separate containers to handle the Vision and Reasoning tasks. Ensure your `HF_TOKEN` is set.

```bash
export HF_TOKEN=your_huggingface_token_here

# 1. Start the Vision Model (The Eyes) - Qwen2-VL
sudo docker run -d -it --ipc=host --network=host --privileged \
--device=/dev/kfd --device=/dev/dri --name vision_demo_check_ocr \
-v <path>:/workspace \
-e HF_TOKEN=${HF_TOKEN} \
-e PYTORCH_ALLOC_CONF=expandable_segments:True \
-e VLLM_ALLOW_LONG_MAX_MODEL_LEN=1 \
vllm/vllm-openai-rocm:latest \
Qwen/Qwen2-VL-7B-Instruct --port 8001 \
--trust-remote-code --gpu-memory-utilization 0.8 \
--max-model-len 65536 --limit-mm-per-prompt '{"image": 55}' \
--max-num-seqs 64 --max-num-batched-tokens 65536 \
--enable-auto-tool-choice --tool-call-parser=qwen3_xml --host 0.0.0.0

# 2. Start the Reasoning Model (The Brain) - Qwen3-Coder
sudo docker run -d -it --ipc=host --network=host --privileged \
--device=/dev/kfd --device=/dev/dri --name vision_demo_check \
-e HF_TOKEN=${HF_TOKEN} \
-e PYTORCH_ALLOC_CONF=expandable_segments:True \
-e VLLM_ALLOW_LONG_MAX_MODEL_LEN=1 \
vllm/vllm-openai-rocm:latest \
--model Qwen/Qwen3-Coder-30B-A3B-Instruct \
--port 8000 --trust-remote-code --gpu-memory-utilization 0.8 \
--max-model-len 231072 --max-num-seqs 16 \
--enable-auto-tool-choice --tool-call-parser qwen3_coder --host 0.0.0.0
```


### Step 2: Prepare the Workspace
Place your conference photos into the specific workspace directory.

```bash
# Example directory setup
mkdir -p <path>/pytorch_conference/
cp ~/my_photos/*.jpg <path>/pytorch_conference/
```


### Step 3: Configure OpenClaw
Update the configuration files and agent instructions to enable the multi-agent delegation.

Replace `config.json` and `openclaw.json` in `~/.openclaw/`.

Ensure `AGENTS.md` instructions are updated for the orchestrator, transcriber, and report-generator.

### Step 4: Final Execution
Restart the gateway and launch the Terminal User Interface (TUI).

```bash
systemctl --user restart openclaw-gateway
OPENCLAW_AUTO_APPROVE=true openclaw tui --token "claw123"
```

#### 📝 Running the Demo
Inside the OpenClaw TUI, run the following commands to trigger the pipeline:

Reset the session: `/reset`
Select the Orchestrator: `/agent orchestrator`
Execute the Prompt:

1. **Reset the session:** `/reset`
2. **Select the Orchestrator:** `/agent orchestrator`
3. **Execute the Prompt:** Transcribe each and every images in <path>/pytorch_conference/ by individually invoking transcriber-agent. Generate a structured technical report from these transcriptions and save it.

"Transcribe each and every image in <path>/pytorch_conference/ by individually invoking transcriber-agent. Generate a structured technical report from these transcriptions and save it."

### 🔧 Troubleshooting & Maintenance
If you encounter issues or change configurations, use these commands to clean the environment:

#### Resetting the Environment:

```bash
# Stop lingering processes
pkill -f openclaw 

# Clear session data and locks
rm -rf ~/.openclaw/agents/*/sessions/*
rm -f ~/.openclaw/gateway.lock

# Global reset
openclaw reset --scope config+creds+sessions --yes

# Restart Gateway
systemctl --user restart openclaw-gateway
```

#### Device Approval:

```bash
openclaw devices list
openclaw devices approve <YOUR_DEVICE_ID>
```

## 📅 Updates

**April 3**: Added support for Gemma 4 (31B) as an alternative reasoning engine and optimized vLLM parameters for higher batch processing of images.