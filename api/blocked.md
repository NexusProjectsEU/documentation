---
title: Blocked User Check
description: Integrate ProteX blacklist checks into your Discord bot or game server
order: 1
---

# Blocked User Check Integration

Learn how to integrate ProteX blacklist checks into your Discord bot or server to automatically detect and handle blocked users.

---

## Overview

The `/api/blocked/{discordId}` endpoint allows you to check if a user is on the ProteX blacklist. This is useful for:
- Auto-banning toxic users when they join your server
- Preventing blocked users from using certain features
- Displaying warnings to moderators
- Syncing bans across multiple servers

**Endpoint:** `GET /api/blocked/{discordId}`
**Authentication:** Bearer token required
**Full documentation:** [https://nxpdev.dk/api-docs](https://nxpdev.dk/api-docs)

---

## Discord Bot Integration

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

## Advanced Example: Logging & Notifications

```javascript
client.on("guildMemberAdd", async (member) => {
  const result = await isProtexBlacklisted(member.id);

  if (result.blocked) {
    const modChannel = member.guild.channels.cache.get("MOD_CHANNEL_ID");
    await modChannel.send({
      embeds: [{
        title: "🚫 Blocked User Detected",
        description: `${member.user.tag} is on the ProteX blacklist`,
        fields: [
          { name: "User", value: `<@${member.id}>`, inline: true },
          { name: "Entries", value: result.totalActiveEntries.toString(), inline: true },
          { name: "Blacklist ID", value: result.blacklistId, inline: true }
        ],
        color: 0xFF0000,
        timestamp: new Date()
      }]
    });

    await member.ban({
      reason: `ProteX Blacklist (${result.totalActiveEntries} active entries)`
    });
  }
});
```

---

## FiveM Integration

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

---

## Best Practices

### 1. Use Caching
Cache results for 5 minutes to avoid hitting rate limits:

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

### 2. Handle Errors Gracefully
Always have a fallback if the API is unavailable:

```javascript
async function isProtexBlacklisted(userId) {
  try {
    const res = await fetch(`${BASE_URL}/api/blocked/${userId}`, {
      headers: { "Authorization": `Bearer ${PROTEX_API_KEY}` },
      timeout: 5000
    });

    if (!res.ok) {
      console.error(`ProteX API error: ${res.status}`);
      return { blocked: false };
    }

    return await res.json();
  } catch (error) {
    console.error("ProteX API unavailable:", error);
    return { blocked: false };
  }
}
```

### 3. Log Actions
Keep track of all automated actions for audit purposes:

```javascript
if (result.blocked) {
  console.log(`[ProteX] Banned user ${member.id} - ${result.totalActiveEntries} entries`);

  await db.bans.create({
    userId: member.id,
    reason: "ProteX Blacklist",
    entries: result.totalActiveEntries,
    timestamp: new Date()
  });
}
```

---

## Response Format

### User is Blocked
```json
{
  "blocked": true,
  "discordId": "123456789012345678",
  "blacklistId": "blk_abc123",
  "username": "ToxicUser#1234",
  "totalActiveEntries": 2
}
```

### User is Not Blocked
```json
{
  "blocked": false,
  "discordId": "123456789012345678",
  "message": "User is not blocked"
}
```

---

## Next Steps

- Explore all endpoints at [https://nxpdev.dk/api-docs](https://nxpdev.dk/api-docs)
- Join our [Discord Server](https://discord.gg/protex) for support
- Check out the [FiveM script](https://github.com/NexusProjectsEU/ProteX-FiveM) for game servers