---
name: voice-input
description: "Josh sometimes sends messages via iOS voice input from his iPad, which produces natural speech patterns (uhs, likes, pauses, \"comma\", \"period\")."
metadata: 
  node_type: memory
  type: user
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

Josh frequently uses **iOS voice dictation** from his iPad to talk to me, so his messages can contain spoken-language artifacts that don't carry semantic meaning:

- Filler words ("uh", "uhm", "like", "you know")
- Spoken punctuation ("comma", "period", "question mark")
- Restarts and self-corrections mid-sentence
- Unusual line breaks where pauses got transcribed
- Occasional homophone errors (e.g. "their" vs "there", "to" vs "too")

**How to apply:**
- Treat these as natural speech, not as confusion or instructions to literally include "uh" in code.
- Don't ask for clarification when the meaning is clear despite the disfluencies.
- Don't ever quote them back in commit messages, summaries, or written responses — strip them when restating.
- Spoken "comma" or "period" inside a sentence is voice dictation that didn't quite land; interpret it as punctuation, not as the literal word.
- Self-corrections override what came before (e.g. "let's do the green button, no the blue one" = blue).
