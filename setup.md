# Local FLUX.1 schnell Image Generator: Complete Setup

A fully local, free-to-run text-to-image system with a simple web page (prompt box, size, steps, seed).

- **Model:** FLUX.1 [schnell], GGUF quantized
- **Backend:** ComfyUI (runs hidden on localhost)
- **Front end:** a small Gradio app (`app.py`) on port 7860
- **No cloud, no paid API.** Everything runs on your own GPU.

> Items marked **(verify)** come from memory or secondary sources. Check them against the linked repos before relying on them.

---

## 1. Architecture

```
Browser (any device on your LAN)
        |  http://<machine-ip>:7860
        v
Gradio app  (app.py)        <- builds a workflow, polls for the result
        |  http://127.0.0.1:8188   (localhost only)
        v
ComfyUI + ComfyUI-GGUF      <- loads FLUX, runs the sampler, saves the image
        |
        v
GPU (CUDA)
```

ComfyUI has no login, so it stays bound to `127.0.0.1`. Only the Gradio page is exposed to the LAN.

## 2. Pick your profile

| | Profile A: Linux box | Profile B: Windows PC |
|---|---|---|
| GPU | RTX 3060, 12 GB VRAM | NVIDIA, 6 GB VRAM |
| RAM | 16 GB or more | 16 GB |
| Diffusion model | `flux1-schnell-Q4_K_S.gguf` (~6.78 GB) | `flux1-schnell-Q3_K_S.gguf` or `Q2_K` (**verify** sizes on the repo) |
| Start size | 768 x 1024 | 512 x 512, then step up |
| Extra ComfyUI flag | none | `--lowvram` |
| Speed | a few seconds after first load | slower, tens of seconds to minutes (rough estimate) |

Both profiles use the same text encoders, VAE and `app.py`.

## 3. Files you need

| File | Source repo | Size | Destination folder |
|---|---|---|---|
| `flux1-schnell-Q4_K_S.gguf` (or Q3_K_S / Q2_K) | `city96/FLUX.1-schnell-gguf` | ~6.78 GB (Q4_K_S) | `models/unet` |
| `clip_l.safetensors` | `comfyanonymous/flux_text_encoders` | ~0.25 GB | `models/clip` |
| `t5xxl_fp8_e4m3fn.safetensors` | `comfyanonymous/flux_text_encoders` | ~4.9 GB | `models/clip` |
| `ae.safetensors` | `black-forest-labs/FLUX.1-schnell` (gated) | ~335 MB | `models/vae` |

Reserve **30 GB or more** of free disk space.

### The VAE access problem

The `black-forest-labs/FLUX.1-schnell` repo requires you to be logged in and to have accepted its terms. Without that you get `Access denied. Model ... requires approval.`

**Option 1: official route**

1. Sign in at huggingface.co and accept the terms on the FLUX.1-schnell model page.
2. Create a **read** token at `https://huggingface.co/settings/tokens`.
3. Run `hf auth login` (older versions: `huggingface-cli login`), then download `ae.safetensors`.

**Option 2: ungated mirror (verify the path exists)**

```bash
hf download Comfy-Org/Lumina_Image_2.0_Repackaged split_files/vae/ae.safetensors --local-dir /tmp/ae
mv /tmp/ae/split_files/vae/ae.safetensors models/vae/ae.safetensors
```

Confirm the file is about 335 MB either way.

---

## 4. Profile A: Linux (RTX 3060 12 GB)

### 4.1 Inspect

```bash
nvidia-smi
lsb_release -a && uname -m
df -h ~
python3 --version; git --version
```

You need a working NVIDIA driver (535 or newer is fine). You do **not** need the CUDA toolkit, because the PyTorch wheels bundle their own CUDA runtime.

### 4.2 Install ComfyUI and the GGUF node

