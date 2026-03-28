---
name: communicating-with-rid
description: How scheduled task output works. Use when deciding whether to message Rid or stay silent during a scheduled task.
---

# Communicating with Rid

Everything you output during a scheduled task gets sent to Rid as a Telegram message. There is no "thinking out loud" — if you write it, he sees it.

## 1. The `[silent]` Tag

If you decide not to message Rid — because the timing is wrong, you already spoke recently, or there's simply nothing worth saying — include `[silent]` anywhere in your output. The system will suppress the message.

```
[silent] He's out running. Nothing urgent. I'll catch him later.
```

Rid will not see this. You can still write internal reasoning after the tag — it gets logged but not delivered.

## 2. When to Use

- You checked both taskboards and there's nothing to surface
- You already spoke to Rid recently and another message would be redundant
- The timing is bad (he's sleeping, busy, or just stepped away)
- Your only output would be an explanation of why you're not messaging

## 3. When NOT to Use

- There's a task or follow-up that needs his attention — always deliver those
- He asked you a question and is waiting for a response
- Something urgent came in from email or elsewhere

## 4. What NOT to Do

- Don't output reasoning like "I'll skip this one" without the `[silent]` tag — it will be sent as a message
- Don't use `[silent]` to avoid difficult conversations — if something needs saying, say it
