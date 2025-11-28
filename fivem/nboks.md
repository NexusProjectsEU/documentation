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
3. Import the SQL file to create the required database tables
4. Configure your license key in `sv_config.lua`
5. Configure `sh_config.lua` and `sv_config.lua` to match your server setup

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
-- License Key (required)
nboks.sv.licensekey = "INSERT-LICENSE"

-- Discord Webhooks (optional)
nboks.sv.discordwebhook = {
    adminLogs = "",    -- Webhook URL for admin activity logs
    paymentLogs = ""   -- Webhook URL for payment logs
}

-- Reminder System
nboks.sv.reminderSystem = {
    enabled = true,          -- Enable/disable reminder system
    feePercent = 10,         -- Fee percentage per reminder (%)
    checkInterval = 24,      -- Hours between reminder checks
    maxReminders = 3,        -- Maximum reminders (0 = unlimited)
    applyToFinesOnly = false -- Only apply reminders to fines
}

-- Admin System
nboks.sv.AdminIdentifiers = {
    ["discord:123456789012345678"] = {
        viewInvoices = true,
        actions = {
            deleteInvoices = true,
            markAsPaid = true,
            markAsUnpaid = true,
            forcePay = true,
            refund = true
        },
        auditLogs = true
    }
}
```

**License Key:** Your unique license key from purchase (required)

**Discord Webhooks:**
- `adminLogs` - Logs admin actions (mark paid, delete, refund, etc.)
- `paymentLogs` - Logs invoice creation, payments, and reminder fees

**Reminder System:** Automatically adds late fees to unpaid invoices

**Admin System:** Configure admin access by player identifiers (discord:, steam:, license:, fivem:, ip:)

---

## Database Setup

Execute this SQL to create the required tables:

```sql
CREATE TABLE IF NOT EXISTS `nboks_invoices` (
  `id` varchar(50) NOT NULL,
  `identifier` varchar(50) NOT NULL,
  `sender` varchar(100) DEFAULT NULL,
  `senderidentifier` varchar(50) NOT NULL,
  `type` varchar(20) NOT NULL,
  `amount` int(11) NOT NULL,
  `createdAt` DATETIME NOT NULL,
  `description` text DEFAULT NULL,
  `isPaid` tinyint(1) NOT NULL DEFAULT 0,
  `isRefunded` tinyint(1) NOT NULL DEFAULT 0,
  `lastReminderCheck` DATETIME DEFAULT NULL,
  `reminderCount` int(11) NOT NULL DEFAULT 0,
  `reminderAudit` JSON DEFAULT NULL,
  `metadata` JSON DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `identifier` (`identifier`),
  KEY `isPaid` (`isPaid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `nboks_user_settings` (
  `identifier` varchar(50) NOT NULL,
  `primaryColor` varchar(7) DEFAULT '#2c4a5a',
  `secondaryColor` varchar(7) DEFAULT '#1a2e3a',
  `accentColor` varchar(7) DEFAULT '#ef4444',
  `sidebarColor` varchar(7) DEFAULT '#232323',
  `headerGradientStart` varchar(7) DEFAULT '#3d5a6b',
  `headerGradientEnd` varchar(7) DEFAULT '#2c4a5a',
  `contentBackgroundColor` varchar(7) DEFAULT '#ffffff',
  `listItemBackgroundColor` varchar(7) DEFAULT '#f9fafb',
  `updatedAt` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`identifier`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `nboks_audit_logs` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `adminIdentifier` varchar(50) NOT NULL,
  `action` varchar(50) NOT NULL,
  `targetInvoiceId` varchar(50) DEFAULT NULL,
  `targetIdentifier` varchar(50) DEFAULT NULL,
  `details` JSON DEFAULT NULL,
  `createdAt` DATETIME DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `adminIdentifier` (`adminIdentifier`),
  KEY `action` (`action`),
  KEY `createdAt` (`createdAt`)
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

## Admin Panel

The admin panel provides powerful tools for managing invoices across all players.

### Accessing Admin Panel

1. Configure admin access in `sv_config.lua` using player identifiers
2. Players with admin access will see an "Admin Panel" button in the UI
3. The panel shows all invoices in the system with filtering and sorting

### Admin Permissions

```lua
viewInvoices = true,        -- Can view all invoices in the system
actions = {
    deleteInvoices = true,  -- Can delete invoices
    markAsPaid = true,      -- Can mark invoices as paid
    markAsUnpaid = true,    -- Can mark paid invoices as unpaid
    forcePay = true,        -- Can force pay invoices (removes money from player)
    refund = true           -- Can refund paid invoices
},
auditLogs = true  -- Can view audit logs of all admin actions
```

### Admin Actions

**Mark as Paid**
- Marks an unpaid invoice as paid
- Does NOT remove money from the player
- Useful for manual payment verification
- Triggers payment success notification for the player if online

**Mark as Unpaid**
- Marks a paid invoice back to unpaid
- Does NOT refund money to the player
- Triggers payment status update for the player if online

**Force Pay**
- Removes money from the player's bank account
- Marks the invoice as paid

**Refund**
- Requires invoice to be paid
- Requires receiver (original payer) to be online
- Removes money from sender (if online) and returns to receiver
- Marks invoice as unpaid and refunded (`isRefunded = 1`)
- Shows error if sender has insufficient funds

**Delete Invoice**
- Permanently removes invoice from database
- Notifies player if they are online
- Cannot be undone

### Audit Logs

All admin actions are logged in the `nboks_audit_logs` table:
- Admin identifier and name
- Action type (MARK_AS_PAID, DELETE_INVOICE, etc.)
- Target invoice ID and player identifier
- Additional details (amount, type, etc.)
- Timestamp

Admins with `auditLogs = true` can view the complete audit trail in the admin panel.

---

## Exports

### sendInvoice

Sends an invoice or fine to a player.

**Syntax:**
```lua
exports['nboks']:sendInvoice(identifier, sender, senderidentifier, invoiceType, amount, description, metadata)
```

**Parameters:**
- `identifier` (string) - Target player's identifier (user_id for vRP, identifier for ESX, citizenid for QB/Qbox)
- `sender` (string) - Name of the sender (e.g., "Police Department", "City Hall") - **can be `nil`** (will auto-fetch from senderidentifier)
- `senderidentifier` (string) - Sender's identifier
- `invoiceType` (string) - Type of invoice: `"fine"` or `"invoice"`
- `amount` (number) - Invoice amount
- `description` (string) - Description of the invoice (optional)
- `metadata` (table) - Array of breakdown items (optional) - displayed as a table in the UI

**Metadata Structure:**
```lua
{
    { title = "Item description", amount = 1000 },
    { title = "Another item", amount = 500 }
}
```

**Returns:** A result object:

```lua
-- Success:
{ success = true, id = "INV_...", sender = "...", type = "fine", amount = 5000, ... }

-- Validation errors:
{ success = false, reason = "invalid_identifier", message = "Invalid identifier" }
{ success = false, reason = "invalid_sender", message = "Invalid sender" }
{ success = false, reason = "invalid_type", message = "Type must be 'fine' or 'invoice'" }
{ success = false, reason = "invalid_amount", message = "Amount must be a positive number" }
{ success = false, reason = "license_invalid", message = "License validation failed" }
```

**Example with metadata (Police fine with law breakdown):**
```lua
local metadata = {
    { title = "Straffelovens § 119 - Vold mod tjenestemand", amount = 5000 },
    { title = "Færdselslovens § 3 - Hensynsløs kørsel", amount = 2500 },
    { title = "Våbenlovens § 1 - Ulovlig våbenbesiddelse", amount = 7500 }
}

exports['nboks']:sendInvoice(
    identifier,
    "Politiet",         -- Can also be nil
    officerIdentifier,  -- Can also be nil
    "fine",
    15000,
    "Sigtelse for flere lovovertrædelser",
    metadata
)
```

**Auto-fetch sender name:**
If `sender` is `nil` or empty, the system will automatically fetch the sender's name from the `senderidentifier` when displaying invoices.

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

**Returns:** A result object or boolean:

```lua
-- Success:
true

-- Invoice not found or already paid:
false

-- Validation errors:
{ success = false, reason = "invalid_identifier", message = "Invalid identifier" }
{ success = false, reason = "invalid_invoice_id", message = "Invalid invoice ID" }
{ success = false, reason = "license_invalid", message = "License validation failed" }
```

**Example:**
```lua
local result = exports['nboks']:forcePayInvoice(user_id, "INV_1234567890_5678")
if result == true then
    print("Invoice paid successfully")
elseif result == false then
    print("Invoice not found or already paid")
elseif type(result) == "table" and not result.success then
    print("Validation error: " .. result.message)
end
```

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
- `senderidentifier` (string) - Sender's identifier
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

-- Validation errors:
{ success = false, reason = "invalid_identifier", message = "Invalid identifier" }
{ success = false, reason = "invalid_sender", message = "Invalid sender" }
{ success = false, reason = "invalid_type", message = "Type must be 'fine' or 'invoice'" }
{ success = false, reason = "invalid_amount", message = "Amount must be a positive number" }
{ success = false, reason = "license_invalid", message = "License validation failed" }
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

### getInvoices

Retrieves invoices from the database with flexible filtering options.

**Syntax:**
```lua
local result = exports['nboks']:getInvoices(filters)
```

**Filter Parameters (all optional):**
```lua
{
    identifier = "1",                -- Filter by receiver identifier
    sender = "Politiet",             -- Filter by sender name
    senderidentifier = "5",          -- Filter by sender identifier
    type = "fine",                   -- Filter by type: "fine" or "invoice"
    status = "unpaid",               -- Filter by status: "paid", "unpaid", or "all"
    limit = 50,                      -- Max results (default: 100, max: 1000)
    orderBy = "date_desc"            -- Order: "date_desc", "date_asc", "amount_desc", "amount_asc"
}
```

**Security Note:** The `limit` parameter is validated and capped between 1-1000. The `orderBy` parameter uses a whitelist approach to prevent SQL injection.

**Returns:**
```lua
{
    success = true,
    invoices = { ... },  -- Array of invoice objects
    count = 10           -- Number of results
}
```

**Examples:**

```lua
-- Get all unpaid fines from "Politiet"
local result = exports['nboks']:getInvoices({
    sender = "Politiet",
    type = "fine",
    status = "unpaid"
})

if result.success then
    print("Found " .. result.count .. " unpaid fines")
    for _, invoice in ipairs(result.invoices) do
        print(invoice.id .. " - " .. invoice.amount .. " kr.")
    end
end

-- Get all invoices for a specific player
local result = exports['nboks']:getInvoices({
    identifier = playerIdentifier,
    status = "all"
})

-- Get fines sent by a specific officer
local result = exports['nboks']:getInvoices({
    senderidentifier = officerIdentifier,
    type = "fine",
    orderBy = "date_desc",
    limit = 20
})

-- Get highest unpaid invoices
local result = exports['nboks']:getInvoices({
    status = "unpaid",
    orderBy = "amount_desc",
    limit = 10
})
```

**Invoice Object Structure:**
```lua
{
    id = "INV_1234567890_5678",
    identifier = "1",           -- Receiver
    sender = "Politiet",        -- Can be nil
    senderidentifier = "5",
    type = "fine",
    amount = 5000,
    createdAt = "2025-01-15 14:30:00",
    description = "Speeding violation",
    isPaid = false,
    isRefunded = false,
    reminderCount = 2,
    reminderAudit = { ... },
    metadata = {
        { title = "Item 1", amount = 2500 },
        { title = "Item 2", amount = 2500 }
    }
}
```

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
TriggerServerEvent("nboks:server:payInvoice", invoiceId)
```

**Parameters:**
- `invoiceId` (string) - Invoice ID

**Security Note:** The server fetches the invoice amount and type from the database. Never send amount/type from client to prevent mod menu exploits where players could modify the payment amount.

**Result:** Triggers either `nboks:client:paymentSuccess` or `nboks:client:paymentFailed`

---

#### `nboks:server:checkAdmin`
Checks if the player has admin access.

**Usage:**
```lua
TriggerServerEvent("nboks:server:checkAdmin")
```

**Result:** Triggers `nboks:client:setAdmin` and `nboks:client:setAdminPermissions`

---

#### `nboks:server:admin:getAllInvoices`
Retrieves all invoices in the system (admin only).

**Requires:** `viewInvoices` permission

**Usage:**
```lua
TriggerServerEvent("nboks:server:admin:getAllInvoices")
```

**Result:** Triggers `nboks:client:admin:setAllInvoices`

---

#### `nboks:server:admin:markAsPaid`
Marks an invoice as paid without removing money.

**Requires:** `markAsPaid` permission

**Parameters:**
- `invoiceId` (string) - Invoice ID

**Security:** Validates permission, logs action to audit logs, sends webhook notification

---

#### `nboks:server:admin:markAsUnpaid`
Marks a paid invoice as unpaid.

**Requires:** `markAsUnpaid` permission

**Parameters:**
- `invoiceId` (string) - Invoice ID

---

#### `nboks:server:admin:forcePay`
Forces payment of an invoice, removing money from player.

**Requires:** `forcePay` permission

**Parameters:**
- `invoiceId` (string) - Invoice ID

**Behavior:**
- If player is online: Removes money from player's bank
- If player is offline: Only marks as paid
- If sender has identifier: Adds money to sender's account

---

#### `nboks:server:admin:refund`
Refunds a paid invoice.

**Requires:** `refund` permission

**Parameters:**
- `invoiceId` (string) - Invoice ID

**Requirements:**
- Invoice must be paid
- Receiver must be online
- Sender must have sufficient funds (if online)

**Result:** Sets `isPaid = 0` and `isRefunded = 1`

---

#### `nboks:server:admin:deleteInvoice`
Permanently deletes an invoice.

**Requires:** `deleteInvoices` permission

**Parameters:**
- `invoiceId` (string) - Invoice ID

---

#### `nboks:server:admin:getAuditLogs`
Retrieves audit logs of admin actions.

**Requires:** `auditLogs` permission

**Usage:**
```lua
TriggerServerEvent("nboks:server:admin:getAuditLogs")
```

**Result:** Triggers `nboks:client:admin:setAuditLogs` with log data

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

#### `nboks:client:updateInvoicePaidStatus`
Updates the paid status of an invoice.

**Parameters:**
- `invoiceId` (string) - Invoice ID
- `isPaid` (boolean) - New paid status

---

#### `nboks:client:updateInvoiceRefundStatus`
Updates the refund status of an invoice.

**Parameters:**
- `invoiceId` (string) - Invoice ID
- `isRefunded` (boolean) - Refund status
- `isPaid` (boolean) - Paid status (optional)

---

#### `nboks:client:removeInvoice`
Removes an invoice from the UI (used when admin deletes).

**Parameters:**
- `invoiceId` (string) - Invoice ID to remove

---

#### `nboks:client:setAdmin`
Sets whether the player is an admin.

**Parameters:**
- `isAdmin` (boolean) - Admin status

---

#### `nboks:client:setAdminPermissions`
Sets the admin permissions for the player.

**Parameters:**
- `permissions` (table) - Admin permissions object or nil

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
    TriggerServerEvent("nboks:server:payInvoice", data.id)
    cb("ok")
end)
```

**Security Note:** Only the invoice ID should be sent to the server. The server fetches the actual amount and type from the database to prevent mod menu exploits.

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
    local metadata = nil

    if data.rawData then
        metadata = {}
        local count = 0

        for _, ticketData in pairs(data.rawData) do
            if ticketData.ticket then
                local title = ticketData.ticket.title or "Ukendt overtrædelse"
                local ticketAmount = 0

                -- Parse the cols JSON to get the price
                if ticketData.ticket.cols then
                    local cols = json.decode(ticketData.ticket.cols)
                    if cols and cols.price then
                        ticketAmount = cols.price
                    end
                end

                table.insert(metadata, {
                    title = title,
                    amount = ticketAmount
                })
                count = count + 1
            end
        end

        if count == 1 then
            description = "Bøde for 1 lovovertrædelse"
        elseif count > 1 then
            description = "Bøde for " .. count .. " lovovertrædelser"
        end
    end

    exports['nboks']:sendInvoice(identifier, sender, nil, invoiceType, amount, description, metadata)
end)
```

This integration automatically creates a breakdown table showing each law violation and its cost.

---

### Custom Job Integration

#### Mechanic Shop Invoice with Breakdown

```lua
-- Mechanic shop invoice example with itemized breakdown
RegisterCommand("mechanicbill", function(source, args)
    local targetId = tonumber(args[1])

    if not targetId then
        TriggerClientEvent('chat:addMessage', source, {
            args = {"System", "Usage: /mechanicbill [playerid]"}
        })
        return
    end

    local targetIdentifier = getPlayerIdentifier(targetId)

    if targetIdentifier then
        -- Example: Itemized breakdown of mechanic work
        local metadata = {
            { title = "Motor reparation", amount = 2500 },
            { title = "Dæk skift (4 stk)", amount = 1200 },
            { title = "Bremseklodser", amount = 800 },
            { title = "Olieskift", amount = 500 },
            { title = "Arbejdsløn", amount = 1000 }
        }

        local totalAmount = 0
        for _, item in ipairs(metadata) do
            totalAmount = totalAmount + item.amount
        end

        exports['nboks']:sendInvoice(
            targetIdentifier,
            "Mechanic Shop",
            nil,
            "invoice",
            totalAmount,
            "Reparation af køretøj",
            metadata  -- This creates a detailed breakdown table in the invoice
        )

        TriggerClientEvent('chat:addMessage', source, {
            args = {"System", "Invoice sent with detailed breakdown!"}
        })
    end
end, false)
```

**Result:** The player will see a complete breakdown table showing:
- Motor reparation: 2.500 kr
- Dæk skift (4 stk): 1.200 kr
- Bremseklodser: 800 kr
- Olieskift: 500 kr
- Arbejdsløn: 1.000 kr
- **Total: 6.000 kr**

---

#### Hospital Invoice with Treatment Breakdown

```lua
-- Hospital invoice with medical treatment breakdown
RegisterServerEvent("hospital:sendBill")
AddEventHandler("hospital:sendBill", function(patientId, treatments)
    local identifier = getPlayerIdentifier(patientId)

    if identifier then
        local metadata = {
            { title = "Akut behandling", amount = 1500 },
            { title = "Medicin", amount = 500 },
            { title = "Røntgen scanning", amount = 800 },
            { title = "Ambulance transport", amount = 700 }
        }

        local total = 0
        for _, treatment in ipairs(metadata) do
            total = total + treatment.amount
        end

        exports['nboks']:sendInvoice(
            identifier,
            "Hospital",
            nil,
            "invoice",
            total,
            "Medicinsk behandling og transport",
            metadata
        )
    end
end)
```

---

#### Police Fine with Law Violations Breakdown

```lua
-- Police fine with specific law violations
RegisterCommand("policefine", function(source, args)
    local targetId = tonumber(args[1])

    if not targetId then return end

    local identifier = getPlayerIdentifier(targetId)
    local officerIdentifier = getPlayerIdentifier(source)

    if identifier then
        -- Example: Multiple law violations with individual fines
        local violations = {
            { title = "Straffelovens § 119 - Vold mod tjenestemand", amount = 5000 },
            { title = "Færdselslovens § 3 - Hensynsløs kørsel", amount = 2500 },
            { title = "Færdselslovens § 54 - Spirituskørsel", amount = 10000 },
            { title = "Våbenlovens § 1 - Ulovlig våbenbesiddelse", amount = 7500 }
        }

        local totalFine = 0
        for _, violation in ipairs(violations) do
            totalFine = totalFine + violation.amount
        end

        exports['nboks']:sendInvoice(
            identifier,
            "Politiet",
            officerIdentifier,
            "fine",
            totalFine,
            "Bøde for " .. #violations .. " lovovertrædelser",
            violations
        )
    end
end, false)
```

**Result:** The player will see each law violation with its specific fine amount, making it clear what they're being charged for.

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

## Security Features

### Audit Logging

**Complete activity tracking:**
- All admin actions logged to database
- Includes admin identifier, action, target, and timestamp
- Additional details (amount, type, previous values)
- Cannot be deleted by admins (only viewable)

### Security Logging

**Console warnings for suspicious activity:**
- Invalid payment attempts
- Permission bypass attempts
- Invalid input data
- Unauthorized admin actions

**Example log:**
```
^1[N-Boks SECURITY]^7 Player 5 (PlayerName) attempted to mark invoice as paid without permission
^1[N-Boks SECURITY]^7 Player 12 (PlayerName) attempted to save invalid color for primaryColor: javascript:alert(1)
```

---

## Discord Webhooks

Configure separate webhooks for different log types in `sv_config.lua`:

```lua
nboks.sv.discordwebhook = {
    adminLogs = "https://discord.com/api/webhooks/...",   -- Admin actions
    paymentLogs = "https://discord.com/api/webhooks/..."  -- Payments and invoices
}
```

### Payment Logs Webhook

**Logged Events:**

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

### Admin Logs Webhook

**Logged Events:**

1. **Invoice Marked as Paid**
   - Admin information
   - Receiver information
   - Amount and type
   - Invoice ID

2. **Invoice Marked as Unpaid**
   - Admin information
   - Receiver information
   - Amount and type
   - Invoice ID

3. **Invoice Force Paid**
   - Admin information
   - Receiver information
   - Amount and type
   - Status (online/offline)
   - Invoice ID

4. **Invoice Refunded**
   - Admin information
   - Receiver information
   - Sender information
   - Amount and type
   - Invoice ID

5. **Invoice Deleted**
   - Admin information
   - Receiver information
   - Amount and type
   - Was paid status
   - Invoice ID

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

---

## User Settings

Players can customize the UI colors through the settings dialog.

### Available Settings

- Primary Color
- Secondary Color
- Accent Color
- Sidebar Color
- Header Gradient Start
- Header Gradient End
- Content Background Color
- List Item Background Color

Settings are saved per player in the `nboks_user_settings` table and persist across sessions.

---

## License System

N-Boks uses a license validation system to verify your purchase.

### Configuration

Set your license key in `sv_config.lua`:

```lua
nboks.sv.licensekey = "INSERT-LICENSE"
```

### How It Works

1. **Initial Validation:** On resource start, the license is validated against our API
2. **HWID Generation:** A unique Hardware ID is generated and stored in `hwid.dat`
3. **Heartbeat System:** Continuous validation ensures the resource runs only with a valid license
4. **Protected Exports:** All exports require a valid license to function

### Console Output

**Successful validation:**
```
[N-Boks] Validating license...
[N-Boks] License validated successfully
```

**Failed validation:**
```
[N-Boks] Validating license...
[N-Boks] LICENSE VALIDATION FAILED
[N-Boks] Reason: Invalid license key
```

### HWID Reset

If you need to reset your HWID (e.g., server migration):

1. Delete the `hwid.dat` file in the resource folder
2. Restart the resource
3. A new HWID will be generated automatically

**Note:** You may need to contact support to reset the HWID on the license server.

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "No license key configured" | Set your license key in `sv_config.lua` |
| "Connection failed" | Check your server's internet connection |
| "Invalid license" | Verify your license key is correct |
| "HWID mismatch" | Reset HWID or contact support |

---

### Exports

| Export | Description |
|--------|-------------|------------------|
| `sendInvoice` | Send invoice/fine to player |
| `forcePayInvoice` | Force pay invoice |
| `chargePlayer` | Charge player immediately |
| `getInvoices` | Retrieve invoices with filters |
| `checkReminders` | Trigger reminder check |

### Server Events

| Event | Description | Permission Required |
|-------|-------------|---------------------|
| `nboks:server:getInvoices` | Get player's invoices | None |
| `nboks:server:payInvoice` | Pay an invoice | None |
| `nboks:server:checkAdmin` | Check admin status | None |
| `nboks:server:admin:getAllInvoices` | Get all invoices | viewInvoices |
| `nboks:server:admin:markAsPaid` | Mark as paid | markAsPaid |
| `nboks:server:admin:markAsUnpaid` | Mark as unpaid | markAsUnpaid |
| `nboks:server:admin:forcePay` | Force pay invoice | forcePay |
| `nboks:server:admin:refund` | Refund invoice | refund |
| `nboks:server:admin:deleteInvoice` | Delete invoice | deleteInvoices |
| `nboks:server:admin:getAuditLogs` | Get audit logs | auditLogs |

---

## Support

For support, bug reports, or feature requests, please contact the developer or visit the support channels provided with your purchase.
