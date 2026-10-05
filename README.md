# n8n Daily Top 2 Calendar Digest

An n8n workflow that runs every morning at 6 AM, reads today's Google Calendar events, uses an LLM to pick the two most important ones, and emails a short HTML briefing with their times and why they matter. On days with no events, no email is sent.

## How it works

Schedule Trigger → Google Calendar (today's events) → Edit Fields (keep title, description, time, attendees) → Aggregate (combine into one list) → Basic LLM Chain (Groq) → Gmail

The workflow is deliberately linear rather than agent-based. An earlier version gave an AI agent calendar and email tools, but it looped, sent duplicate emails, and hit Groq's free-tier rate limit (~6,900 tokens per run). The linear version makes exactly one LLM call per run (~400 tokens).

## How the top 2 are chosen

The model ranks events using rules defined in the Basic LLM Chain prompt, in this order:

1. Keywords like interview, deadline, exam, client, demo, review or presentation
2. More attendees
3. Has a description (suggests preparation is needed)
4. Longer duration

Routine events (lunch, gym, breaks, focus time) are ignored unless nothing else exists. Each pick cites the rule that made it important, and the remaining events are listed under "Also today". Edit the prompt to change the rules or keywords.

## Setup

1. In n8n, go to **Workflows → Import from File** and select `workflow.json`.
2. Create and connect these credentials:
   - **Google Calendar OAuth2**
   - **Gmail OAuth2**
   - **Groq API key** (free at console.groq.com)
3. In the **Get many events** node, select your calendar.
4. In the **Send a message** node, replace `your-email@example.com` with your email address.
5. In **Workflow Settings**, set the timezone to yours (the default is `Asia/Kolkata`).
6. Run it once with **Execute workflow** to test, then **Publish**.

## Tips

- Events with descriptions and guests get better picks, because the model has more to go on.
- If you hit Groq rate limits, pick a model with a higher output-token limit in the Groq Chat Model node.
