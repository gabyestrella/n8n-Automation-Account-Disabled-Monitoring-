# Meta Ads – Account Disabled Monitoring

<p align="center">
  <img src="assets/workflow.png" width="800" alt="n8n Workflow — Meta Ads Account Disabled Monitoring" />
</p>

<p align="center">
  <strong>Daily automated sweep across the Meta portfolio → instant Slack alert when an account goes down</strong><br/>
  Every morning at 9:00 AM. Same-day visibility. Zero manual checking.
</p>

---

## About This Repository

This is a portfolio showcase of a production automation I designed, built, and actively maintain at Beyond Marketing. The full workflow runs in a private n8n cloud instance.

This repository documents the architecture decisions and business logic behind the system.

---

## My Role — Designer, Architect & Builder

**Gaby Estrella**  
[LinkedIn](https://www.linkedin.com/in/gaby-estrella-/) · [GitHub](https://github.com/gabyestrella) · [gabyestrella.dev](https://gabyestrella.dev)

This automation came from a real operational problem I lived through as a marketing strategist — and solved by combining that experience with AI engineering.

In healthcare advertising, Meta's ad policies are strict. Accounts get disabled for billing issues, policy flags, or compliance reviews — sometimes with no warning. Before this system existed, strategists would discover a downed account days after it happened, sometimes two or three days later. By then, campaigns had been silent, leads had been lost, and clients had no idea why.

I designed this workflow because I knew exactly what the cost of that delay was — not in abstract terms, but from direct experience managing healthcare clients on Meta. The value wasn't just in building the automation. It was in knowing what to automate, why it mattered, and how to make it reliable enough that the team would actually trust it.

### What I built

**Daily portfolio sweep.**
The workflow calls Meta's Graph API every morning to pull the full list of ad accounts connected to Beyond Marketing's token. It doesn't rely on manual reporting or dashboard checks — it goes straight to the source.

**Two-stage filter logic.**
I designed a two-step evaluation: first, exclude accounts marked `[INACTIVE]` in their name — those are accounts the team has intentionally paused, so a non-active status on them is expected and not worth alerting. The workflow only proceeds with accounts that are supposed to be running. For those, it checks whether the `account_status` is anything other than `1` (active). If something is down that shouldn't be — the team knows that morning.

**Immediate Slack alert with context.**
When an account is flagged, the workflow posts directly to `#marketing` with the account name, ID, and status code so the strategist has enough context to act immediately — no digging through Ads Manager to figure out which account or what happened.

**Stateless by design.**
Every run is a fresh pull. No stored state, no memory between executions. Clean, reliable, easy to maintain.

---

## The Impact

Before this system, a downed account could go undetected for two to three days — sometimes longer. In healthcare advertising, where campaigns are running for wound care, TMS, Spravato, and Medicare services, that gap means missed leads and clients who have no idea why their pipeline stopped.

- **Same-morning detection.** The team now knows about account issues within hours of them happening, not days.
- **Immediate action.** Strategists open Slack, see the alert, and go fix it — no investigation required, no manual account-by-account checking.
- **Team trust.** The workflow has become a reliable daily signal. The team doesn't think about account health anymore because they know this system is watching it.

That shift — from reactive discovery to proactive awareness — is what this automation was built for.

---

## Architecture

### Flow

```
Schedule Trigger (daily 9:00 AM)
        │
        ▼
HTTP Request → GET /me/adaccounts (Meta Graph API v24.0)
        │
        ▼
Split Out → one item per ad account
        │
        ▼
IF – Is [INACTIVE]? → name contains "[INACTIVE]"
        │
        ├── true → No Operation (intentionally paused — skip)
        │
        └── false → IF – Account Disabled? → account_status != 1
                          │
                          └── true → Slack – Account Disabled Alert → #marketing
```

### Logic explained

Accounts tagged `[INACTIVE]` in their name are ones the team has intentionally paused. A non-active status on those is expected — so the workflow skips them entirely.

The accounts that matter are the ones **without** that tag — the ones that are supposed to be running. If any of those come back with a status other than `1` (active), that's an unplanned outage, and the alert fires.

### Node breakdown

| Node | What it does |
|---|---|
| Schedule Trigger | Fires daily at 9:00 AM server time |
| HTTP Request | Calls `GET /me/adaccounts` with fields: `id, name, account_id, account_status`, limit 200 |
| Split Out | Explodes the accounts array so each account is evaluated individually |
| IF – Is [INACTIVE]? | If name contains `[INACTIVE]` → skip. If not → proceed to status check |
| No Operation | Terminal sink for intentionally inactive accounts |
| IF – Account Disabled? | Numeric check: `account_status != 1` (1 = active in Meta's schema) |
| Slack – Account Disabled Alert | Posts to `#marketing` with account name, ID, and status code |

### Tech stack

| Layer | Technology |
|---|---|
| Automation | n8n |
| Ad Data | Meta Ads Graph API v24.0 |
| Alert Delivery | Slack API (OAuth2) |

---

## Configuration

- **Inactive marker:** accounts whose name contains `[INACTIVE]` are intentionally paused and skipped
- **Monitored accounts:** everything without that tag — accounts expected to be running
- **Active status code:** `1` (anything else on a monitored account triggers an alert)
- **Schedule:** daily, 09:00 AM server time
- **Slack channel:** `#marketing`

---

## Marking an Account as Intentionally Inactive

If an account is being paused on purpose and shouldn't trigger alerts:

1. Rename the account in Meta Business Suite to include `[INACTIVE]` in the name
2. The next morning's run will skip it automatically

To resume monitoring, remove `[INACTIVE]` from the name.

---

<p align="center">
  A campaign going dark for three days in healthcare isn't just a performance issue — it's a patient pipeline going silent.<br/>
  This system exists so that never happens on our watch.<br/><br/>
  <strong>Same-day awareness. Every day.</strong>
</p>
