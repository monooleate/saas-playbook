# Contributing

This playbook is opinionated. Contributions that match the opinion will be
merged quickly; contributions that fight it will be discussed.

## The opinion

1. **Solo founder perspective.** Every recommendation must be doable by one
   person on a Sunday afternoon. If a pattern requires a Platform Engineering
   team, mark it explicitly and offer a "small-team alternative".
2. **Stack-agnostic core, three concrete examples.** Patterns live in
   `README.md` / `checklist.md` / `best-practices.md` / `gotchas.md`. Code lives
   in `examples/sveltekit-supabase.md`, `examples/nextjs-prisma.md`,
   `examples/remix-planetscale.md`. New stacks need a strong justification and
   maintainer commitment.
3. **Real production only.** Every gotcha must have happened to someone, with
   enough specifics that it sounds real (error message, what broke, how it was
   detected). Theoretical risks belong in `best-practices.md`.
4. **Every claim is sourced.** "Stripe does X" must link to Stripe docs.
   "We learned Y from incident Z" must give enough context to be plausible.
   Vague pattern X "is a known anti-pattern" without a source will be rejected.
5. **No marketing-speak.** No "delightful," no "best-in-class," no "enterprise-
   grade." Plain English describing what the code or the human does.

## What gets merged

- New gotchas you've actually hit, written as: situation → symptom → root cause
  → fix.
- Best-practice references with a working URL and a one-paragraph summary of
  what the source says.
- A new example for an existing stack, replacing or extending an existing one.
- A new topic, only if (a) it doesn't fit any existing topic, (b) it's
  genuinely needed by a solo founder, and (c) you commit to filling at least
  the README, checklist, and one example.

## What gets pushed back

- Adding a fourth example stack without removing one. Three is the cap;
  fragmentation kills usefulness.
- "AI-generated content from a single prompt." This is fine as a draft, but
  every paragraph in the merged version must be reviewed and corroborated.
- Recommending a paid service without naming the free alternative.
- Recommending the latest hot framework. The playbook tracks production-proven,
  not new-and-shiny. New stack? Wait six months, then propose.

## Process

1. Open an issue describing the change. Quick alignment check.
2. PR with the changes. Include:
   - Updated `README.md` / `CHANGELOG.md` if scope shifts.
   - Working external links (run a link checker locally).
   - Code snippets that compile (tests in the example repo where possible).
3. One reviewer, one round of feedback, then merge. No bikeshedding.

## Style

- Markdown, not MDX. Plain `.md` files render in every tool.
- One concept per heading. If a section is over 200 lines, split it.
- File paths in fenced code blocks. Inline code for shell commands and
  identifiers (`PUBLIC_APP_URL`, `users.id`).
- Date format: `YYYY-MM-DD`. Always absolute.
- No emoji decoration in headings. Emoji in callouts is fine if it carries
  information (⚠️ warning, 📌 update marker).
- Hungarian or other non-English notes from source projects are translated to
  English in the playbook. Source comments may stay in original language only
  inside `examples/` quoted blocks where authenticity matters.

## License acknowledgement

Contributions are MIT-licensed. By submitting a PR you confirm you have the
right to release the content under MIT.
