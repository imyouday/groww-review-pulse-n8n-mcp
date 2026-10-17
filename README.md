# Groww Review Pulse: n8n + MCP with Approval Gate

An AI workflow that turns raw Groww Google Play reviews into (1) a weekly internal product pulse and (2) a customer-facing fee explainer. It logs both through MCP and sends the summary email, only after a human approves.

Built for NextLeap PM Fellowship, Milestone 5.

## What it does

```
Upload CSV → Clean & sample → Step 1: Insights (themes, quotes, fee issue)
→ Fetch 2 official pages → Step 2: Weekly Pulse (≤250 words)
→ Step 3: Fee Explainer (≤6 bullets, 2 sources, "Last checked")
→ Validate → APPROVAL GATE (Gmail "Send & Wait")
→ Approved? ─ yes → AI Agent + MCP Client → MCP Server → Google Doc append + Gmail email (sent)
            └ no  → stop, nothing is logged
```

## What fee issue was identified from reviews

**Brokerage per order, DP (depository) charges and statutory add-ons (GST etc.).**

Reviews from Oct 10–14, 2026 (184 analysed after cleaning) show small traders who feel per-order and DP charges are high or unclear, and some report unexpected deductions. The CSV has no "exit load" mentions, so that fee was not used.

Top themes found: High Fees & Charges · Customer Support & Withdrawals · Trading Execution, Performance & Bugs.

## Where MCP approval happens

1. **Approval gate:** the `APPROVAL GATE (Gmail Send & Wait)` node emails the pulse, the explainer and the validation checks. The workflow pauses until you click **Approve & Log** or **Reject**.
2. **Approved?** (IF node) sends only approved runs to the MCP Executor Agent. Rejected runs stop at `Rejected: No Action`.
3. **MCP calls** (`Own MCP Client` → `MCP Server Trigger`) expose two tools:
   - `append_to_doc`: appends the structured entry to a Google Doc
   - `send_email`: sends the weekly pulse + fee explainer email, only after approval

Logged entry shape: `{date, top_themes, weekly_pulse, identified_fee_issue, explanation_bullets, source_links}`

## How to run

1. Import `workflows/MCP-Server.json` into n8n. Set Google Docs + Gmail credentials and your Doc URL, activate it, and copy the MCP Server Trigger **Production URL**.
2. Import `workflows/Review-Pulse-Pipeline.json`. Paste that URL into `Own MCP Client`. Add your OpenAI credential on the model nodes and your email on the approval node.
3. In `Config: Official Sources`, set the 2 official source URLs for the fee.
4. Open the form (`Upload Reviews CSV`), upload a Groww reviews CSV (a sample is in `data/reviews_sample.csv`), and run.
5. Check your inbox, click **Approve & Log**, then check the Google Doc and your inbox for the summary email.

## Repo contents

| Path | What |
|---|---|
| `workflows/` | n8n workflow JSONs (credentials removed) |
| `data/reviews_sample.csv` | 50-row sample of the reviews CSV |
| `outputs/weekly_pulse.md` | Generated weekly pulse |
| `outputs/fee_explainer.md` | Generated fee explainer |
| `outputs/doc_entry.txt` | Entry appended to the Google Doc |
| `outputs/email.txt` | Email text |
| `screenshots/` | Workflow, approval email, Doc entry, email |
| `SOURCES.md` | Source list (4–6 URLs) |

## Limitations

- The original brief asks for an email *draft* with no auto-send. This version sends the email, but only after the human approval step. To get draft-only behaviour, set the Gmail tool in the MCP Server to Resource: Draft, Operation: Create.
- The CSV covers only Oct 10–14, 2026 (5 days), not 8–12 weeks.
- Themes come from an LLM over a sample of up to 160 reviews, so counts are approximate.
- The fee explainer uses only text fetched from the official pages. Recheck the links when the "Last checked" date gets old.
- Reviews are public Google Play reviews.
