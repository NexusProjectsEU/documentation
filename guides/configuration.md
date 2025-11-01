---
title: Configuration
description: Learn how to configure ProteX for your server
category: guides
order: 0
---

# Configuration Guide

This guide will walk you through configuring ProteX to tailor its security features to your server's specific needs.

---

## 1. Access the Configuration Panel

1. **Sign In**: Log in to your ProteX Dashboard using Discord at [nxpdev.dk/dashboard/server](https://nxpdev.dk/dashboard/server)
2. **Select Your Server**: From the dashboard, choose the server you wish to configure
3. **Navigate to Settings**: Click the **"Configure Server"** button next to your server

---

## 2. Server Settings

### Broadcast Channel
Choose a channel where ProteX will send block announcements and updates.

- **Purpose**: Receives notifications when blocked users are detected or removed
- **Options**: Select any text channel from your server, or choose "None" to disable
- **Recommended**: Create a dedicated channel like `#protex-logs` for better organization

### Error Log Channel
Select a channel for error logs and system notifications.

- **Purpose**: Receives technical errors and system alerts
- **Options**: Select any text channel from your server, or choose "None" to disable
- **Recommended**: Use a staff-only channel to keep error logs private

### Action on Block
Choose what action ProteX should take when a blocked user joins your server.

| Action | Description |
|--------|-------------|
| **None - Just log** | Only log the detection without taking action |
| **Kick from server** | Automatically kick the blocked user |
| **Ban from server** | Automatically ban the blocked user (Recommended) |

> **Note**: Ensure ProteX has the necessary permissions (`Kick Members` and `Ban Members`) to enforce these actions.

---

## 3. Save Your Configuration

1. Review all your settings carefully
2. Click the **"Save Configuration"** button at the bottom
3. You'll see a success message when your settings are saved
4. Click **"Cancel"** to return to the server list without saving

---

## 4. View Audit Log

Switch to the **"Audit"** tab to view:
- Recent configuration changes
- Moderation actions taken by ProteX
- Who made each change and when
- Detailed metadata for each action

### Audit Log Features
- Click **"Refresh"** to update the audit log
- Each entry shows:
  - Action type with color-coded badge
  - Actor (who made the change)
  - Discord ID of the actor
  - Timestamp
  - Detailed metadata in JSON format

---

## 5. Test Your Configuration

After saving your configuration:

1. **Check Permissions**: Verify ProteX has the required Discord permissions
2. **Monitor Audit Log**: Check the audit tab to confirm your changes were recorded
3. **Verify Actions**: Test with a known blocked user (if safe to do so)

---

## 6. Troubleshooting

| Issue | Solution |
|-------|----------|
| Configuration not saving | Refresh the page and try again, or check your internet connection |
| Channels not appearing | Ensure ProteX has "View Channels" permission for those channels |
| Actions not working | Verify ProteX has `Kick Members` and `Ban Members` permissions |
| No audit entries | Configuration changes may take a few moments to appear in the log |
| "Failed to load configuration" error | Ensure you have administrator permissions on the server |

---

## 7. Best Practices

### Channel Setup
- ✅ Create dedicated channels for ProteX logs
- ✅ Restrict log channels to staff members only
- ✅ Use different channels for broadcasts and errors
- ✅ Enable channel history so you can review past events

### Action Configuration
- ✅ Use **"Ban from server"** for maximum protection
- ✅ Start with **"None"** if testing the system
- ✅ Monitor the broadcast channel regularly
- ✅ Review audit logs to track system activity

### Security
- ✅ Only give configuration access to trusted administrators
- ✅ Regularly review audit logs for unauthorized changes
- ✅ Keep error logs private to prevent information leakage
- ✅ Test configuration changes in a safe environment first

---

## Need Help?

If you encounter issues or have questions about configuration:
- Check the [Troubleshooting](#6-troubleshooting) section above
- Join our [Discord server](https://discord.gg/cBp3tee8hJ) for support