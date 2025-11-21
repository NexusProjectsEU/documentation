---
title: N-Boks
description: N-Boks is an invoicing system that is primarily used on Danish servers.
order: 0
---

# N-Boks Documentation

A modern invoice and fine management system for FiveM with support for multiple frameworks.

---

## Installation

1. Place the `nboks` folder in your server's `resources` directory
2. Add `ensure nboks` to your `server.cfg`
3. Import the SQL file to create the required database table
4. Configure `sh_config.lua` and `sv_config.lua` to match your server setup

---

## Configuration

### Shared Config (`sh_config.lua`)

```lua
nboks.sh.framework = "Qbox"  -- Available: "vRP", "ESX", "QBCore", "Qbox"
nboks.sh.language = "en"      -- Available: "da", "en"
nboks.sh.currency = "€"       -- Currency symbol (e.g., "kr.", "$", "€", "£")
```

**Parameters:**
- `framework` - Your server's framework (vRP, ESX, QBCore, or Qbox)
- `language` - UI language (Danish or English)
- `currency` - Currency symbol displayed throughout the UI

### Server Config (`sv_config.lua`)

```lua
nboks.sv.discordwebhook = ""  -- Discord webhook URL for logging

nboks.sv.reminderSystem = {
    enabled = true,           -- Enable/disable reminder system
    feePercent = 10,         -- Fee percentage per reminder (%)
    checkInterval = 24,      -- Hours between reminder checks
    maxReminders = 3,        -- Maximum reminders (0 = unlimited)
    applyToFinesOnly = false -- Only apply reminders to fines
}
```

**Discord Webhook:** Logs invoice creation, payments, and reminder fees

**Reminder System:** Automatically adds late fees to unpaid invoices

---

## Database Setup

Execute this SQL to create the `invoices` table:

```sql
CREATE TABLE IF NOT EXISTS `invoices` (
  `id` varchar(50) NOT NULL,
  `identifier` varchar(50) NOT NULL,
  `sender` varchar(100) NOT NULL,
  `senderidentifier` varchar(50) DEFAULT NULL,
  `type` varchar(20) NOT NULL,
  `amount` int(11) NOT NULL,
  `createdAt` datetime NOT NULL,
  `description` text DEFAULT NULL,
  `isPaid` tinyint(1) NOT NULL DEFAULT 0,
  `lastReminderCheck` datetime DEFAULT NULL,
  `reminderCount` int(11) DEFAULT 0,
  `reminderAudit` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `identifier` (`identifier`),
  KEY `isPaid` (`isPaid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## Commands

### `/nboks`
Opens the invoice UI for the player.

**Usage:**
```lua
/nboks
```

---

## Exports

### sendInvoice

Sends an invoice or fine to a player.

**Syntax:**
```lua
exports['nboks']:sendInvoice(identifier, sender, senderidentifier, invoiceType, amount, description)
```

**Parameters:**
- `identifier` (string) - Target player's identifier (user_id for vRP, identifier for ESX, citizenid for QB/Qbox)
- `sender` (string) - Name of the sender (e.g., "Police Department", "City Hall")
- `senderidentifier` (string) - Sender's identifier (optional, can be nil)
- `invoiceType` (string) - Type of invoice: `"fine"` or `"invoice"`
- `amount` (number) - Invoice amount
- `description` (string) - Description of the invoice (optional)

**Returns:** Creates invoice in database and notifies the player if online

---

### forcePayInvoice

Forces payment of an invoice, removing money from the player's bank.

**Syntax:**
```lua
local success = exports['nboks']:forcePayInvoice(identifier, invoiceId)
```

**Parameters:**
- `identifier` (string) - Player's identifier
- `invoiceId` (string) - Invoice ID to pay

**Example:**
```lua
local paid = exports['nboks']:forcePayInvoice(user_id, "INV_1234567890_5678")
if paid then
    print("Invoice paid successfully")
else
    print("Payment failed - player offline or insufficient funds")
end
```

**Returns:** `true` if payment successful, `false` otherwise

---

### checkReminders

Manually triggers reminder fee checks for a player.

**Syntax:**
```lua
exports['nboks']:checkReminders(identifier)
```

**Parameters:**
- `identifier` (string) - Player's identifier

**Example:**
```lua
-- Check and apply reminder fees for a specific player
exports['nboks']:checkReminders(user_id)
```

**Note:** Reminders are automatically checked when players spawn if the reminder system is enabled.

---

### chargePlayer

Sends an invoice AND immediately charges the player. The invoice is created as already paid.

**Syntax:**
```lua
local result = exports['nboks']:chargePlayer(identifier, sender, senderidentifier, invoiceType, amount, description)
```

