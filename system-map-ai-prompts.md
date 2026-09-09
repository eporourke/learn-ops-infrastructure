# System Map AI Prompts

## 1. Describe the System

I'd like an ASCII diagram that tells me about the system in the lms folder. I need a rundown of all the services, ports, and frameworks and how they each connect. Can you please output it here to the terminal?

❯ does the diagram represent how the systems communicate re: the arrows? in terms of whether they both send each other things, so both directions, or one just sending to another, so one directional arrow

Good catch — it's inconsistent. Some arrows show "who initiates the connection," others show "which way the data conceptually flows," and I mixed the two conventions without being clear about it. Here's the honest breakdown per connection:

Truly bidirectional (request → response, shown correctly as two-way):
- Client ↔ API — client sends the HTTP request, API sends back JSON. This one I drew accurately with two separate arrows.

One I drew as one-directional but is actually request/response (should be two-way):
- API → database — I only drew the query going down. Postgres also returns rows/results back up to the API. Same for Monarch → GitHub — I drew a single downward arrow, but GitHub returns issue data/confirmations back to Monarch.

One I got backwards entirely:
- database → postgres_exporter — I drew this pointing from the database down to the exporter. That's wrong. postgres_exporter is the one that opens a connection to Postgres and queries its stats views. The arrow should point postgres_exporter → database.

Pull/scrape connections (connection is initiated by the receiver, but the arrow shows the metrics payload, which is the opposite direction):
- Prometheus → API, Prometheus → postgres_exporter, Prometheus → Monarch — in reality Prometheus opens the connection and does the GET, so the initiation arrow points Prometheus → target, but the thing you probably care about (metrics data) flows target → Prometheus. I drew these as if data flows toward Prometheus, which is defensible but I didn't label which direction was "connection" vs "payload."
- Grafana ← Prometheus — same issue: Grafana initiates the query, Prometheus's response is the data. I drew it as Prometheus → Grafana (payload direction), not showing that Grafana is the one opening the connection.

Pub/sub (same ambiguity as scraping, just inverted):
- Valkey → Monarch labeled "SUBSCRIBE" and Valkey → valkey-monitor labeled "MONITOR" — these arrows show message push direction correctly (Valkey does push data to subscribers), but the label is misleading: it's actually Monarch/valkey-monitor that initiates the connection and issues the SUBSCRIBE/MONITOR command to Valkey. So connection-initiation goes the opposite way from what the arrow shows.
- API → Valkey PUBLISH — this one's clean, genuinely one-directional, fire-and-forget, no response needed.

Genuinely one-directional, no ambiguity:
- Monarch → Slack (webhook/bot notification) — Monarch sends, Slack doesn't push anything back to Monarch on its own initiative.

So short answer: the diagram conflates "who opens the connection" with "which way data flows," and one arrow (database↔postgres_exporter) is flipped outright. Want me to redraw it with a consistent convention — e.g., solid arrow = who initiates/connects, and a small ⇢ or annotation for response/payload direction?

## 2. Convert to a Mermaid Diagram