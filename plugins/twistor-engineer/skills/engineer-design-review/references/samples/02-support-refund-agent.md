# RFC: support agent that resolves refund and account tickets end to end

Status: RFC, asking for review before we start the build. Owner: Support Tooling.

## Problem

About 40% of support tickets are refund requests or account changes (email change, plan
downgrade, address fix). Each takes an agent 6 to 9 minutes, mostly copying data between the
helpdesk and the admin console. Median first response is 5 hours.

## Goals

- Resolve refund and account-change tickets without a human for the common cases.
- First response under 5 minutes for those tickets.
- Keep humans in the loop for anything the agent is unsure about.

## Non-goals

- Phone and chat support. Email only in v1.
- Fraud review. The existing fraud rules still run on every refund.

## Design

### Flow

1. A new ticket arrives in the helpdesk. A webhook enqueues it.
2. A worker builds the prompt: the ticket thread, the customer record and our refund policy doc.
3. The model plans and calls tools in a loop until it marks the ticket resolved or escalated.
4. The worker posts the model's reply on the ticket.

### Model

The current frontier model through our existing LLM gateway, temperature 0.

### Tools and access

To avoid building one tool per action, the agent gets a single generic tool,
`admin_api(method, path, body)`, which calls any endpoint of our internal admin API. It runs with
the existing `support-automation` service key, which has the `admin:*` scope because the old
macros used it. This covers refunds, account edits, plan changes and anything else support needs
later, with no new tool work.

The agent can also run `helpdesk.reply` and `helpdesk.set_status`.

### Escalation

The system prompt tells the agent to escalate to a human if it is unsure, if the refund is over
$500, or if the customer is angry. Escalation sets the ticket to "needs human" and adds a summary
note.

## Launch plan

1. Build the worker and tools (2 weeks).
2. Internal demo to the support leads using five tickets they pick.
3. If the demo goes well, turn it on for all new refund and account tickets.
4. Support leads review a sample of resolved tickets in the weekly ops meeting.

## Metrics

- Share of tickets resolved without a human.
- First response time.
- CSAT on agent-resolved tickets, from the existing survey.

## Cost

Estimated 12 model calls per ticket at current prices, within the support tooling budget.

## Risks

- The model could give a wrong answer. Mitigation: escalation rules in the system prompt, and
  weekly sampling by support leads.
- Customers may not like talking to a bot. Mitigation: the reply is signed as the support team.