```bash
sudo apt update && sudo apt install -y git python3-venv python3-pip
mkdir -p ~/imageGen && cd ~/imageGen
git clone https://github.com/Comfy-Org/ComfyUI.git
cd ComfyUI
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
pip install -r requirements.txt

git clone https://github.com/city96/ComfyUI-GGUF custom_nodes/ComfyUI-GGUF
pip install -r custom_nodes/ComfyUI-GGUF/requirements.txt
pip install -U huggingface_hub gradio requests pillow
```

Check the ComfyUI README for the currently recommended PyTorch index URL (`cu126`, `cu128`, and so on).

### 4.3 Download models

Run from `~/imageGen/ComfyUI` with the venv active:

```bash
hf download city96/FLUX.1-schnell-gguf flux1-schnell-Q4_K_S.gguf --local-dir models/unet
hf download comfyanonymous/flux_text_encoders clip_l.safetensors t5xxl_fp8_e4m3fn.safetensors --local-dir models/clip
# VAE: see "The VAE access problem" in section 3
ls -lh models/unet models/clip models/vae
```

If `hf` is not found, use `huggingface-cli download` with the same arguments.

### 4.4 Create `app.py`

Save the code from **section 6** as `~/imageGen/app.py`.

### 4.5 Create `start.sh`

The quoted `'EOF'` is important. It stops bash from expanding anything while the file is written.

```bash
cat > ~/imageGen/start.sh <<'EOF'
#!/usr/bin/env bash
cd ~/imageGen/ComfyUI
source venv/bin/activate

# ComfyUI backend (localhost only), log to file
python main.py --listen 127.0.0.1 --port 8188 > ~/imageGen/comfy.log 2>&1 &
COMFY_PID=$!
trap "kill $COMFY_PID 2>/dev/null" EXIT

# wait until the backend answers
until curl -s http://127.0.0.1:8188/system_stats >/dev/null; do sleep 2; done
echo "ComfyUI is up. UI on port 7860 (Ctrl+C stops everything)"

# Gradio UI in the foreground
python ~/imageGen/app.py
EOF
chmod +x ~/imageGen/start.sh
```

If you created the file with `touch` and an editor, it will not be executable. Run `chmod +x ~/imageGen/start.sh` or start it with `bash ~/imageGen/start.sh`.

### 4.6 Run

```bash
~/imageGen/start.sh
```

Wait 10 to 30 seconds until you see "ComfyUI is up", then open `http://<server-ip>:7860` from any device on your LAN. Press Ctrl+C to stop both processes.

To keep it running after you close SSH, use tmux:

```bash
tmux new -s flux
~/imageGen/start.sh
# detach: Ctrl+B then D      reattach: tmux attach -t flux
```

### 4.7 LAN access and firewall

Gradio already listens on `0.0.0.0:7860`. If `ufw` is active:

```bash
sudo ufw status
sudo ufw allow from 192.168.1.0/24 to any port 7860 proto tcp
```

Adjust the subnet to match your network. Do **not** open port 8188. Reserve a fixed IP for the machine in your router's DHCP settings so the URL stays stable.

---

## 5. Profile B: Windows (6 GB VRAM, 16 GB RAM)

### 5.1 Prerequisites

1. Install **Git for Windows** from git-scm.com.
2. Update your NVIDIA driver and confirm it with `nvidia-smi` in Command Prompt.
3. Set a large page file (system-managed, or about 16 to 24 GB) on an SSD: *Settings > System > About > Advanced system settings > Performance > Advanced > Virtual memory*.
4. Keep at least 25 GB free on the target drive.

### 5.2 Install ComfyUI (portable build)

Download the official **Windows portable NVIDIA** build from the Releases page of `github.com/Comfy-Org/ComfyUI` and extract it with 7-Zip, for example to `D:\AI\ComfyUI_windows_portable`. Then:

```bat
cd D:\AI\ComfyUI_windows_portable\ComfyUI\custom_nodes
git clone https://github.com/city96/ComfyUI-GGUF
cd ..\..
python_embeded\python.exe -m pip install -r ComfyUI\custom_nodes\ComfyUI-GGUF\requirements.txt
python_embeded\python.exe -m pip install gradio requests pillow huggingface_hub
```

### 5.3 Download models

