# /wait-what

## What it does

Re-pitches the message that just lost you: one line of context for where the agent is and why it
matters, then the point again in short plain sentences using your project's own vocabulary instead
of better-dev's. Its defining constraint is what it refuses to be: a concision setting. Aiming a
fix at the output ("be brief", "tldr") trims the reply while the gap that lost you stays open;
aiming it at the reader - you could not follow that - brings back both the point and the
background it needed, in less space.

## When to reach for it

| Situation | Route |
|---|---|
| A reply just lost you - "wait", "what?", "you lost me", "what does that mean?" | here; it fires on the signal itself, and `/wait-what` by name works too |
| Shaping how every message reads, before anything fails to land | the comms style block `/onboard` writes |
| An artifact its owning skill renders in full (a contract at its gate, a review verdict) | still rendered in full - the re-pitch wraps it, never replaces it |

## Where it fits

Off the chain entirely - it changes the next message, never the work. It pairs with the comms block
the way a correction pairs with a habit: the block is the standing preventive shape, this is the
named move for the moment the shape still failed.

## Common questions

**Why is the skill only a few lines long?** By design, and the design is recorded
(`docs/DECISIONS.md` D30): a corrective against volume that itself grows teaches the volume, not
the rule. The library's authoring standard names this failure class - a skill about a property of
its own text fails by growing - and this skill is its working example.

## It's working if

- The reply after a "wait, what?" opens with one line of where things stand, then the point in your
  project's words - shorter than the message it replaces and carrying context that message lacked.
- The vocabulary that lost you does not come back in the re-pitch; your repo's own terms do.
