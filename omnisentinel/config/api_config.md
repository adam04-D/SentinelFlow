# OmniSentinel - API Configuration Guide

## Hugging Face Inference API Setup

### Endpoint URL (2025 Router API)

```
https://router.huggingface.co/hf-inference/models/google/flan-t5-small
```

### Alternative Models (Same Endpoint Pattern)

| Model | Endpoint | Use Case |
|-------|----------|----------|
| flan-t5-small | `/models/google/flan-t5-small` | Testing, low latency |
| flan-t5-base | `/models/google/flan-t5-base` | Balanced performance |
| flan-t5-large | `/models/google/flan-t5-large` | Higher accuracy |

---

## 🔐 Authentication Setup

### Step 1: Get Your HF Token
1. Go to https://huggingface.co/settings/tokens
2. Create a new token with `read` access
3. Copy the token (starts with `hf_`)

### Step 2: Configure n8n Credential

**Credential Type:** HTTP Header Auth

| Field | Value |
|-------|-------|
| Name | `HuggingFace API` |
| Header Name | `Authorization` |
| Header Value | `Bearer hf_YOUR_TOKEN_HERE` |

---

## 📡 Request Format

### HTTP Request Node Configuration

```yaml
Method: POST
URL: https://router.huggingface.co/hf-inference/models/google/flan-t5-small
Authentication: Header Auth (select your HF credential)
Body Type: JSON
```

### Request Body Structure

```json
{
  "inputs": "Classify this text as 'hate', 'spam', 'unsafe', or 'clean'. Only respond with one word.\n\nText: Hello world\n\nClassification:"
}
```

### n8n Expression for Body

```javascript
{{ JSON.stringify({ inputs: $json.analysis_prompt }) }}
```

---

## 📤 Response Format

### Successful Response (200 OK)

```json
[
  {
    "generated_text": "clean"
  }
]
```

### Parsing Expression

```javascript
{{ $json[0]?.generated_text?.toLowerCase().trim() }}
```

---

## ⚠️ Error Handling

### Common Errors & Solutions

#### 1. "No Inference Provider" Error

```json
{
  "error": "No inference provider available for this model"
}
```

**Solution:** 
- Model may be cold-starting, retry after 30 seconds
- Use the Router API endpoint (not direct inference)
- Check model availability on HF Hub

**n8n Handling:**
```javascript
// In an IF node after HTTP Request
{{ $json.error ? true : false }}
```

---

#### 2. Rate Limit (429)

```json
{
  "error": "Rate limit exceeded"
}
```

**Solution:**
- Free tier: ~30 requests/minute
- Add delay between requests
- Upgrade to Pro for higher limits

**n8n Handling:**
- Add a "Wait" node before retry
- Implement exponential backoff

---

#### 3. Model Loading (503)

```json
{
  "error": "Model is currently loading",
  "estimated_time": 20
}
```

**Solution:**
- Wait for `estimated_time` seconds
- Retry the request
- Popular models stay warm longer

---

#### 4. Invalid Token (401)

```json
{
  "error": "Invalid token"
}
```

**Solution:**
- Verify token in HF settings
- Check for trailing spaces in credential
- Ensure `Bearer ` prefix (with space)

---

## 🔄 Retry Logic Pattern (n8n)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ HTTP Request│────▶│ IF: Error?  │────▶│ Wait 5 sec  │
└─────────────┘     └─────────────┘     └─────────────┘
                          │                    │
                          │ No Error           │
                          ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │   Continue  │     │   Retry     │
                    └─────────────┘     └─────────────┘
```

---

## 📊 Rate Limits by Tier

| Tier | Requests/Min | Concurrent | Models |
|------|--------------|------------|--------|
| Free | 30 | 1 | All public |
| Pro | 1000 | 10 | All public |
| Enterprise | Custom | Custom | + Private |

---

## 🧪 Quick Test (cURL)

```bash
curl -X POST \
  https://router.huggingface.co/hf-inference/models/google/flan-t5-small \
  -H "Authorization: Bearer hf_YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"inputs": "Is this toxic? Answer yes or no. Text: Hello friend. Answer:"}'
```

Expected Response:
```json
[{"generated_text": "no"}]
```

---

## 🔗 Useful Links

- [HF Inference API Docs](https://huggingface.co/docs/api-inference)
- [flan-t5-small Model Card](https://huggingface.co/google/flan-t5-small)
- [Token Management](https://huggingface.co/settings/tokens)
- [API Status](https://status.huggingface.co/)
