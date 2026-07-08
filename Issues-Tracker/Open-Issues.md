# 🔴 Open Issues — All Clients

> This file is a manual tracker for cross-client issues. Use the Dataview query below for a live view pulled directly from each client's `Open Issues` section.

---

```dataview
TASK
FROM "Clients"
WHERE !completed
GROUP BY file.link
```

---

## Manually Logged Issues (Fallback)

| Client | Issue | Raised | Owner |
|---|---|---|---|
| upGrad-UGSOT | System prompt hallucination on fee waiver queries | 2026-06-05 | Engineering |
| SMFG India Credit | VAPT remediation doc pending legal review | 2026-06-01 | Legal |
| Muthoot Finance | API docs for core banking integration pending | 2026-07-01 | Intern |
| Aspire Finance | UAT sign-off pending from ops team | 2026-06-20 | Client |
| Escorts Kubota | UAT sign-off overdue — client not responding | 2026-06-15 | Vaibhav |

---

*Last updated: 2026-07-08 | Update manually when Dataview is not available*
