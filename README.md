# BizOwl Brief Desk

**Four AI agents and one human turn a founder's funding application into an anonymised, investor-ready brief in a Discord channel. In about 4 minutes, with nothing going out until a person approves it.**

Built with [Switch](https://www.switchagents.ai/) for the [Flip the Switch hackathon](https://www.wemakedevs.org/hackathons/switch) (WeMakeDevs × DevTools Academy × Switch), Track 1: Best Business Workflow.

📺 **Demo video:** [add link]

---

## The problem

[BizOwl](https://bizzowl.com) helps early-stage founders in India's Tier-2 and Tier-3 cities raise money. Founders apply through BizOwl's fundraising form (the link is shared in our 400+ member WhatsApp community), and the BizOwl team turns each application into a one-page brief written from an investor's point of view. Founder names and contact details are hidden, so investors judge the startup first.

Doing that by hand takes **[X hours] per application** <!-- TODO: replace with the time BizOwl confirms -->. It is slow, it is easy to leak a phone number or a team member's name, and it is easy to miss the questions an investor will ask straight away, like a startup calling itself "Scaling" while raising a pre-seed round.

## What Brief Desk does

A team member pastes an application into `#brief-desk`. Four agents pass it along, each doing one job, all in the same room the team works in:

| Agent | Job |
| --- | --- |
| **intake** | Checks every field the BizOwl form asks for. If something is missing, drafts a short WhatsApp message for the team to send the founder. Flags contradictions as "Questions an investor will ask". Moves personal details into a confidential field. |
| **analyst** (role: `brief-analyst`) | Writes a 9-part brief from an investor's point of view, using only facts from the application. Anything unclear is marked "to confirm". |
| **privacy** (role: `privacy-reviewer`) | Strips founder, team, mentor and advisor names, phone numbers, emails, LinkedIn and pitch deck links. Checks every number against the application and flags anything that was made up. |
| **matcher** | Picks the top 3 investors from the room's investor list, explains each fit (and any mismatch), drafts an intro email for each, then waits. |

Then a person types `@matcher approve`, or `@matcher approve Saffron only` to approve just one investor. Only then are the brief and the approved emails saved to `outbox/` and posted in the linked `#investor-outreach` room, with a note of who approved them.

![Architecture](docs/architecture.svg)

## What it catches

These all happened in test runs on fictional applications ([`samples/applications/`](samples/applications/)):

- **Made-up details.** The analyst wrote that the CTO built "2 apps with 50k+ downloads each". The application never said "each". Privacy caught it, plus two other claims that weren't in the application, and marked all three "to confirm". (KisanKart)
- **Personal data hidden in free text.** A founder put her name, email, phone and LinkedIn inside the solution and team answers, and asked to hide the company name. Privacy removed all of it, and matcher even named the output files `healthtech-tier2` instead of the company. (MediSaathi)
- **Credit is not revenue.** Rs 1.1 crore of credit tracked through a kirana app could easily read as revenue. Intake flagged it, and every agent after it, including the investor emails, kept it as "credit tracked through the app". (PaySetu)
- **Honest matching.** No investor in the list covers spacetech. Matcher said so, called two options weak, and recommended sending to one. The human approved that one only. (OrbitKraft)
- **Duplicates.** When the same application was pasted twice, intake noticed and didn't run it again.

We also ran one **real** BizOwl application through the desk privately (see [Privacy](#privacy)). The agents took about **4 minutes** from paste to "Type approve to finalise", found six questions an investor would ask, and a search of the finished brief found no team names, links or contact details.

## How Switch is used

Switch is the whole workflow, not a wrapper around it. Without it, these would be four separate agent sessions with no shared room, no hand-offs and no visible approval step.

| Switch feature | How Brief Desk uses it |
| --- | --- |
| **Rooms on a team chat app** | Each Discord channel is a Switch room. The team and the agents work in the same place, and every hand-off is visible. |
| **Room instructions** | Every agent reads the desk's rules when it joins: never write founder personal details, never invent facts, nothing is final until a person approves. |
| **Agents handing off by mention** | intake → analyst → privacy → matcher, each one addressing the next by name in the room. |
| **Roles** | `brief-analyst` and `privacy-reviewer` are exclusive room roles. Each agent takes its role when it starts, so the job can move to a different agent later without redesigning the hand-offs. |
| **Room document** | The investor list is a document in the room, so every agent sees the same data. |
| **Linked rooms** | `#brief-desk` and `#investor-outreach` are linked both ways ("approved briefs go here" / "where briefs are prepared"). Matcher posts approved work only to its own desk's outreach room. |
| **Human in the loop** | Approval happens in chat, can name specific investors, and is recorded in the outreach post. |
| **Permissions** | Agents answer anyone in their rooms and ask before risky actions. |
| **Template** | The whole setup, four agents, four rooms, roles, links and the investor list, is one file: [`templates/bizowl-brief-desk.yaml`](templates/bizowl-brief-desk.yaml). |

We built most of this by talking to Switch's own admin agent (`switch-buddy-admin`) in Discord: it created the agents and rooms, set the roles, and exported the template.

## Run it yourself

You need a Mac (Switch Console also has early-access Windows and Linux builds), [Docker Desktop](https://www.docker.com/products/docker-desktop/), and a Claude subscription or API key for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview).

**1. Install the tools**

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude   # sign in in the browser, then type /exit
```

Install [Switch Console](https://github.com/sandbox-quantum/switch/releases) (v0.40.1 or later). In **Settings → Agent providers**, Claude Code should show **Signed in**. Then **Add a server → On this computer** and wait until it's running.

**2. Clone this repo.** This folder is the agents' working folder.

```bash
git clone https://github.com/YOUR-USERNAME/bizowl-brief-desk.git
```

**3. Connect Discord**

1. Create a Discord server, then in the [Discord Developer Portal](https://discord.com/developers/applications) create an application and a bot, and copy its token.
2. On the bot page, turn on **Server Members Intent** and **Message Content Intent**.
3. Under OAuth2 → URL Generator, tick `bot` and `applications.commands`, and give it View Channels, Send Messages, Send Messages in Threads, Manage Webhooks, Manage Channels, Manage Roles, Read Message History, Attach Files and Add Reactions. Open the URL to add the bot to your server.
4. Turn on Developer Mode in Discord, right-click your server and copy its ID.
5. In Switch Console: **Home → Messaging apps → Connect → Discord**, paste the token and server ID. Then **Link my account**.

**4. Create the desk from the template.** Add [`templates/bizowl-brief-desk.yaml`](templates/bizowl-brief-desk.yaml) to Switch Console's **Templates** (or ask `switch-buddy-admin` to install it from that file), then **Use template** with:

- **Machine:** This computer
- **Working folder:** the folder you cloned
- **Messaging app:** your Discord

This creates the four agents and four rooms: `#brief-desk` and `#investor-outreach` for testing, `#desk-live` and `#outreach-live` for demos.

**5. Try it.** Post any message in `#brief-desk` first, then paste a file from [`samples/applications/`](samples/applications/). When matcher asks, reply in the channel with `@matcher approve` (or name the investors you want).

Tips: reply to an agent inside its thread so it keeps the context, and mention agents by their exact name.

## Privacy

- Every startup, founder and investor in this repo is **fictional**.
- To run a real application, start the message with `@intake REAL: new application:`. Every file for it is saved under `private/` (git-ignored), and it is only ever posted to `#investor-outreach`, never the live demo room.
- The agents never send anything to anyone. Approved emails are drafts for the team.

## Repo layout

```
templates/bizowl-brief-desk.yaml   the full Switch setup (source of truth)
agents/                            readable copies of each agent's instructions
room/instructions.md               room rules, roles and links
room/documents/investors.sample.csv  the investor list (fictional)
samples/applications/              fictional applications to paste in
examples/outputs/                  approved briefs and emails from test runs
docs/architecture.svg              the diagram above
```

## Team

- [Name]: [role]
- [Name]: [role]

Built for BizOwl, an Indian startup ecosystem platform for founders in Tier-2 and Tier-3 cities.

## License

[MIT](LICENSE)