Place the files from section 3 under `D:\AI\ComfyUI_windows_portable\ComfyUI\models\` (`unet`, `clip`, `vae`). Use the Q3_K_S or Q2_K GGUF file instead of Q4_K_S. The simplest route is to download them in your browser from each repo's Files tab. Command-line downloads work too:

```bat
python_embeded\Scripts\hf.exe download city96/FLUX.1-schnell-gguf flux1-schnell-Q3_K_S.gguf --local-dir ComfyUI\models\unet
```

**Optional RAM saver (verify):** a GGUF version of the T5 encoder exists (search "t5-v1_1-xxl-encoder-gguf" on Hugging Face). It loads through the **DualCLIPLoader (GGUF)** node from ComfyUI-GGUF. Confirm the file names and node class name in that repo's README before switching.

### 5.4 Create `app.py`

Save the code from **section 6** as `D:\AI\app.py`, then change:

- `UNET = "flux1-schnell-Q3_K_S.gguf"` (match your actual file name)
- Slider defaults: width 512, height 640, steps 4

### 5.5 Create `start.bat`

Save next to `app.py`:

```bat
@echo off
cd /d D:\AI\ComfyUI_windows_portable
start "ComfyUI" python_embeded\python.exe -s ComfyUI\main.py --listen 127.0.0.1 --port 8188 --lowvram
timeout /t 30 > nul
python_embeded\python.exe D:\AI\app.py
```

The 30-second wait is a guess. If the first click errors, wait for ComfyUI to finish loading and retry.

### 5.6 LAN access

In an Administrator Command Prompt:

```bat
netsh advfirewall firewall add rule name="FluxUI" dir=in action=allow protocol=TCP localport=7860 profile=private
```

Find your IP with `ipconfig` and browse to `http://<your-windows-ip>:7860`.

### 5.7 Low-VRAM tips

- Start at 512x512, 4 steps, then move up to 512x768 and 640x800 if stable.
- Close browsers and other GPU-heavy apps. Watch VRAM in Task Manager's GPU tab.
- On out-of-memory errors: drop to Q2_K, lower the resolution, and check nothing else is using the GPU.
- If FLUX is too slow, SDXL Turbo or Lightning models run comfortably on 6 GB, with weaker prompt following and text rendering.

---

## 6. `app.py` (same for both profiles)

