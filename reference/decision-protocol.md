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
