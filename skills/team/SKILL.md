---
name: team
description: Manages your company's bisque.team membership — see the roster, invite people, remove them, and control which groups can open which presentations. Use when asked to invite or add someone to the team, company, or org, to list or manage members, to make someone an admin, to remove or reinstate a member, or to work with team groups or seats.
---

# Team

bisque.team is the company layer: presentations published to a company
library instead of a public channel, visible to the groups you decide. This
skill manages who is in the company and what they can open.

You reach it through the Bisque MCP server, so every call runs as the person
whose credentials you hold. Only an administrator can change anything, and the
server enforces that on every call regardless of what you send.

**Membership changes cost money and revoke access.** Each active member is a
billed seat, and removing someone cuts their access to every company
presentation immediately. Confirm before you change anything.

## 1. Find the organization

```sh
curl -s -X POST https://bisque.today/mcp \
  -H "Authorization: Bearer $BISQUE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"list_orgs","arguments":{}}}'
```

Resolve credentials the way the `present` skill does: `$BISQUE_API_KEY` with
`$BISQUE_USER_ID` from the environment, otherwise `~/.bisque/config.json`,
whose profiles look like `profiles.<name>.apiKey` / `.userId`. Pick
`$BISQUE_PROFILE`, or the `profile` in a `.bisque.json` in the working
directory; failing that the profile named `present`, then `default`, then the
only usable one. There is no anonymous path here — every tool needs an
account.

**When several accounts are configured and none is pinned, ask which one.**
Every call here runs as that person, so guessing means administering the
wrong company, or reporting "not found" for an org the other account
administers.

`list_orgs` returns each organization's slug, name, seat count, and whether
you can administer it. When the user says "my team" without naming one, call
this first rather than guessing a slug. When `isAdmin` is false, say so — the
management tools will refuse, and it is better to explain that than to hit a
"not found".

## 2. Read the roster before changing it

```json
{ "name": "get_org_roster", "arguments": { "org": "acme" } }
```

It returns every member (account id, email, active or removed, groups), every
group with its active count, and every invite with its status and link. This
is where a person's account id comes from, and it is how you check what is
actually true before reporting it.

## 3. Make the change

Every tool below takes `org`. The ones that act on a person take `uid` or
`email` — pass the email the user gave you and the server resolves it against
this org's roster.

| Tool                       | What it does                                   |
| -------------------------- | ---------------------------------------------- |
| `invite_to_org`            | Create an invite link for one email address    |
| `revoke_org_invite`        | Kill a pending invite's link                   |
| `remove_org_member`        | Revoke a membership and free its seat          |
| `reinstate_org_member`     | Restore a removed member, billing a seat again |
| `set_org_group_membership` | Grant or revoke one group for one member       |
| `create_org_group`         | Add a group                                    |
| `delete_org_group`         | Remove a group                                 |

**Inviting.** `invite_to_org` returns a join link and sends nothing. Give the
link to the user and tell them plainly they need to send it. The invitee has
to sign in with exactly that address, verified, so an address you guessed at
produces an invite nobody can accept — ask for it rather than inferring it
from a name.

**Admins are a group, not a role.** Promote someone by granting them the
`admins` group and demote them by revoking it:

```json
{
  "name": "set_org_group_membership",
  "arguments": {
    "org": "acme",
    "email": "sam@acme.com",
    "group": "admins",
    "active": true
  }
}
```

The last remaining admin can be neither removed nor demoted; the server
refuses with `LAST_ADMIN`. Report that refusal as it is — it is protecting the
company from locking itself out.

**Groups are also access.** A company presentation is shared with one group,
so a group is both "the marketing team" and "who can open the marketing
briefing". `delete_org_group` is refused while any presentation still points
at it (`GROUP_IN_USE`); the fix is to move those presentations, not to retry.

## 4. Say what happened

Report the seat count after a membership change — it is what the company is
billed for. After an invite, give the URL and the "nobody was emailed" caveat
in the same breath. After a promotion, say what the person can now do.

## Rules

- **Confirm before every destructive call.** Name the person and the
  organization in the confirmation: "Remove sam@acme.com from Acme? They lose
  access to every company presentation." That covers removing, revoking an
  invite, demoting an admin, and deleting a group.
- **Never guess an email address.** An invite to the wrong address is an
  invite a stranger can accept.
- **Never say someone was emailed.** Bisque sends no invite mail at all. The
  user delivers the link.
- **A refusal is an answer.** `LAST_ADMIN`, `GROUP_IN_USE`, `NOT_A_MEMBER`
  and `RESERVED_GROUP` each mean the change was wrong, not that the call
  needs retrying with different arguments.
- **Reading the roster is not reporting it.** It carries every member's email
  address; answer the question the user asked instead of pasting the roster
  into the conversation.
- **A "not found" on an org you can see means you are not an admin.** The
  server answers identically for "no such organization" and "you may not
  manage it", on purpose. Say which one you believe it is and why.
