# OmniSentinel - Automated Content Moderation Gateway

## Architecture Overview

```
┌─────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   INPUT     │───▶│  HF INFERENCE    │───▶│  DECISION GATE  │
│  (Webhook)  │    │  (flan-t5-small) │    │   (IF Nodes)    │
└─────────────┘    └──────────────────┘    └─────────────────┘
                                                    │
                          ┌─────────────────────────┼─────────────────────────┐
                          ▼                         ▼                         ▼
                   ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
                   │    CLEAN    │          │   REVIEW    │          │   BLOCKED   │
                   │   (Pass)    │          │  (Flag)     │          │  (Reject)   │
                   └─────────────┘          └─────────────┘          └─────────────┘
```

## Tech Stack
- **Orchestrator:** n8n (Local/Cloud)
- **Inference:** Hugging Face Inference API (Router API 2025)
- **Model:** google/flan-t5-small
- **Protocol:** REST/POST

## Quick Start

1. Import the workflow JSON into n8n
2. Configure your HF API credential
3. Test with sample inputs

## File Structure
```
omnisentinel/
├── README.md                    # This file
├── workflows/
│   └── content_moderation.json  # Main n8n workflow
├── prompts/
│   └── moderation_prompts.md    # Prompt templates
├── config/
│   └── api_config.md            # API configuration guide
└── docs/
    └── decision_logic.md        # IF Node documentation
```
