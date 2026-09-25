# Rules for Claude — Starter Set for Fred

From Matt (KO6NOI), 2026-09-24. These come from months of using Claude on radio and shack
projects. The short version: Claude is a great tool, but it will sometimes sound sure when it's
guessing. These rules make it tell you when it's guessing.

## How to set it up

1. In the Claude app, open **Settings → General** and find the instructions / personal
   preferences box. (Per Anthropic's help center: account-wide instructions that apply to
   every conversation.)
2. Paste the rules below into that box and save.
3. Optional: for a big project (like the Anytone/DMR setup), make a **Project** and put
   project facts in its instructions: radio model, repeaters, your DMR ID.

## The rules (copy everything in the box)

```
1. If you don't know, say "I don't know" in your first sentence. Don't guess and don't
   give me an answer just to make me feel better.
2. Before any setting, frequency, wiring, part, or step-by-step procedure, tell me the
   source: the radio manual, the manufacturer, the official docs. If it's from a forum or
   general experience, say so. If you have no source, say so and stop.
3. Mark anything you're inferring: "This is my guess, not confirmed."
4. When something stops working, go in this order: what changed since it last worked,
   then power and cables, then the radio's own settings, then the software settings.
   Suggest buying parts or reflashing only after those are checked.
5. Before a long procedure, tell me if there's a simpler way to get the same result.
6. One step at a time for anything that touches the radio. Wait for me to report back.
7. Keep answers short and plain unless I ask for detail.
8. This is a hobby. There's no rush. Getting it right beats getting it fast.
```

## Why each rule is there

- **1-3:** Claude can sound equally confident whether it read the manual or not. These make
  it say which.
- **4:** Most "broken" gear turns out to be a setting or a cable. Matt bought parts that
  weren't defective before learning this.
- **5:** A long procedure is sometimes unnecessary. A $35 part once replaced a whole
  firmware flash.
- **6:** Several changes at once make it impossible to tell which one broke something.

## Good habits when asking

- Say what changed: "It worked until I ___."
- Paste exact error messages or readings. Don't paraphrase them.
- Ask "What's your source?" when an answer sounds too smooth.
- Ask "Is there another way?" before a big job.
