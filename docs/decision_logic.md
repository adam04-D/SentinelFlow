# OmniSentinel - Decision Logic Documentation

## IF Node Architecture

The decision logic uses a **cascading gate pattern** where content flows through multiple checkpoints.

---

## 🚦 Gate Flow Diagram

```
                    ┌────────────────────┐
                    │   HF Classification │
                    │   (hate/spam/unsafe/clean)
                    └─────────┬──────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │      GATE 1: BLOCKED CHECK    │
              │  IF classification contains   │
              │     'hate' OR 'unsafe'        │
              └───────────────┬───────────────┘
                     │                 │
                   TRUE              FALSE
                     │                 │
                     ▼                 ▼
              ┌─────────────┐  ┌───────────────────────────────┐
              │   BLOCKED   │  │      GATE 2: REVIEW CHECK     │
              │  (403)      │  │  IF classification contains   │
              └─────────────┘  │         'spam'                │
                               └───────────────┬───────────────┘
                                      │                 │
                                    TRUE              FALSE
                                      │                 │
                                      ▼                 ▼
                               ┌─────────────┐  ┌─────────────┐
                               │   REVIEW    │  │    CLEAN    │
                               │   (202)     │  │    (200)    │
                               └─────────────┘  └─────────────┘
```

---

## 📋 IF Node Configurations

### Gate 1: Blocked Content Check

**Node Type:** IF  
**Purpose:** Catch severe violations immediately

```yaml
Conditions:
  - Mode: OR (Any match triggers)
  
  Condition 1:
    Left Value: {{ $json.classification }}
    Operation: Contains
    Right Value: hate
    
  Condition 2:
    Left Value: {{ $json.classification }}
    Operation: Contains
    Right Value: unsafe
```

**n8n Settings:**
```json
{
  "conditions": {
    "options": {
      "caseSensitive": false,
      "typeValidation": "loose"
    },
    "conditions": [
      {
        "leftValue": "={{ $json.classification }}",
        "rightValue": "hate",
        "operator": {
          "type": "string",
          "operation": "contains"
        }
      },
      {
        "leftValue": "={{ $json.classification }}",
        "rightValue": "unsafe",
        "operator": {
          "type": "string",
          "operation": "contains"
        }
      }
    ],
    "combinator": "or"
  }
}
```

---

### Gate 2: Review Queue Check

**Node Type:** IF  
**Purpose:** Flag borderline content for human review

```yaml
Conditions:
  - Mode: AND (All must match)
  
  Condition 1:
    Left Value: {{ $json.classification }}
    Operation: Contains
    Right Value: spam
```

**n8n Settings:**
```json
{
  "conditions": {
    "options": {
      "caseSensitive": false,
      "typeValidation": "loose"
    },
    "conditions": [
      {
        "leftValue": "={{ $json.classification }}",
        "rightValue": "spam",
        "operator": {
          "type": "string",
          "operation": "contains"
        }
      }
    ],
    "combinator": "and"
  }
}
```

---

## 🏷️ Result Tagging (Edit Fields Nodes)

### Result: BLOCKED

```json
{
  "status": "BLOCKED",
  "reason": "{{ $json.classification }}",
  "action": "Content rejected - violates community guidelines",
  "code": 403
}
```

### Result: REVIEW

```json
{
  "status": "REVIEW",
  "reason": "{{ $json.classification }}",
  "action": "Content flagged for human review",
  "code": 202
}
```

### Result: CLEAN

```json
{
  "status": "CLEAN",
  "reason": "{{ $json.classification }}",
  "action": "Content approved - no issues detected",
  "code": 200
}
```

---

## 🔀 Connection Logic

### Understanding n8n IF Node Outputs

```
IF Node
  ├── Output 0 (TRUE)  → Condition matched
  └── Output 1 (FALSE) → Condition not matched
```

### Wiring Pattern

```
Parse HF Response
  ├──────────────────────▶ Gate 1 (Blocked Check)
  │                            ├── TRUE  → Result: BLOCKED
  │                            └── FALSE → Result: CLEAN
  │
  └──────────────────────▶ Gate 2 (Review Check)
                               ├── TRUE  → Result: REVIEW
                               └── FALSE → (not connected)
```

**Important:** Both gates receive the same input simultaneously (parallel execution).

---

## 🎛️ Advanced: Priority-Based Routing

For stricter control, use **sequential gates**:

```
Parse Response
     │
     ▼
┌─────────────┐
│ Gate: Hate? │
└──────┬──────┘
   │       │
  YES      NO
   │       │
   ▼       ▼
BLOCK  ┌─────────────┐
       │Gate: Unsafe?│
       └──────┬──────┘
          │       │
         YES      NO
          │       │
          ▼       ▼
       BLOCK  ┌─────────────┐
              │ Gate: Spam? │
              └──────┬──────┘
                 │       │
                YES      NO
                 │       │
                 ▼       ▼
              REVIEW   CLEAN
```

**Benefit:** Clear priority hierarchy  
**Drawback:** More nodes, slower execution

---

## 📊 Decision Matrix

| Classification | Gate 1 Result | Gate 2 Result | Final Status |
|----------------|---------------|---------------|--------------|
| `hate`         | TRUE          | FALSE         | BLOCKED      |
| `unsafe`       | TRUE          | FALSE         | BLOCKED      |
| `spam`         | FALSE         | TRUE          | REVIEW       |
| `clean`        | FALSE         | FALSE         | CLEAN        |
| `error`        | FALSE         | FALSE         | CLEAN*       |

*Consider adding an error handling gate for production

---

## 🐛 Debugging Tips

### 1. Check Classification Value

Add a "Code" node to log the classification:
```javascript
console.log('Classification:', $json.classification);
return $json;
```

### 2. Test Each Gate Independently

1. Disable all connections except Gate 1
2. Send test input with "hate" content
3. Verify TRUE branch activates
4. Repeat for Gate 2 with "spam" content

### 3. Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Always goes to CLEAN | Classification not extracted | Check Parse Response node |
| Never matches | Case sensitivity | Set `caseSensitive: false` |
| Multiple outputs fire | Parallel connections | Use Merge node to combine |

---

## 🔧 Expression Reference

### Get classification from HF response
```javascript
{{ $json[0]?.generated_text?.toLowerCase().trim() }}
```

### Reference previous node data
```javascript
{{ $('Node Name').item.json.fieldName }}
```

### Conditional default
```javascript
{{ $json.classification || 'unknown' }}
```

### Check if contains any keyword
```javascript
{{ ['hate', 'unsafe'].some(k => $json.classification.includes(k)) }}
```
