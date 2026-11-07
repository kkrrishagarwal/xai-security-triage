# XAI Security Triage Room

Explainable, human-checked phishing triage in your team chat, built as a
single [Switch](https://github.com/sandbox-quantum/switch) template.

**Demo video:** _add your YouTube/Loom link here_

## The problem

AI-based phishing filters have two practical problems:

1. **Black-box scores.** They say "dangerous" without showing why, so an
   analyst can't verify or overturn the rating.
2. **False positives.** They act on that score directly, so a real
   vendor's invoice or a customer's email gets blocked.

This room fixes both: every rating comes with evidence, and no response
is proposed for approval without business context from a human.

## How it works

Two Claude Code agents and you, in one Slack channel.

```
 you / teammates ──► @whitebox-analyst ──(High only)──► @context-responder
                         │                                   │
                  rating + evidence             draft on hold + ONE question
                                                             │
                                          you answer ──► tightened draft ──► you approve
```

### Whitebox Analyst (`{team}-whitebox-analyst`, addressing: `anyone`)

Anyone in the channel can drop in an email, URL, QR text or code snippet.
The analyst replies in a fixed, short format:

```
🔍 RISK: High  (confidence: High)
WHY:
1. [A] Lookalike letter in sender domain: "U+0430"
2. [B] Hidden instruction to AI: "ignore your previous instructions"
3. [D] Urgency + credential ask: "suspended in 2 hours"
4. [E] Reply-To on unrelated webmail: "urgent.refunds@gmail.com"
NOT CHECKED: domain age, SPF/DKIM/DMARC (no headers)
```

- **Evidence for every finding**: each line carries a short quote or a
  Unicode code point. No quote, no finding.
- **Character-level inspection**: it writes the sample to a file in its
  own directory and inspects it with a local script, to catch lookalike
  letters, zero-width and bidirectional characters. It never opens,
  fetches or resolves a link.
- **Robust against attacks on the AI**: the sample is treated strictly as
  data. Embedded prompt injection is reported as a finding, not followed.
- **Neutral**: lookalike characters are described by code point, never by
  country or language.
- Six check families: (A) adversarial perturbations, (B) hidden prompt
  injection, (C) synthetic text (weak signal only), (D) social
  engineering, (E) infrastructure, (F) malicious code.
- Escalates to the responder **only** on High.

### Context Responder (`{team}-context-responder`, addressing: `owner-agents`)

Wakes only when the analyst hands it a High rating. Three phases, never
merged:

1. **Draft and one context question**: a block rule scoped to the exact
   indicators and a staff warning email, both on hold, plus one question
   to the owner, e.g. _"Is this domain a known vendor?"_
2. **Apply the context**: stands down ("false positive avoided") if the
   owner confirms it's legitimate, or tightens the draft if not.
3. **Approval**: only now asks the owner to approve, edit or reject.

It only drafts. It never runs a firewall, DNS or email command, and only
the owner's answers and approvals count.

## How the channel is set up

The template's `room:` block creates `{team}-xai-triage` on the messaging
app the deployer picks (Slack in our demo), with both agents and the
deployer (`{$creator}`) in it, then posts a kickoff that wakes both
agents. Addressing policies enforce the roles: teammates can talk to the
analyst, but only the owner and the analyst can wake the responder.

## How to run it

### Prerequisites

- A Switch server (Switch Cloud, or local: see
  [local development](https://github.com/sandbox-quantum/switch/blob/main/docs/old/LOCAL_DEVELOPMENT.md))
- [Switch Console](https://github.com/sandbox-quantum/switch/releases)
  connected to that server
- [Claude Code](https://claude.com/claude-code) installed and signed in on
  the machine the agents run on (Codex or OpenCode also work)
- A messaging app connected to the server. For Slack: in Console, Home →
  Messaging apps → Connect → Slack. Switch uses Socket Mode, so nothing
  needs to be publicly reachable.
- Your Slack account linked to your Switch user, so the agents recognise
  you as their owner

### Deploy

1. In Switch Console, open **Templates → Import a template**, paste
   [`xai-security-triage.template.yaml`](xai-security-triage.template.yaml)
   and save it.
2. Open it and click **Use template**.
3. Enter a **team** prefix (lowercase, e.g. `acme`), pick your messaging
   app, and click **Create all**.
4. A channel `acme-xai-triage` appears with both agents, who introduce
   themselves.

### Try it

In the channel:

```
@acme-whitebox-analyst analyse this email:
<paste demo-samples/1-phishing.txt>
```

Copy the sample from the file, not from a web page: it contains
invisible characters on purpose.

- The analyst rates it **High** with evidence and escalates.
- The responder posts a held draft and asks you one question. Answer in
  the thread, e.g. `@acme-context-responder No, not a vendor.`
- It tightens the draft and asks for approval:
  `@acme-context-responder approve`

Then send `demo-samples/2-legit-client.txt` the same way: it is rated
**Low** and the responder is not triggered.

## Files

| File | What it is |
|---|---|
| `xai-security-triage.template.yaml` | The Switch template: params, two agents, room, kickoff |
| `demo-samples/1-phishing.txt` | Phishing sample: lookalike letters (U+0430, U+0399), a zero-width space (U+200B), and a hidden prompt-injection comment |
| `demo-samples/2-legit-client.txt` | A legitimate client invoice, for the false-positive check |

## Built for

Flip the Switch Hackathon by WeMakeDevs — Track 1: Best Business Workflow.
