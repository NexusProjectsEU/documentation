---
title: Introduction
description: Getting started with the ProteX API - create API keys and start integrating
order: 0
---

# API Introduction

Welcome to the ProteX API! This guide will help you get started with creating API keys and making your first requests.

---

## Complete API Documentation

For a complete reference of all available API endpoints, parameters, and responses, visit our interactive API documentation:

**[https://nxpdev.dk/api-docs](https://nxpdev.dk/api-docs)**

This documentation shows all endpoints with:
- Detailed parameter descriptions
- Request/response examples

---

## 1. Create an API Key

### Step 1: Go to API Keys
1. Log in to the **ProteX Dashboard**
2. Click **"API Keys"** in the menu
3. Press **"Create New API Key"**

### Step 2: Name Your Key
```
Key Name: "Production Bot"
```
> Use clear names like "Dev Bot", "Main Server", etc.

### Step 3: Copy the Key **IMMEDIATELY**
```
sk_••••••••••••••••••••••••••••
```
> **You can only view the full key once!**

### Step 4: Store It Securely
```env
PROTEX_API_KEY=sk_••••••••••••••••••••••••••••
```
> Save in `.env`, password manager, or server environment variables.

---

## 2. Test Your API Key

Test your API key with a simple blacklist check:

```bash
curl -H "Authorization: Bearer sk_••••••••••••••••••••••••••••" \ "https://nxpdev.dk/api/blocked/123456789012345678"
```

> **Try other endpoints**: Use the [API docs](https://nxpdev.dk/api-docs) to test all available endpoints interactively.

### Response (Not Blocked)
```json
{
  "blocked": false,
  "discordId": "123456789012345678",
  "message": "User is not blocked"
}
```

### Response (Blocked)
```json
{
  "blocked": true,
  "discordId": "123456789012345678",
  "blacklistId": "blk_...",
  "username": "ToxicUser#1234",
  "totalActiveEntries": 2
}
```

---

## 3. Rate Limits

| Limit | Window | Action |
|-------|--------|--------|
| 100 requests | 60 seconds | Standard for all keys |

> **Tip**: Cache results for 5 minutes to reduce calls.

```javascript
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000;

async function checkWithCache(userId) {
  const key = `bl_${userId}`;
  const cached = cache.get(key);

  if (cached && cached.expires > Date.now()) return cached.data;

  const data = await isProtexBlacklisted(userId);
  cache.set(key, { data, expires: Date.now() + CACHE_TTL });
  return data;
}
```

---

## 4. Troubleshooting

| Error | Solution |
|------|----------|
| `Invalid API key` | Create a new key and copy correctly |
| `Rate limit exceeded` | Wait 60 seconds or implement caching |
| `blocked: false` | The user is **not** blacklisted |

---

## 5. API Reference

For the full API reference with all available endpoints, visit:

**[https://nxpdev.dk/api-docs](https://nxpdev.dk/api-docs)**

### Quick Example: Check Blacklist Status

```
GET /api/blocked/{discordId}
Authorization: Bearer {your_key}
```

**Response:**
```json
{
  "blocked": true,
  "discordId": "123456789012345678",
  "username": "User#0001",
  "totalActiveEntries": 1
}
```

> **See more endpoints**: Visit the [API docs](https://nxpdev.dk/api-docs) for user management, server settings, webhooks, and more.

---

## 6. Manage Your Keys

### Regenerate a Key
1. Go to **API Keys**
2. Click **Regenerate** for a new key
3. Copy the new key immediately

### Delete a Key
1. Go to **API Keys**
2. Click **Delete** to remove
3. Confirm deletion

> **Warning**: Old keys stop working immediately after regeneration or deletion.
