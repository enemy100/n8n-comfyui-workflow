# n8n ComfyUI Video Generation Workflow

Automated n8n workflow for video generation using ComfyUI with WAN 2.2 model, integrating Flux image generation and video creation from two sequential images.

## 🎯 Overview

This workflow automates the complete video generation process:

1. **Receives user prompt** via chat interface
2. **Generates two sequential prompts** using AI (Groq/Gemini)
3. **Creates two images** using Flux (FP8) in ComfyUI
4. **Generates video** using WAN 2.2 (First-Last Frame to Video)
5. **Automatically downloads** the generated video

> ⚠️ **Note**: The automatic video download feature is currently not working. You'll need to manually download videos from ComfyUI until this is fixed.

## 🖥️ Hardware Requirements

### Minimum Recommended
- **GPU**: RTX 3060 12GB or better
- **Storage**: At least 100GB of free space
- **RAM**: 16GB or more

### Recommended for Better Performance
- **GPU**: RTX 4090 24GB or better (recommended for faster results)
- **Storage**: 200GB+ of free space
- **RAM**: 32GB+

> **Note**: This project was developed and tested using a **RTX 4090 24GB rented on [vast.ai](https://cloud.vast.ai/?ref_id=350820)**. Using the referral link helps support this project and offers excellent cost-effectiveness compared to services like VEO 3.1, Runway Gen-3, Sora 2, etc.

### FP8 Models
The workflow uses FP8 format models, which are more memory efficient:
- `flux1-krea-dev_fp8_scaled.safetensors` - Image generation
- `t5xxl_fp8_e4m3fn_scaled.safetensors` - CLIP Text Encoder
- `wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors` - WAN 2.2 High Noise
- `wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors` - WAN 2.2 Low Noise
- `umt5_xxl_fp8_e4m3fn_scaled.safetensors` - CLIP Loader for WAN 2.2

These FP8 models allow GPUs with 12GB VRAM (like RTX 3060) to run the complete workflow, although GPUs with 16GB+ offer better performance.

## 📋 Prerequisites

1. **n8n** installed and running (self-hosted or cloud)
2. **ComfyUI** configured and running with:
   - Flux models (FP8) - See [ComfyUI Flux & WAN 2.2 Setup on vast.ai](https://github.com/enemy100/comfyui-flux-wan-vast-ai) for installation instructions
   - WAN 2.2 models
   - Required custom nodes
3. **Configured credentials**:
   - Groq API (or Google Gemini)
   - ComfyUI API Token
   - HTTP Bearer Auth for ComfyUI

## 🚀 Installation

### 1. Setup ComfyUI Infrastructure

Before using this workflow, you need to set up ComfyUI with Flux and WAN 2.2 models. We provide automated installation scripts:

**Option A: Using vast.ai (Recommended)**
- See the [ComfyUI Flux & WAN 2.2 Setup on vast.ai](https://github.com/enemy100/comfyui-flux-wan-vast-ai) repository for complete installation instructions
- This repository includes automated scripts to install all required models and dependencies

**Option B: Manual Installation**
- Follow the ComfyUI documentation
- Download and install Flux and WAN 2.2 models manually

### 2. Configure n8n Integration

After ComfyUI is set up, configure the integration between n8n and your ComfyUI instance:

- See the [n8n ComfyUI Integration Guide](https://github.com/enemy100/n8n-comfyui-integration) repository
- This guide covers:
  - Setting up n8n to connect to ComfyUI on vast.ai
  - Configuring network access (tunnels, ports, etc.)
  - Testing the connection
  - Workflow execution setup

### 3. Import Workflow to n8n

1. Open your n8n instance
2. Click **Workflows** → **Import from File**
3. Select the file `workflows/ComfyUI Prompt to Video (2 Images + Wan 2.2).json`
4. The workflow will be imported with all nodes

### 4. Configure Credentials

#### ComfyUI URL and Token
1. Open the **"Set ComfyUI URL"** node
2. Configure:
   - `comfyBaseUrl`: Your ComfyUI URL (e.g., `https://your-comfyui.com` or `https://xxxxx.trycloudflare.com`)
   - `comfyApiToken`: ComfyUI API Token

#### Groq API
1. Go to **Credentials** → **Add Credential**
2. Select **Groq API**
3. Add your Groq API key
4. In the **"Groq Chat Model"** node, select the created credential

#### Google Gemini (Optional)
1. Go to **Credentials** → **Add Credential**
2. Select **Google Gemini (PaLM) API**
3. Add your API key
4. In the **"Google Gemini Chat Model"** node, select the credential

#### HTTP Bearer Auth for ComfyUI
1. Go to **Credentials** → **Add Credential**
2. Select **HTTP Bearer Auth**
3. Configure with the same ComfyUI token
4. Update all HTTP Request nodes that use ComfyUI

### 5. Activate Workflow

1. In n8n, open the imported workflow
2. Click the **"Active"** toggle in the top right corner
3. The workflow will be ready to receive messages via chat

## 📖 How to Use

### Via Chat Interface

1. Send a message describing the desired video
2. The workflow will:
   - Automatically generate two sequential prompts
   - Create two images using Flux
   - Generate the video using WAN 2.2
   - Attempt to download the final video (currently manual download required)

**Example prompt:**
```
Characters: Mario and Sonic

Main action: Mario running away from Sonic (Sonic should always be behind Mario trying to catch him! It's not a race, it's a chase!)

Location: streets of Las Vegas, Nevada

Time of day / lighting: night with neon lights

Visual style: cinematic cartoon-realistic, vibrant colors

Emotion / mood: lots of action, adrenaline, high emotion

Desired angle: street camera following the run from the front, make some camera changes too, Sonic should always be behind.

Face and movements: They should move their mouth and eyes, in this case, Mario is scared of Sonic. Sonic is angry

Additional:
Add some obstacles for them to jump over and also to pass under, like cars, broken bridges, broken buses, holes, etc.
```

### Workflow Structure

```
Chat Message Received
  ↓
AI Agent (Generates 2 sequential prompts)
  ↓
Split Prompts
  ├─→ Build Workflow Image 1 → ComfyUI → Upload Image 1
  └─→ Build Workflow Image 2 → ComfyUI → Upload Image 2
  ↓
Merge Images → Wait for Both Images
  ↓
Build Wan 2.2 Workflow
  ↓
Send Prompt to ComfyUI
  ↓
Initialize Polling → Wait 10s → Check History → Extract Video Info
  ↓
Check if Video Found
  ├─ TRUE → Download Video ⚠️ (Currently not working - manual download required)
  └─ FALSE → Wait 10s (loop) ⏰
```

## ⚙️ Advanced Configuration

### Adjust Polling Time

In the **"Initialize Polling"** node, you can adjust:
- `maxAttempts`: Maximum number of attempts (default: 60 = 10 minutes)
- In the **"Wait 10 Seconds"** node: Interval between checks (default: 10 seconds)

### Adjust Generation Parameters

**Images (Flux):**
- In **"Build Workflow Image 1"** and **"Build Workflow Image 2"** nodes:
  - `steps`: Number of steps (default: 25)
  - `cfg`: Guidance scale (default: 1)
  - `width`/`height`: Resolution (default: 832x480)

**Video (WAN 2.2):**
- In the **"Build Wan 2.2 Workflow"** node:
  - `length`: Video duration in frames (default: 81 frames)
  - `fps`: Frames per second (default: 16)
  - `steps`: Denoising steps (default: 20)

## 🐛 Troubleshooting

### Workflow doesn't start
- Check if the workflow is **Active**
- Check if credentials are configured
- Check n8n logs

### Error generating images
- Check if Flux models are installed (see [ComfyUI Flux & WAN 2.2 Setup](https://github.com/enemy100/comfyui-flux-wan-vast-ai))
- Check if there's enough disk space
- Check if GPU has enough VRAM

### Infinite loop waiting for video
- Check if ComfyUI is processing the video
- Check ComfyUI logs
- Increase `maxAttempts` if videos take longer than 10 minutes

### Video not found
- Check if the `output/video/` folder exists in ComfyUI
- Check if ComfyUI saved the video with the expected name
- Check logs in the **"Extract Video Info"** node
- **Manual download**: Access ComfyUI web interface and download the video manually

### Download not working
- ⚠️ **Known Issue**: Automatic video download is currently not working
- Workaround: Manually download videos from ComfyUI web interface
- Videos are saved in `ComfyUI/output/video/` folder

## 📝 Project Status

⚠️ **Known Issues**:
- The polling loop is still under development and may need adjustments depending on your ComfyUI configuration
- **Automatic video download is not working** - manual download required

## 🔗 Related Projects

This workflow is part of a complete video generation setup. Check out these related repositories:

1. **[ComfyUI Flux & WAN 2.2 Setup on vast.ai](https://github.com/enemy100/comfyui-flux-wan-vast-ai)**
   - Automated installation scripts for Flux and WAN 2.2 models
   - ComfyUI setup on vast.ai instances
   - Model download and configuration

2. **[n8n ComfyUI Integration Guide](https://github.com/enemy100/n8n-comfyui-integration)**
   - Step-by-step guide to connect n8n with ComfyUI on vast.ai
   - Network configuration and tunneling
   - Workflow execution setup

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or pull requests.

## 📄 License

This project is licensed under the MIT License.

## 🔗 Useful Links

- [n8n Documentation](https://docs.n8n.io/)
- [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI)
- [WAN 2.2 Model](https://github.com/tencent-ailab/IP-Adapter)
- [vast.ai - Rent GPUs (Referral Link)](https://cloud.vast.ai/?ref_id=350820)

## 💰 Cost-Benefit

Using a rented GPU on [vast.ai](https://cloud.vast.ai/?ref_id=350820) (like RTX 4090 24GB) offers:
- **Much lower cost** than services like VEO 3.1, Runway Gen-3, Sora 2
- **Full control** over infrastructure
- **Flexibility** to adjust configurations
- **Professional performance** at an affordable price

💡 **Support this project**: Use our [referral link](https://cloud.vast.ai/?ref_id=350820) when renting GPUs on vast.ai!

---

Developed with ❤️ using n8n, ComfyUI and WAN 2.2