**Parameters:**
- `identifier` (string) - Target player's identifier
- `sender` (string) - Name of the sender
- `senderidentifier` (string) - Sender's identifier (optional, can be nil)
- `invoiceType` (string) - Type of invoice: `"fine"` or `"invoice"`
- `amount` (number) - Amount to charge
- `description` (string) - Description (optional)

**Returns:** A table with the result:

```lua
-- Success:
{ success = true, reason = "paid", message = "Payment successful", invoice = {...} }

-- Player not online:
{ success = false, reason = "player_offline", message = "Player is not online" }

-- Not enough money:
{ success = false, reason = "insufficient_funds", message = "Player has insufficient funds", balance = 5000, required = 10000 }

-- Payment failed:
{ success = false, reason = "payment_failed", message = "Failed to remove money from player" }
```

**Example:**
```lua
local result = exports['nboks']:chargePlayer(
    identifier,
    "Hospital",
    nil,
    "invoice",
    2500,
    "Medical treatment"
)

if result.success then
    print("Charged successfully! Invoice ID: " .. result.invoice.id)
else
    if result.reason == "insufficient_funds" then
        print("Player only has " .. result.balance .. " but needs " .. result.required)
    elseif result.reason == "player_offline" then
        print("Player is not online")
    end
end
```

**Note:** Unlike `sendInvoice`, this export requires the player to be online and have sufficient funds.

---

## Events

### Server Events

#### `nboks:server:getInvoices`
Triggered by the client to retrieve all invoices for a player.

**Usage:**
```lua
TriggerServerEvent("nboks:server:getInvoices")
```

**Returns:** Triggers `nboks:client:setInvoices` with invoice data

---

#### `nboks:server:payInvoice`
Triggered when a player attempts to pay an invoice.

**Usage:**
```lua
TriggerServerEvent("nboks:server:payInvoice", invoiceId, invoiceType, amount)
```

**Parameters:**
- `invoiceId` (string) - Invoice ID
- `invoiceType` (string) - Type ("fine" or "invoice")
- `amount` (number) - Amount to pay

**Result:** Triggers either `nboks:client:paymentSuccess` or `nboks:client:paymentFailed`

---

### Client Events

#### `nboks:client:openUI`
Opens the invoice UI for the player.

**Usage:**
```lua
TriggerClientEvent("nboks:client:openUI", source)
```

**Example:**
```lua
-- Open UI for specific player
RegisterCommand("showboks", function(source)
    TriggerClientEvent("nboks:client:openUI", source)
end, false)
```

---

#### `nboks:client:setInvoices`
Sets the invoice list in the UI.

**Usage:**
```lua
TriggerClientEvent("nboks:client:setInvoices", source, invoices)
```

**Parameters:**
- `invoices` (table) - Array of invoice objects

**Invoice Object Structure:**
```lua
{
    id = "INV_1234567890_5678",
    sender = "Police Department",
    type = "fine",
    amount = 5000,
    createdAt = "2025-01-15 14:30:00",
    description = "Speeding violation",
    isPaid = false,
    reminderAudit = {}
}
```

---

#### `nboks:client:addInvoice`
Adds a new invoice to the UI (real-time update).

**Usage:**
```lua
TriggerClientEvent("nboks:client:addInvoice", source, invoice)
```

**Parameters:**
- `invoice` (table) - Invoice object

---

#### `nboks:client:paymentSuccess`
Notifies the client that payment was successful.

**Usage:**
```lua
TriggerClientEvent("nboks:client:paymentSuccess", source, invoiceId)
```

**Parameters:**
- `invoiceId` (string) - ID of paid invoice

---

#### `nboks:client:paymentFailed`
Notifies the client that payment failed.

**Parameters:**
- `invoiceId` (string) - ID of failed invoice

---

#### `nboks:client:updateInvoiceAmount`
Updates an invoice amount (used by reminder system).

**Parameters:**
- `invoiceId` (string) - Invoice ID
- `newAmount` (number) - New invoice amount

---

## NUI Callbacks

### close
Triggered when the player closes the UI.

**Client-side handling:**
```lua
RegisterNUICallback("close", function(_, cb)
    closeInvoiceUI()
    cb("ok")
end)
```

---

### payInvoice
Triggered when the player clicks the pay button.

**Client-side handling:**
```lua
RegisterNUICallback("payInvoice", function(data, cb)
    TriggerServerEvent("nboks:server:payInvoice", data.id, data.type, data.amount)
    cb("ok")
end)
```

**Data sent from NUI:**
```javascript
{
    id: "INV_1234567890_5678",
    type: "fine",
    amount: 5000
}
```

---

## Integration Examples

### Police Tablet Integration (Omik Polititablet)