```python
import io, json, random, time
import requests
import gradio as gr
from PIL import Image

COMFY = "http://127.0.0.1:8188"

# Edit UNET to match your downloaded file (Windows: the Q3_K_S / Q2_K name)
UNET = "flux1-schnell-Q4_K_S.gguf"
CLIP_L = "clip_l.safetensors"
T5 = "t5xxl_fp8_e4m3fn.safetensors"
VAE = "ae.safetensors"


def build_workflow(prompt, width, height, steps, seed):
    return {
        "1": {"class_type": "UnetLoaderGGUF", "inputs": {"unet_name": UNET}},
        "2": {"class_type": "DualCLIPLoader",
              "inputs": {"clip_name1": CLIP_L, "clip_name2": T5, "type": "flux"}},
        "3": {"class_type": "VAELoader", "inputs": {"vae_name": VAE}},
        "4": {"class_type": "CLIPTextEncode",
              "inputs": {"text": prompt, "clip": ["2", 0]}},
        "5": {"class_type": "CLIPTextEncode",
              "inputs": {"text": "", "clip": ["2", 0]}},
        "6": {"class_type": "EmptyLatentImage",
              "inputs": {"width": width, "height": height, "batch_size": 1}},
        "7": {"class_type": "KSampler",
              "inputs": {"model": ["1", 0], "positive": ["4", 0],
                         "negative": ["5", 0], "latent_image": ["6", 0],
                         "seed": seed, "steps": steps, "cfg": 1.0,
                         "sampler_name": "euler", "scheduler": "simple",
                         "denoise": 1.0}},
        "8": {"class_type": "VAEDecode",
              "inputs": {"samples": ["7", 0], "vae": ["3", 0]}},
        "9": {"class_type": "SaveImage",
              "inputs": {"images": ["8", 0], "filename_prefix": "flux_ui"}},
    }


def generate(prompt, width, height, steps, seed, randomize):
    if not prompt.strip():
        raise gr.Error("Enter a prompt first.")
    seed = random.randint(0, 2**32 - 1) if randomize else int(seed)
    wf = build_workflow(prompt, int(width), int(height), int(steps), seed)

    try:
        r = requests.post(f"{COMFY}/prompt", json={"prompt": wf}, timeout=30)
    except requests.exceptions.ConnectionError:
        raise gr.Error("Cannot reach ComfyUI on 127.0.0.1:8188. Is it running?")
    if r.status_code != 200:
        raise gr.Error(f"ComfyUI rejected the workflow: {r.text[:500]}")
    pid = r.json()["prompt_id"]

    start = time.time()
    while time.time() - start < 600:
        h = requests.get(f"{COMFY}/history/{pid}", timeout=30).json()
        if pid in h:
            entry = h[pid]
            if entry.get("status", {}).get("status_str") == "error":
                raise gr.Error(f"Generation failed: {json.dumps(entry['status'])[:500]}")
            img = entry["outputs"]["9"]["images"][0]
            data = requests.get(f"{COMFY}/view", params=img, timeout=60).content
            return Image.open(io.BytesIO(data)), seed, f"{time.time()-start:.1f}s"
        time.sleep(0.5)
    raise gr.Error("Timed out waiting for ComfyUI.")


with gr.Blocks(title="FLUX Schnell") as demo:
    gr.Markdown("## FLUX.1 schnell (local)")
    with gr.Row():
        with gr.Column():
            prompt = gr.Textbox(label="Prompt", lines=5)
            with gr.Row():
                # Windows 6 GB: change defaults to 512 and 640
                width = gr.Slider(512, 1536, 768, step=64, label="Width")
                height = gr.Slider(512, 1536, 1024, step=64, label="Height")
            steps = gr.Slider(1, 8, 4, step=1, label="Steps")
            with gr.Row():
                seed = gr.Number(42, label="Seed", precision=0)
                randomize = gr.Checkbox(True, label="Random seed")
            btn = gr.Button("Generate", variant="primary")
        with gr.Column():
            out = gr.Image(label="Result", type="pil")
            used_seed = gr.Number(label="Seed used", precision=0)
            took = gr.Textbox(label="Time")
    btn.click(generate, [prompt, width, height, steps, seed, randomize],
              [out, used_seed, took])

demo.queue(max_size=8).launch(server_name="0.0.0.0", server_port=7860)
```

Notes on the workflow:

- FLUX schnell is distilled and ignores guidance, so CFG stays at **1.0** and the negative prompt is empty. Negative prompts have no effect.
- Node class names (`UnetLoaderGGUF`, `DualCLIPLoader`) were written from knowledge of the ComfyUI-GGUF repo and were not validated against every version. If ComfyUI rejects the workflow, the error appears in the Gradio page. Check the node names in the ComfyUI-GGUF README.
- To get a known-good graph, open ComfyUI's built-in FLUX schnell template, swap the loader for **Unet Loader (GGUF)**, and use *Save (API Format)* with Dev Mode enabled.

## 7. Verify the backend

```bash
curl -s http://127.0.0.1:8188/system_stats | head -c 300
curl -s http://127.0.0.1:8188/object_info/UnetLoaderGGUF | head -c 150
```

Both should return JSON. If the second is empty, the GGUF node failed to load. Check `~/imageGen/comfy.log` (Linux) for import errors.

