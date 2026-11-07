# discord-agent-starter

An open-source starter for a company agent that works in a Discord server: the manifest, the agent prompt, and the approval rules that decide what it may touch, in one repository you own.

Kortix is the open-source CompanyOS: your agents, their skills, your company memory, and every connector in one git repo you own, with the agents working on real cloud computers. This repository is the smallest honest version of that for Discord. It is not a chat bot with a personality file. It is an agent with a job, a named set of connectors, and a rule that holds a risky action until a person approves it.

## What is in here

| File | What it decides |
|---|---|
| `kortix.yaml` | The agent, the connectors it may reach, the read-only policy set, the blocked operations, and the approval lifetime. |
| `agents/discord-ops.md` | The agent's prompt: how it answers, what it may touch, when it asks, what it never does. |

Two files, because those are the two things that go wrong. A bot that answers well but reaches everything is a liability, and a bot that reaches nothing is a search box. The manifest is where you draw the line, and it is reviewable in a pull request like any other code.

## Add it to a server

The three steps are the ones Kortix's own Guide documents for the Discord channel. If this is your first time, [how to add an AI agent to Discord](https://agentsindiscord.com/how-to-add-an-ai-agent-to-discord) walks the same three steps and names the failure at each one.

**1. Add Kortix to your server.** Open Customize, then Connectors, select the Channels scope, and select Add to Discord. On Discord's page, pick the server. You need the Manage Server permission there. Kortix allows your own Discord account at once.

**2. Mention the app with a task.** In a text or announcement channel, mention the app and say what you want:

```
@Kortix what changed in the release?
```

A mention starts a thread on the message, and one thread is one session. A later mention in that thread continues the same session, so the agent keeps the context of the conversation without you repeating it.

**3. Allow the people.** The allowed-people list decides who may talk to the agent. Anyone else gets a reply showing their user ID and appears under Allowed users, then Wants access. Discord has no account link, so this list is the whole access control: there is no `/login` to fall back on.

## What the agent may touch

`kortix.yaml` declares one connector besides the Discord channel itself: a helpdesk API, with a policy set called `read-only`.

```yaml
connectors:
  - slug: helpdesk
    provider: http
    base_url: https://helpdesk.example.com/api
    auth:
      type: bearer
    policy_sets:
      read-only:
        description: Reads tickets and customers. Never writes.
        policies:
          - match: "get_*"
            action: always_run
          - match: "*"
            action: block
```

Read that as three separate decisions. `get_*` runs without asking, because reading a ticket is not a change. Everything else on that connector is blocked, so the agent cannot create, update or delete a ticket even if a prompt tells it to. And `policies` blocks `helpdesk.delete_*` at the project level, above the agent, so no agent in this project can call it.

The agent gets no connector, secret, skill or repository until the manifest grants it. That is the default, not a setting you turn on.

## The approval rule

`approvals: ttl: 24h` sets how long a held action waits for a person. An action that needs approval is held, not queued and forgotten: it shows under Approvals in the session header and on the Review page, and it expires after a day rather than sitting there indefinitely.

The prompt tells the agent what to do with that: say the action is waiting and where the person approves it, and never claim an action it did not take. A held action is the normal case for anything that changes a system, and the agent should treat it as one.

## Limits worth knowing before you promise anything

- A server can serve several projects; the Discord connector acts only in the project's own server, threads and direct messages.
- One project makes at most 60 Discord calls a minute.
- Kortix threads archive after a day without messages. A new message opens the thread again.
- Without the Message Content intent, the agent cannot read other people's messages. Discord shows it only the messages that mention it, direct messages and its own. To ask about a conversation, paste the part you mean.
- The app asks for view channels, send messages, send messages in threads, create public threads, read message history, attach files, add reactions and embed links. When a channel denies one, Kortix says so.

## Run it

```sh
curl -fsSL https://kortix.com/install | bash
kortix login
git clone https://github.com/Kortix-seo/discord-agent-starter.git
cd discord-agent-starter
kortix validate
kortix ship
```

`kortix validate` prints what each agent enables, which is the fastest way to see a grant you did not mean to give. `kortix ship` validates the manifest, commits, offers to set the missing secret and connects the declared connectors.

Then add the channel as above, and set `HELPDESK_TOKEN` in Secrets before the first mention that needs it.

## Where to read more

The setup walk, with the failure modes at each step, and the comparison between an open-source agent and a hosted bot are both in [the open-source Discord AI agent guide](https://agentsindiscord.com/open-source-discord-ai-agent), which starts from the manifest question. [What an agent may reach in a company's tools](https://agentsindiscord.com/discord-agent-with-company-tools) covers connectors and grants, and [what an approval covers](https://agentsindiscord.com/approvals-in-discord) covers the held action and the record it leaves. If your team is deciding where the agent should live, [Discord or Slack for an AI agent](https://agentsindiscord.com/discord-vs-slack-for-ai-agents) compares the two rooms. And [self-hosting a Discord agent](https://agentsindiscord.com/self-hosted-discord-agent) names which half of the stack stays on your server.

The channel reference, including the Discord section, is at https://kortix.com/docs/guide#channels. The manifest reference, every key, is at https://kortix.com/docs/developers#kortixyaml-reference. Kortix itself is at https://kortix.com.
