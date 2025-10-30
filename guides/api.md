---
title: API Integration
description: Create and use API keys to integrate ProteX blacklist checks into your bot or application
category: guides
order: 1
---

# API Integration Guide

This guide walks you step-by-step through creating an API key and integrating ProteX blacklist checks into your bot or application.

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

```bash
curl -H "Authorization: Bearer sk_••••••••••••••••••••••••••••" \
  "https://nxpdev.dk/api/blocked/123456789012345678"
```

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

## 3. Integrate Into Your Bot

### Discord.js (JavaScript/TypeScript)

```javascript
const PROTEX_API_KEY = process.env.PROTEX_API_KEY;
const BASE_URL = "https://nxpdev.dk";

async function isProtexBlacklisted(userId) {
  try {
    const res = await fetch(`${BASE_URL}/api/blocked/${userId}`, {
      headers: { "Authorization": `Bearer ${PROTEX_API_KEY}` }
    });
    return await res.json();
  } catch {
    return { blocked: false };
  }
}

client.on("guildMemberAdd", async (member) => {
  const result = await isProtexBlacklisted(member.id);
  
  if (result.blocked) {
    await member.ban({
      reason: `ProteX Blacklist (${result.totalActiveEntries} entries)`
    });
  }
});
```

### Python (discord.py)

```python
import aiohttp
import os

API_KEY = os.getenv("PROTEX_API_KEY")
BASE_URL = "https://nxpdev.dk"

async def check_protex(user_id: str):
    async with aiohttp.ClientSession() as session:
        async with session.get(
            f"{BASE_URL}/api/blocked/{user_id}",
            headers={"Authorization": f"Bearer {API_KEY}"}
        ) as resp:
            return await resp.json()

@bot.event
async def on_member_join(member):
    data = await check_protex(str(member.id))
    if data.get("blocked"):
        await member.ban(reason="ProteX Blacklist")
```

---

## 4. Rate Limits

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

## 5. Troubleshooting

| Error | Solution |
|------|----------|
| `Invalid API key` | Create a new key and copy correctly |
| `Rate limit exceeded` | Wait 60 seconds or implement caching |
| `blocked: false` | The user is **not** blacklisted |

---

## 6. API Reference

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

---

## 7. Regenerate or Delete Key

1. Go to **API Keys**
2. Click **Regenerate** for a new key
3. Click **Delete** to remove

> **Warning**: Old keys stop working immediately.

---

## 8. FiveM Integration

For **FiveM servers**, we provide a ready-to-use script that automatically checks players against the ProteX blacklist on connection.

### Features
- **Automatic player checking** on server join
- **Configurable kick messages** for blocked players
- **Built-in caching** to respect rate limits
- **Easy installation** and configuration

### Installation
1. Download the script from [GitHub](https://github.com/NexusProjectsEU/ProteX-FiveM)
2. Extract to your `resources` folder
3. Add your API key to the configuration file
4. Add `ensure protex` to `server.cfg`

### GitHub Repository
```
https://github.com/NexusProjectsEU/ProteX-FiveM
```

> Free and open-source. Contributions welcome!