# Contributing

Thanks for wanting to make `/team` better.

- **Found a bug?** [Open an issue](https://github.com/bisque-cloud/bisque-team-skills/issues)
  with what you asked your agent, what it ran, and what happened. Include your
  agent (Claude Code, Gemini CLI, Codex) and your OS. Never paste a roster,
  an invite link, or an API key into an issue — describe the shape of the
  problem instead.
- **Found a security problem?** Email security@bisque.cloud rather than
  opening an issue. This skill changes who can read a company's presentations,
  so a report about it should not be public while it is unfixed.
- **Have a fix or improvement?** Open a pull request — small and focused beats
  large and sweeping. For anything substantial, an issue first saves you from
  building something we can't take.

The guardrails in `SKILL.md` — confirming before destructive calls, never
claiming an invite was emailed, reporting refusals rather than retrying — are
load-bearing rather than stylistic. A change that removes one needs to say
what replaces it.

We review every PR. Changes reach this repo through our release process, so
your PR may be closed with your change committed and credited rather than
merged in place — either way, you'll see your change (and your name on it) in
the repo and in the next release.

By contributing you agree your contributions are licensed under the
[MIT License](./LICENSE).
