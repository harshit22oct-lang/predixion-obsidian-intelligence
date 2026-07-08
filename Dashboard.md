# 🧠 Predixion AI — Client Intelligence Hub

> [!abstract] 📌 System Overview
> **Founder:** Vaibhav Goyal &nbsp;|&nbsp; **System:** Obsidian Intelligence v1.0
> **Auto-synced:** 🎙️ Fireflies &nbsp;·&nbsp; 💼 HubSpot &nbsp;·&nbsp; 📧 Outlook
>
> 🤖 This dashboard refreshes every morning at **7:00 AM** via live API sync.

---

## 🟢 Active Clients

> [!success]- Currently engaged accounts — live roster
> ```dataview
> TABLE 
>   last_contact AS "Last Contact",
>   status AS "Status",
>   products AS "Products",
>   contact_name AS "Contact"
> FROM "Clients"
> WHERE status = "Active"
> SORT last_contact ASC
> ```

---

## 🔴 Dormancy Alert — Action Required (7+ days)

> [!danger]- ⚠️ No contact in over 7 days — immediate follow-up needed
> ```dataview
> TABLE 
>   last_contact AS "Last Contact",
>   contact_name AS "Contact",
>   contact_email AS "Email"
> FROM "Clients"
> WHERE date(today) - date(last_contact) > dur(7 days)
> SORT last_contact ASC
> ```

---

## 📋 Open Issues & Action Items

> [!warning]- Outstanding tasks across every client file
> ```dataview
> TASK
> FROM "Clients"
> WHERE !completed
> GROUP BY file.link
> ```

---

## 📊 All Clients Overview

> [!note]- Full roster, most recently contacted first
> ```dataview
> TABLE
>   status AS "Status",
>   products AS "Products",
>   last_contact AS "Last Contact",
>   contact_name AS "Contact"
> FROM "Clients"
> SORT last_contact DESC
> ```

---

> [!quote] 
> Predixion AI Intelligence System · Built for Vaibhav Goyal
> Powered by 🎙️ Fireflies · 💼 HubSpot · 📧 Microsoft Outlook