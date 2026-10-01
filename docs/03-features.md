[Docs](./README.md) › **Features**

# Features

What each page does, in plain words. Every list page shares the same toolkit: **KPI cards**, **status tabs**, **search**, **filters**, **sortable columns** and **pagination**.

## Pages

### 🔐 Sign in and accounts

- Create an account or sign in with email and password.
- The login is remembered with a secure session cookie, so a page refresh keeps you signed in.
- Signing out clears all data from the screen, so the next person starts fresh.

### 📊 Dashboard

- Four KPI cards: properties, units, monthly revenue and outstanding payments, each with a small trend line.
- Revenue chart with a *Monthly / Quarterly / Yearly* range.
- Occupancy donut chart, a property performance table, and a revenue breakdown by property.
- **Action required:** overdue payments, leases expiring within 30 days, and open repairs.
- Recent payments, recent repairs and an activity timeline.

### 🏢 Properties and units

- Property cards with photo, address, occupancy bar and monthly revenue.
- Add or edit a property, including a photo upload.
- Open a property to manage its **units** (name, floor, type, rent, status).
- Unit names must be unique inside a property, and the unit type sets the bedroom count (a "2 BR" always has 2 bedrooms).

### 👥 Tenants

- One table with each tenant's property, unit, rent, lease status and payment status.
- Phone numbers have a country picker and format as you type.
- From a tenant, jump straight to **their** leases or payments. The link is a normal URL, so it can be shared.

### 📄 Leases

- Track active, expiring and expired leases with term, rent and deposit.
- Picking a tenant fills in their property and rent for you.

### 💳 Payments

- See collected, pending and overdue rent, plus the collection rate.
- Record a payment or edit an old one. New payments can't be dated in the past.
- Filter by property and payment method (bank, card or cash).

### 🔧 Maintenance

- A request can target the **whole property**, **specific units** or **specific tenants**.
- Set priority, category, estimated cost and an assigned staff member.
- Move a request through its lifecycle: *Open → In Progress → Paused → Completed* (or *Scheduled*). Every change is saved in a history trail.
- Manage the maintenance staff list. Only active staff can be assigned.

### 🗂️ Documents

- Upload PDF, Word, Excel or text files (up to 10 MB) and link them to a property, unit, tenant or lease.
- Five filters, plus a *grouped by property* view.
- Track expiry dates, edit details, move a file to another property, or archive it.

### 🔔 Notifications and inbox

- A notification bell for new tenants, new repair requests and overdue payments. Mark one or all as read.
- A messages panel for tenant conversations. *This is the one feature that uses local demo data instead of the API.*

## What happens automatically

These run without anyone clicking a button:

| Automation | What it does |
| --- | --- |
| **Monthly rent** | Every day at 00:05 UTC (and when the server starts), the API creates the month's rent for each active lease. Next month's rent appears 3 days before the month starts. |
| **Overdue flag** | Rent still *Pending* the day after it's due becomes *Overdue*. Due Oct 1 → overdue on Oct 2. |
| **No double billing** | A unique index allows only one automatic bill per lease per month, even if two runs overlap. A manual payment for that month also counts. |
| **Notifications** | Adding a tenant, creating a repair request or a payment going overdue creates a notification. |
| **Live occupancy** | A unit with a current tenant or lease shows as *Occupied*, worked out from live data rather than typed in. |
| **Safe deletes** | You can't delete a unit or staff member that is still in use. Propora lists what's blocking the delete instead. |

## Built for everyone

- **Responsive:** works on phones, with a mobile menu and card-style rows on small screens.
- **Accessible:** visible keyboard focus, status shown with a text label (never colour alone), and reduced-motion support.
- **Friendly feedback:** short toasts confirm every action, and errors show the server's own message.

---

<div align="center">

[← Screenshots](./02-screenshots.md) · [Docs home](./README.md) · [System design →](./04-system-design.md)

</div>
