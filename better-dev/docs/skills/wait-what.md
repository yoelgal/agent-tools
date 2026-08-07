# /wait-what

## What it does

Re-pitches the message that just lost you: one line of context for where the agent is and why it
matters, then the point again in short plain sentences using your project's own vocabulary instead
of better-dev's. Its defining constraint is what it refuses to be: a concision setting. Naming the
output ("be brief", "tldr") gets a clipped message that is shorter and no clearer; this skill names
the listener's failed comprehension, so the missing context arrives together with the fewer words.

## When to reach for it

Say "wait", "what?", "you lost me", or "what does that mean?" the moment a reply does not land -
the skill fires on the signal itself, and `/wait-what` by name works too. For shaping how every
message reads before anything fails to land, that is the comms style block `/onboard` writes, not
this skill; and an artifact whose owning skill requires it rendered in full (a contract at its
gate, a review verdict) still renders in full - the re-pitch wraps it, never replaces it.

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
