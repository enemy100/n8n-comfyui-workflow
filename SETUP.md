# Setup Guide

Quick setup instructions for the n8n ComfyUI workflow.

## Prerequisites

- n8n instance (self-hosted or cloud)
- ComfyUI instance running and accessible
- API keys for Groq (or Google Gemini)

## Step-by-Step Setup

### 1. Import Workflow

1. Open n8n
2. Go to **Workflows** → **Import from File**
3. Select `workflows/ComfyUI Prompt to Video (2 Images + Wan 2.2).json`

### 2. Configure Credentials

Replace all placeholder values in the workflow:

- `YOUR_COMFYUI_URL_HERE` → Your ComfyUI URL
- `YOUR_COMFYUI_API_TOKEN_HERE` → Your ComfyUI API token
- `YOUR_WEBHOOK_ID_HERE` → Your webhook ID (if using webhook trigger)
- `YOUR_BEARER_AUTH_CREDENTIAL_ID` → Your HTTP Bearer Auth credential ID
- `YOUR_GROQ_API_CREDENTIAL_ID` → Your Groq API credential ID
- `YOUR_GEMINI_API_CREDENTIAL_ID` → Your Gemini API credential ID (optional)

### 3. Test the Workflow

1. Activate the workflow
2. Send a test message via chat
3. Monitor the execution logs

## Troubleshooting

See the main README.md for detailed troubleshooting steps.