---

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Connection refused` to `127.0.0.1:8188` | ComfyUI is not running | Start ComfyUI first, or use `start.sh` / `start.bat`. Check `comfy.log`. |
| URL on the server IP does not load | ComfyUI was bound to `127.0.0.1`; or firewall | Use the Gradio port 7860 (bound to `0.0.0.0`) and open that port. |
| `Access denied ... requires approval` | Gated VAE repo | Accept terms, `hf auth login`, or use the mirror (section 3). |
| `Permission denied` running `start.sh` | File is not executable | `chmod +x ~/imageGen/start.sh` or `bash ~/imageGen/start.sh` |
| `No usable temporary directory found` | `/tmp` missing, wrong permissions, or disk full | `df -h /`, free space, `sudo chmod 1777 /tmp` |
| Shell `syntax error near unexpected token '('` | Mangled quotes in a pasted script | Recreate the file with the `cat > ... <<'EOF'` pattern |
| Workflow rejected: model/node not found | File name mismatch or node not loaded | Match `UNET`, `CLIP_L`, `T5`, `VAE` in `app.py` to the real files; check the GGUF node loaded |
| CUDA out of memory | Resolution too high, other GPU apps running | Lower resolution, add `--lowvram`, use a smaller quant, close other apps |
| First image is very slow | Models loading into memory | Normal. Later images are faster. |
| Duplicate 12 GB of models | Backup copy left behind after a rebuild | Confirm models are in `models/`, then delete the backup folder |

Useful monitors: `watch -n1 nvidia-smi` (Linux), Task Manager's GPU tab (Windows).

## 9. Security

- Keep ComfyUI on `127.0.0.1`. It has no authentication.
- The Gradio page also has no login. Keep it on a trusted LAN. Never port-forward 7860 or 8188 to the internet. For remote access use a VPN such as WireGuard or Tailscale, or add basic auth with `launch(auth=("user", "pass"))`.

## 10. Licenses

- FLUX.1 schnell is, as far as I know, Apache 2.0 (**verify** on the model card).
- The GGUF files are community conversions, so check `city96/FLUX.1-schnell-gguf` for its terms.
- The VAE and text encoders come from their own repos with their own terms. Check each model card before commercial use.

## 11. Test prompts

```
Photorealistic editorial portrait of an adult woman in her late twenties, natural skin texture with subtle pores, soft window light from the left, 85mm lens, shallow depth of field, lifelike eyes, calm authentic expression, neutral indoor background, natural color grading
```
```
A steaming cup of masala chai on a rustic wooden table beside a window, rain outside, warm moody light, shallow depth of field, food photography
```
```
A lone astronaut standing on a red desert planet looking at two moons, long shadows, cinematic, dusty atmosphere
```
```
A chalkboard menu outside a cafe with the words "TODAY'S SPECIAL" written at the top in neat handwriting, morning light, photographed at an angle
```

Prompt formula: subject, action or pose, setting, lighting, camera or style, mood. Use a fixed seed while refining a prompt, then switch to random.

## 12. Optional extensions

**Identify or describe images (vision-language models).** FLUX only generates images. To describe uploaded images, add a separate VLM such as Qwen3-VL or Qwen3.5 (via Ollama), or a captioner such as JoyCaption or Florence-2. On 12 GB you generally cannot run a VLM and FLUX at the same time, so load them one after another.

**Fine-tune with a LoRA.** FluxGym and OneTrainer are the most plausible options for 12 GB cards. They train against the original FLUX weights, not the GGUF file, so expect an extra download and gated-repo access. A LoRA is typically trained on 15 to 30 well-captioned images. Windows with 6 GB VRAM is not a realistic training target.

**Cleanup.** After confirming the models are in `models/unet`, `models/clip` and `models/vae`, delete any backup copy to free about 12 GB.

## 13. Quick reference

| Task | Linux | Windows |
|---|---|---|
| Start everything | `~/imageGen/start.sh` | run `start.bat` |
| Stop | Ctrl+C | close both windows |
| UI | `http://<server-ip>:7860` | `http://<windows-ip>:7860` |
| Backend (local only) | `http://127.0.0.1:8188` | `http://127.0.0.1:8188` |
| Logs | `~/imageGen/comfy.log` | ComfyUI console window |
| Models | `~/imageGen/ComfyUI/models/` | `...\ComfyUI\models\` |
| Output images | `~/imageGen/ComfyUI/output/` | `...\ComfyUI\output\` |