```lua
RegisterNetEvent("omik_polititablet:newBillingEvent")
AddEventHandler("omik_polititablet:newBillingEvent", function(data)
    local identifier = data.targetCitizen.identifier
    local amount = data.fineAmount
    local sender = "Politiet"
    local invoiceType = "fine"

    local description = "Politibøde"
    if data.rawData then
        local titles = {}
        for _, ticketData in pairs(data.rawData) do
            if ticketData.ticket and ticketData.ticket.title then
                table.insert(titles, ticketData.ticket.title)
            end
        end
        if #titles > 0 then
            description = table.concat(titles, ", ")
        end
    end

    exports['nboks']:sendInvoice(identifier, sender, nil, invoiceType, amount, description)
end)
```

---

### Custom Job Integration

```lua
-- Mechanic shop invoice example
RegisterCommand("mechanicbill", function(source, args)
    local targetId = tonumber(args[1])
    local amount = tonumber(args[2])
    local description = table.concat(args, " ", 3)

    if not targetId or not amount then
        TriggerClientEvent('chat:addMessage', source, {
            args = {"System", "Usage: /mechanicbill [playerid] [amount] [description]"}
        })
        return
    end

    local targetIdentifier = getPlayerIdentifier(targetId)

    if targetIdentifier then
        exports['nboks']:sendInvoice(
            targetIdentifier,
            "Mechanic Shop",
            nil,
            "invoice",
            amount,
            description
        )

        TriggerClientEvent('chat:addMessage', source, {
            args = {"System", "Invoice sent successfully!"}
        })
    end
end, false)
```

---

### Government Tax System

```lua
CreateThread(function()
    while true do
        Wait(2592000000)

        local players = getAllPlayers()
        for _, playerId in pairs(players) do
            local identifier = getPlayerIdentifier(playerId)

            exports['nboks']:sendInvoice(
                identifier,
                "Tax Authority",
                nil,
                "invoice",
                5000,
                "Monthly property tax"
            )
        end
    end
end)
```

---

## Reminder System

The reminder system automatically adds late fees to unpaid invoices.

### How It Works

1. **Trigger:** Checks run when players spawn or when manually triggered
2. **Interval:** Fees apply every X hours (configured in `checkInterval`)
3. **Fee Calculation:** `feeAmount = currentAmount * (feePercent / 100)`
4. **Maximum Reminders:** Stops after reaching `maxReminders` (0 = unlimited)
5. **Audit Trail:** All fee applications are logged in `reminderAudit`

### Configuration Example

```lua
nboks.sv.reminderSystem = {
    enabled = true,
    feePercent = 10,        -- 10% fee per reminder
    checkInterval = 24,     -- Check every 24 hours
    maxReminders = 3,       -- Max 3 reminders (0 = unlimited)
    applyToFinesOnly = true -- Only fines, not invoices
}
```

### Example Reminder Flow

**Initial invoice:** 1000 kr

After 24 hours (Reminder 1):
- Fee: 1000 × 10% = 100 kr
- New amount: 1100 kr

After 48 hours (Reminder 2):
- Fee: 1100 × 10% = 110 kr
- New amount: 1210 kr

After 72 hours (Reminder 3):
- Fee: 1210 × 10% = 121 kr
- New amount: 1331 kr (max reminders reached)

---

## Discord Webhook

Configure webhook logging in `sv_config.lua`:

```lua
nboks.sv.discordwebhook = "https://discord.com/api/webhooks/..."
```

### Logged Events

1. **Invoice/Fine Sent**
   - Receiver information
   - Sender information
   - Amount and description
   - Invoice ID and timestamp

2. **Payment Success**
   - Player information
   - Amount paid
   - Invoice ID

3. **Reminder Fee Applied**
   - Player information
   - Reminder count
   - Fee percentage and amount
   - Old and new amounts

4. **Player Charged (chargePlayer)**
   - Player information
   - Sender information
   - Amount charged
   - Invoice ID and description

### Webhook Colors

- **Invoice Sent (Fine):** Red (15158332)
- **Invoice Sent (Invoice):** Blue (3447003)
- **Payment Success:** Green (3066993)
- **Player Charged:** Green (3066993)
- **Reminder Applied:** Yellow (16776960)

---

## Framework Support

### Supported Frameworks

- ✅ **vRP** - Uses user_id as identifier
- ✅ **ESX** - Uses identifier
- ✅ **QBCore** - Uses citizenid
- ✅ **Qbox** - Uses citizenid

### Framework-Specific Callbacks

**ESX:**
```lua
ESX.TriggerServerCallback("nboks:server:getInvoices", function(invoices)
    -- Handle invoices
end)
```

**QBCore:**
```lua
QBCore.Functions.TriggerCallback("nboks:server:getInvoices", function(invoices)
    -- Handle invoices
end)
```