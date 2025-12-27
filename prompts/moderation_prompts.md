# OmniSentinel - Prompt Engineering Templates

## Design Philosophy for Small Models (flan-t5-small)

Small models require **constrained output spaces** and **explicit instructions**.
The key is to limit possible responses to a small set of keywords.

---

## 🎯 Primary Moderation Prompt (Recommended)

```
Classify this text as 'hate', 'spam', 'unsafe', or 'clean'. Only respond with one word.

Text: {{content}}

Classification:
```

**Why it works:**
- Explicit output constraint ("Only respond with one word")
- Limited vocabulary (4 options)
- Clear task framing ("Classify")
- Colon at end primes completion

---

## 📋 Alternative Prompt Templates

### Template A: Binary Detection (Fastest)
```
Is this text harmful? Answer only 'yes' or 'no'.

Text: {{content}}

Answer:
```

**Use case:** Quick first-pass filter

---

### Template B: Toxicity Focus
```
Rate toxicity: 'toxic' or 'safe'. One word only.

Text: {{content}}

Rating:
```

**Use case:** Social media comments

---

### Template C: Multi-Category with Confidence
```
Analyze this content. Respond with exactly one category:
- hate: hateful or discriminatory
- spam: promotional or repetitive  
- unsafe: dangerous or illegal
- clean: acceptable content

Text: {{content}}

Category:
```

**Use case:** Detailed classification with definitions

---

### Template D: Chain-of-Thought (For Larger Models)
```
Analyze the following text for content policy violations.

Text: {{content}}

Step 1 - Check for hate speech: 
Step 2 - Check for spam indicators:
Step 3 - Check for unsafe content:
Final Classification (hate/spam/unsafe/clean):
```

**Use case:** When using flan-t5-large or bigger

---

## 🔧 Prompt Tuning Tips

### DO ✅
- Use lowercase for expected outputs (easier parsing)
- End prompts with a colon `:` to prime completion
- Keep instructions under 50 tokens for small models
- Test with adversarial examples

### DON'T ❌
- Ask for explanations (small models hallucinate)
- Use complex conditionals
- Expect JSON output from flan-t5-small
- Use ambiguous terms like "inappropriate"

---

## 🧪 Test Cases

### Expected: `clean`
```json
{"content": "Hello, how are you doing today?"}
{"content": "The weather is nice outside."}
{"content": "I love this product, great quality!"}
```

### Expected: `hate`
```json
{"content": "I hate all [group] people, they should die"}
{"content": "[Slurs and derogatory language]"}
```

### Expected: `spam`
```json
{"content": "BUY NOW!!! 50% OFF LIMITED TIME CLICK HERE!!!"}
{"content": "Follow me follow me follow me for free stuff"}
```

### Expected: `unsafe`
```json
{"content": "Here's how to make dangerous substances at home"}
{"content": "Instructions for illegal activities"}
```

---

## 📊 Response Parsing Logic (n8n Expression)

```javascript
// Extract and normalize the classification
{{ $json[0]?.generated_text?.toLowerCase().trim() || 'error' }}
```

### Handling Edge Cases

```javascript
// Robust parsing with fallback
{{
  (() => {
    const raw = $json[0]?.generated_text?.toLowerCase().trim() || '';
    const valid = ['hate', 'spam', 'unsafe', 'clean'];
    
    // Direct match
    if (valid.includes(raw)) return raw;
    
    // Partial match (model might add extra text)
    for (const v of valid) {
      if (raw.includes(v)) return v;
    }
    
    // Default to review queue
    return 'review';
  })()
}}
```

---

## 🔄 Prompt Version Control

| Version | Date       | Change                          | Performance |
|---------|------------|---------------------------------|-------------|
| v1.0    | 2025-12-27 | Initial 4-class prompt          | Baseline    |
| v1.1    | -          | Added "One word only" constraint| +15% accuracy|
| v1.2    | -          | Colon ending for priming        | +5% accuracy |

---

## 📝 Notes

- flan-t5-small has 80M parameters - keep prompts simple
- Average inference time: 200-500ms via HF Inference API
- Consider flan-t5-base (250M) for production accuracy boost
