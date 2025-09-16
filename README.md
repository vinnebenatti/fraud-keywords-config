# 📘 Keyword Groups Configuration (`keyword_groups.json`)

This file defines the **keyword detection rules** used by the [`keyword-alert-service`](../keyword-alert-service).  
Each rule (or **group**) describes **which messages should trigger alerts**, **where to send them**, and **how to avoid false positives**.

---

## 📂 File Structure

```jsonc
{
  "version": "2025-09-12T00:00:00Z",
  "default": {
    "min_matches": "all",
    "case_insensitive": true,
    "strip_accents": true,
    "strip_urls": true,
    "enabled": true,
    "lookback_minutes": 5
  },
  "groups": [
    {
      "id": "crypto_linked_accounts",
      "description": "coin + linked + account in the same message",
      "keywords": ["coin", "linked", "account"],
      "min_matches": 3
    },
    {
      "id": "card_refund",
      "keywords": ["refund", "chargeback", "dispute"],
      "min_matches": 2,
      "exclusions": ["support", "official channel"]
    }
  ]
}
```

---

## 🧩 Available Fields

### 🔹 **`default` level**
Global configuration applied to all groups (can be overridden inside each group):

- `min_matches`: minimum number of keywords that must match (`"all"` or a number).
- `case_insensitive`: ignore case differences.
- `strip_accents`: remove accents (`á → a`, `ç → c`, etc.).
- `strip_urls`: remove URLs before analysis.
- `lookback_minutes`: time window in minutes used in ClickHouse queries.

---

### 🔹 **Each `group`**
Defines a specific rule:

- `id` *(string)* → unique identifier for the group.
- `description` *(optional)* → description of what the rule detects.
- `keywords` *(array)* → list of keywords to search for.
- `min_matches` *(optional)* → minimum number of keywords that must appear.
- `exclusions` *(optional)* → list of words that **invalidate** a match if present.

---

## ✅ Practical Examples

### 1. Detect phishing with “verification code”
```json
{
  "id": "phishing_code",
  "description": "Detects messages containing code + verification + share",
  "keywords": ["code", "verification", "share"],
  "min_matches": 3
}
```

📌 If a message contains all three words, it generates an alert in `#alert-keyword-group-detection`.

---

### 2. Detect suspicious gambling links
```json
{
  "id": "gambling_links",
  "keywords": ["bet", "gambling", "casino"],
  "min_matches": 2,
  "exclusions": ["official", "approved"]
}
```

📌 If at least 2 of these words appear, an alert is triggered **unless** the text contains “official” or “approved”.

🔔 **Tips:**  
- Use `exclusions` to reduce false positives.  
- Combine keywords with flexible `min_matches` (e.g., 2 out of 4).  
- Always give each group a unique `id` to facilitate deduplication.
