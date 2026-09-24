# Decision protocol

Whenever a `/phaser` command needs the user to decide something, never ask in
bare prose and never bundle several decisions into one message:

1. **Describe it in text first** — what is undecided or in conflict, why it
   matters, and what each candidate buys and costs. Be honest about the cons
   of the option you prefer.
2. **Then present it with the AskUserQuestion tool**, one question per
   decision: one option per candidate (usually 2–3), your recommendation
   first and labeled "(Recommended)", with the key pro AND con packed into
   each option's description so the trade-off is visible in the picker
   itself. Use `multiSelect: true` when the candidates are not mutually
   exclusive, such as a list of requirements; then list the recommended
   candidates first, each labeled "(Recommended)".
3. The tool's built-in "Other" covers free-form answers — don't add your own
   catch-all option. Record the resolution before moving to the next decision.
   Fall back to asking in text only if the tool is unavailable.
4. **Attribute the outcome to the user.** In hand-offs, summaries, plan-file
   notes and OpenSpec artifacts, say what the user chose — never "I resolved"
   or "I decided" for a choice that went through this protocol. First person
   is for choices you made without asking.
5. **Do not ask about obvious choices.** When your confidence is high, the
   option you would recommend has no con — or its only con is extra work —
   no other option is equally good, and the action is reversible, act and
   report it (first person — it is your call, not the user's). Bringing
   OpenSpec artifacts or specs, docs, or CLAUDE.md in line with the actual code
   is always such a choice when it is the recommended option and the code,
   not the doc, is confidently correct. Only steps whose command text
   adopts this clause use it; today that is `/phaser:review` and
   `/phaser:scrutinize`, driven by their subagents' `[auto]` tag. An
   auto-applied review blocker, or an auto-applied fix for a `not-met`
   criterion, is reported as its own information point: what was chosen,
   why it was obvious, and the alternatives.
