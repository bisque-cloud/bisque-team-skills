# bisque-team-skills

An **Agent Skill** for running your company's
[bisque.team](https://bisque.team) membership from your terminal. Ask your
agent to invite a teammate, check who has a seat, or make someone an admin,
and it does it.

bisque.team is the company layer around agent-made presentations: your team
publishes into a shared library instead of a public channel, and groups decide
who can open what. This skill manages the people side of that — the roster,
invites, seats, and groups.

For authoring and publishing presentations, install
[`presenter`](https://github.com/bisque-cloud/presenter) instead. The two are
separate because they are for different people: anyone can publish, while
membership is an administrator's job.

## What you can ask for

> Invite sam@acme.com to the team.

Your agent creates the invite and hands you the join link. Nothing is emailed
— Bisque sends no invite mail, so you send the link yourself, which is also
why it will tell you to.

> Who's on the team, and how many seats are we paying for?

It reads the roster: active members, removed ones, pending invites, and the
seat count your subscription bills.

> Make Sam an admin.

Administrators are a group rather than a role, so it grants Sam the `admins`
group. The last remaining admin can't be demoted or removed — the server
refuses, and your agent will tell you why instead of trying again.

> Remove Dana and tell me what that costs us.

It confirms first, naming the person and the organization, then reports the
seat count afterward.

## Install

```sh
npx skills add bisque-cloud/bisque-team-skills
```

That copies it into your agent's skills directory — `~/.claude/skills/` for
Claude Code, `~/.agents/skills/` for Codex and Gemini.

### Manual install

```sh
git clone https://github.com/bisque-cloud/bisque-team-skills
cp -r bisque-team-skills/skills/team ~/.claude/skills/team   # Claude Code
cp -r bisque-team-skills/skills/team ~/.agents/skills/team   # Codex + Gemini
```

Invoke with `/team` (Claude Code), `$team` (Codex), or let it activate on
description match.

## Requirements

- **A bisque.team organization**, and an administrator account in it. Every
  tool refuses a non-administrator, and reads and writes are both checked
  server-side on every call.
- **A Bisque API key.** The skill reads `$BISQUE_API_KEY`, then the profiles
  in `~/.bisque/config.json` — the same credentials the
  [`present`](https://github.com/bisque-cloud/presenter) skill uses, so if you
  have published a presentation from this machine you are already signed in.
- **Network access to `bisque.today`.** Everything runs through the Bisque
  MCP server; nothing is stored locally.

## What it deliberately can't do

- **Create or delete an organization.** Both are done by Bisque, not by an
  API, so an agent cannot dissolve your company by misreading a sentence.
- **Rename an organization, or change its verified email domains.** Neither
  has an endpoint at all today.
- **Accept an invite for you.** Acceptance matches your verified email
  address, which is a claim about a person rather than a key.
- **Change billing.** Seats follow membership automatically; the subscription
  itself lives in the bisque.team billing page.

## Safety

Membership changes cost money and revoke access, so the skill is written to
confirm before every destructive call, naming the person and the organization.
Its refusals come from the server: the last admin can't be removed, a group
still holding presentations can't be deleted, and an invite always names the
exact verified address that may accept it.

## License

[MIT](./LICENSE)
