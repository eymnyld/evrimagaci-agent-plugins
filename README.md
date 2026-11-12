# Evrim Ağacı for Agent Plugins

[Evrim Ağacı](https://evrimagaci.org) is Turkey's largest independent popular science platform. This plugin lets your assistant search and read its editor-reviewed Turkish articles, curated Q&A answers, featured blogs, science dictionary, podcast show notes and Akademi reading lists, and cite every source it uses. With your permission, your assistant can also save articles, publish in your name and follow approvals on evrimagaci.org.

This community adaptation of [evrimagaci/claude-plugin](https://github.com/evrimagaci/claude-plugin) packages the original workflows using the [Agent Plugins 1.0.0 standard](https://agent-plugins.org/). It is not an official Evrim Ağacı release. It contains instructions and configuration, with no executable runtime code.

| Files | Used by |
|---|---|
| `plugin.json`, `mcp.json`, `skills/` | Agent Plugins 1.0 clients; OpenAI presentation is in `extensions.com.openai` |
| `.agents/plugins/marketplace.json` | ChatGPT/Codex repo marketplace; the plugin lives at the repository root |

This branch targets Agent Plugins 1.0 only. It provides six shared skills and the Evrim Ağacı MCP endpoint using `streamable-http`. Installation, authentication, permission prompts and tool availability depend on the client. For the Claude package, use the original upstream repository linked above.

## What it does

| Skill | Use it to |
|---|---|
| `evrimagaci-kaynak-kontrolu` | check a scientific claim against Evrim Ağacı sources: supported, contradicted or insufficient evidence, with a URL for every point |
| `evrimagaci-konu-ozeti` | summarize a science topic from Evrim Ağacı articles, with sources |
| `evrimagaci-okuma-plani` | build an ordered reading plan for a beginner, intermediate or advanced reader, preferring an existing Akademi list |
| `evrimagaci-bugun` | get a short daily digest: new articles in the topics you follow, picks, unread notifications, the astronomy picture and the question of the day |
| `evrimagaci-onaylar` | handle actions that wait for your approval on evrimagaci.org, and permission prompts |
| `evrimagaci-yazma` | publish, edit, delete and undo comments, posts, questions, answers, messages and drafts, only with text you approved word for word |

The assistant answers in the language you write in. The sources are in Turkish. It quotes them briefly and links each one.

## How to use it

### Agent Plugins 1.0 clients

Clone or download this repository and install its root directory using your client's plugin installation flow. The client should discover `plugin.json`, the six skills in `skills/`, and the `evrimagaci` server in `mcp.json`. If the client only supports MCP, add `https://evrimagaci.org/mcp` as a Streamable HTTP server; MCP-only setup does not install the skills.

### ChatGPT and Codex

This repository includes a marketplace catalog. With a Codex version supporting the current plugin commands, register it using:

```sh
codex plugin marketplace add eymnyld/evrimagaci-agent-plugins --ref agent-plugins
```

To test an unpublished local checkout instead:

```sh
git clone --branch agent-plugins https://github.com/eymnyld/evrimagaci-agent-plugins.git
codex plugin marketplace add ./evrimagaci-agent-plugins
```

Open the Plugins Directory in the ChatGPT desktop app, select **Evrim Ağacı (community)**, and install **Evrim Ağacı**. Restart the app if the local source does not appear. Use natural-language requests to invoke the skills.

The package is not automatically listed in the public plugin directory. Public ChatGPT distribution requires registering and testing the remote MCP integration and submitting the plugin through OpenAI's publishing flow. No registered app ID or credentials are bundled here. See [OpenAI packaging and local marketplace instructions](https://developers.openai.com/plugins/build/plugins) and [submission instructions](https://developers.openai.com/plugins/deploy/submission).

### Accounts and permissions

The server supports public search and reading without an account, although a client may require authentication when connecting. For account-specific tools and writes, use your client's authentication flow with your Evrim Ağacı account (free: https://evrimagaci.org/uye-ol). If your client cannot complete authentication, account tools remain unavailable. Do not put access tokens in the repository.

Grant only the permissions needed for your request. The consent screen shows the assistant's name and requested permissions. Once connected, ask in your own words.

Example prompts:

- "Is it true that we only use 10 percent of our brains? Check it against Evrim Ağacı."
- "Evrim Ağacı'na göre CRISPR nedir? Kaynaklarıyla özetle."
- "I want to learn evolution from scratch. Make me a reading plan from Evrim Ağacı."
- "Bugün Evrim Ağacı'nda ne var?"

You can see every connected assistant, change or time-limit its permissions, see its last actions and remove the connection at https://evrimagaci.org/uye/yapay-zeka.

## Permissions, approvals and undo

| Permission | Lets the assistant |
|---|---|
| None, no sign-in | search and read public content |
| See my account | read your saved items, reading history and lists, follows, notifications, ATP and UP, drafts |
| Organize my library | save and unsave, edit your reading lists, follow topics, mark notifications read |
| Interact for me | react, upvote, report content to moderators, unfollow |
| Write for me | publish comments, posts, questions and answers under your name, edit and delete them, keep drafts |
| Read Sohbet, Write in Sohbet for me | read and write in Sohbet community channels |
| Read my direct messages, Send direct messages for me | read and send direct messages |
| Spend ATP | take actions that cost ATP, each one approved by you on evrimagaci.org |

- **Approvals.** Every action that spends ATP, every direct message sent or edited, and everything else published in your name while "approve what is published" is on (the default), waits for you on evrimagaci.org. The assistant gives you the link and the exact cost. Nothing runs until you approve there.
- **ATP is not money.** ATP is Evrim Ağacı's in-site point balance, earned on the site. The connector never makes payments with money.
- **Label.** Content published or edited through an assistant shows the small label "Asistan aracılığıyla" ("via an assistant"), unless you turn it off.
- **Undo.** Every action is listed under "Son işlemler" on https://evrimagaci.org/uye/yapay-zeka. Reversible actions have an undo button there for seven days. Deletes are final.

## What data it sends

- The plugin runs no code of its own. The assistant sends data only through the Evrim Ağacı connector, to `https://evrimagaci.org/mcp`, a server that Evrim Ağacı runs. The plugin sends nothing to any other service.
- For each tool call, the assistant sends only that tool's arguments: search words, document ids or URLs, and for account tools and writes, the action's content (for example the text of a comment you approved or a list item). The connector does not read your conversation or your other files.
- Evrim Ağacı keeps a record of each action taken through the connector (the tool, a short summary and a link), shown to you under "Son işlemler" and deleted after 180 days. Unpublished drafts are deleted after 90 days without a change.
- Evrim Ağacı stores each approval request with its full arguments (for example the text of a message or a comment) and its outcome. It deletes the request 30 days after it was made.
- Each tool call writes one event to Evrim Ağacı's operational logs. The event holds your account id (none when you are signed out), the assistant, the connection's client id, the tool name, the outcome, the duration and the network address that sent the call. For some tools it also holds the number of results or the type of the document read. It holds no arguments and no results. Evrim Ağacı keeps these logs for 60 days. Search queries and access tokens are not written to the logs.
- When you use the connector without signing in, Evrim Ağacı counts the requests from each network address (for IPv6, from each /64 network) in a one-minute window, and its tool calls in a one-day window. The counter keeps the address for at most one day.
- How Evrim Ağacı handles personal data: the privacy policy at https://evrimagaci.org/s/6964.

## Troubleshooting

- **A tool asks you to sign in.** Use your client’s MCP authentication flow. If the assistant does not continue by itself, ask again.
- **"Sign in to continue" when you are signed out.** The limit for use without an account is reached. Sign in. Your account has its own quota.
- **A permission is missing.** Your client may ask you to connect again. Allow the permission on the consent screen. You can also allow it for the connection at https://evrimagaci.org/uye/yapay-zeka. The change applies to the next request.
- **An action waits for your approval.** Open the approval link that the assistant gives you and select Onayla (approve) or Reddet (reject). Waiting requests are also listed under "Onayınızı bekleyenler" at https://evrimagaci.org/uye/yapay-zeka. A request expires after 30 minutes. After that, ask your assistant again to make a new one.
- **Something else is wrong.** Read the setup steps at https://evrimagaci.org/mcp, or write to info@evrimagaci.org.

## Tracking upstream updates

Keep this fork's Agent Plugins work on `agent-plugins`. Reserve `main` for the upstream Claude history; do not sync or reset `agent-plugins` to upstream. Separate branches preserve the adaptation, but do not import upstream changes automatically.

To review new upstream changes without overwriting this branch:

```sh
git remote add upstream https://github.com/evrimagaci/claude-plugin.git # once
git fetch upstream
git log HEAD..upstream/main -- skills/
git diff HEAD...upstream/main -- skills/
```

Adapt relevant skill and endpoint changes on a temporary branch based on `agent-plugins`, review and validate them, then merge that branch back into `agent-plugins`. Avoid copying the entire upstream tree: it would reintroduce Claude manifests, commands and client-specific instructions. Keep `plugin.json`, `mcp.json` and the marketplace catalog in Agent Plugins format. The MIT license and upstream attribution must remain intact.

## Verify the adaptation

Before publishing, validate `plugin.json` and `mcp.json` against their canonical schemas:

- https://agent-plugins.org/schemas/1.0.0/plugin.schema.json
- https://agent-plugins.org/schemas/1.0.0/mcp.schema.json

Confirm that `mcp.json` points to `https://evrimagaci.org/mcp` and that each skill's frontmatter name matches its directory. Then test installation, public search and fetch, authentication, and an account read in each target client. Schema validation alone does not establish runtime compatibility. Test writes only with text and actions you explicitly approve; never bypass the server's approval flow.

## Links

- About the connector and setup: https://evrimagaci.org/mcp
- Privacy policy: https://evrimagaci.org/s/6964
- Terms of use: https://evrimagaci.org/s/6961
- Content license: https://evrimagaci.org/s/6955
- Support: info@evrimagaci.org

## License

The plugin's files (these instructions and the connector entry) are released under the MIT License (see `LICENSE`). The MIT License does not cover the content on evrimagaci.org: articles, answers, dictionary entries and other site content keep their own license, at https://evrimagaci.org/s/6955.
