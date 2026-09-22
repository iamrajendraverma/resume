# Meet Claude

Study notes — Claude 101, Anthropic Academy · 22 September 2026

---

## What Claude excels at

- **Writing and content creation** — Claude can collaborate with you on social media posts, professional emails, and complex reports. Because Claude is trained to take direction on personality and tone, you can iterate together on structure and clarity until your voice comes through clearly.
- **Research and analysis** — Claude helps you explore research angles, compile findings, and analyze data to surface meaningful insights. You can upload documents and Claude will help you make sense of complex information. This is enabled by Claude's large context window, which can ingest 200K+ tokens (about 500 pages of text or more), with up to 1M tokens available on Pro, Max, Team, and Enterprise plans when using supported models — so Claude can consider extensive materials in a single conversation.
- **Coding assistance** — Coding is one of Claude's greatest strengths. Its strong performance on real-world coding tasks means it can help you write, debug, and explain code across many programming languages.
- **Problem-solving and reasoning** — Claude handles complex cognitive tasks, mathematical problems, strategic thinking and analysis, and research. Claude can respond near-instantly or take time to reason first — a capability called **Thinking**. When a problem calls for careful analysis, Claude can work through it step by step before it answers.
- **Learning new things** — Whether you're picking up a new skill, exploring unfamiliar domains, or working through complex challenges, Claude can adapt to your learning style and pace. **Learning mode** guides your reasoning process rather than handing over answers, which helps develop critical thinking skills.

Get inspired on ways to use Claude in your specific function by exploring the use-case gallery. For a deeper dive into what AI can (and can't) do, see the AI Capabilities course.

---

## Stages of writing a good prompt

1. **Set the stage** — give Claude the role, audience, and context.
2. **Define the task** — say exactly what you want produced.
3. **Specify the rules** — constraints, format, sourcing requirements.

### Example

> Create a market analysis report on the US indie film streaming opportunity. Include market size, growth trends, key competitors, consumer preferences, revenue models, and market gaps. Use current web research with citations. Structure it as a professional report for an investor deck for a new indie streaming app.

Then add your own concept and context on top of the template — the more specific the stage and rules, the closer the first draft lands.

---

## Projects

Projects are ideal for:

- storing knowledge Claude should reference,
- organizing related chats around a specific topic or work area, and
- collaborating with team members who need access to the same shared context.

### How projects handle large knowledge bases

You might wonder what happens when you upload a lot of content. Projects automatically scale to handle large amounts through a process called **Retrieval Augmented Generation (RAG)**. At a high level, Claude can automatically find and use the most relevant parts of your uploaded documents when answering, without you needing to tell it which file to look at.

When your project knowledge approaches the context window limit, Claude stops loading everything at once and instead searches your project's files, retrieving only what's relevant to your question. This expands your project's capacity by up to 10x while maintaining response quality.

You'll see a visual indicator when your project is RAG-enabled, but the experience should feel the same — you can still upload documents, chat with Claude, and get context-aware responses.

### Collaboration: permission levels

When sharing a project, you can choose from three permission levels:

- **Can view** — members can see project contents, access knowledge, and chat, but can't make changes. Read-only access with discussion rights.
- **Can edit** — members have full collaboration power. They can modify instructions, update knowledge, manage other members, and actively contribute.
- **Owner** — project creators control everything, including who sees the project. They can share with specific people or make the project visible to the entire organization.

### Best practices for projects

- **Start focused, then expand.** Begin with a specific use case rather than trying to create one project for everything. You can always add more content as you go.
- **Keep your knowledge base current.** Outdated documents lead to outdated responses. Review and update project knowledge periodically.
- **Write clear instructions.** Be specific about what you want; vague instructions lead to inconsistent results.
- **Name your documents descriptively.** Use `Q4-2025-Sales-Report.pdf`, not `report.pdf`, and group related files together. Claude uses filenames and proximity to understand relationships between documents.
- **Reference documents by name.** Mention specific documents to help Claude focus its search: *"Based on our Q3 report, what were the top customer concerns?"*

---

## Artifacts

Artifacts are the outputs you create with Claude: a document, a deck, a design, a dashboard, a prototype. Instead of getting a long block of code or text buried in the chat, you see the real thing take shape in a dedicated window alongside your conversation, ready to use and refine.

Claude creates an artifact when you ask for something that stands on its own — something you'll want to edit, reuse, or share rather than just read once. If you want to be sure, just say so: *"Create this as an artifact."*

On paid plans, an artifact isn't tied to the conversation that created it. Everything you make is saved in the **Artifacts** tab, so you can come back to it later, keep editing, and share it with others. Think of the conversation as where you create, and the Artifacts tab as where your outputs live. (On the Free plan, an artifact stays with the conversation that created it.)

### Designs, decks, and living documents

Claude has a dedicated way to make each of the most common work deliverables.

> *Notes incomplete — fill in the specifics for designs, decks, and living documents.*

---

## Skills

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Think of them as expertise packages — they teach Claude how to complete specific tasks in a repeatable way.

You've already seen Skills at work if you've used Claude to create Excel spreadsheets, PowerPoint presentations, Word documents, or PDFs; those file creation capabilities are powered by Skills running behind the scenes. But Skills go far beyond document creation. Custom Skills can codify entire repeatable workflows — a quarterly variance analysis methodology, a brand voice review process, a compliance checklist — so Claude follows the same rigorous steps every time.

### Types of Skills

- **Anthropic Skills** — created and maintained by Anthropic. These include enhanced document creation capabilities for Excel, Word, PowerPoint, and PDF files. Claude invokes them automatically when relevant, so you don't need to do anything special to use them.
- **Custom Skills** — ones you or your organization create for specialized workflows and domain-specific tasks. For example, a skill that applies your company's brand guidelines to presentations, structures meeting notes in a specific format, or executes your organization's data analysis workflows.

### Enabling Skills

Skills are available on all plans. To use them you need **Code execution and file creation** enabled, since Skills require Claude's secure sandboxed computing environment to function.

1. Navigate to **Settings → Capabilities**.
2. Ensure **Code execution and file creation** is toggled on.
3. Scroll to the **Skills** section.
4. Toggle individual skills on or off as needed.

- **Enterprise plans** — organization Owners must first enable both Code execution and Skills in Admin settings before individual members can access them.
- **Team plans** — this feature is enabled by default at the organization level.

Once enabled, you'll see available Skills listed in your settings, including Anthropic's built-in Skills and any custom Skills you've uploaded.

### Skills vs. Projects

If both Skills and Projects give Claude more context, when do you use each? **Projects store knowledge; Skills perform tasks.**

- **Projects are knowledge hubs.** They hold the reference materials Claude needs to understand your work — project specs, meeting notes, research documents. When you upload files to a project, Claude draws on that information across every conversation within that project.
- **Skills are procedural machines.** They encode *how* Claude should execute a task — the specific steps, order of operations, and methodology you want followed every time. Skills shine when you have repeatable workflows you want run consistently.

The two features complement each other. A skill can reference knowledge stored in a project — your "customer call prep" skill might pull from customer profiles uploaded to a project's knowledge base. The project provides the **what** (information), the skill provides the **how** (process).

---

## Connectors

**Key takeaways**

- Connectors transform Claude from an assistant into an informed collaborator by giving it access to the same tools, data, and context you use every day. Instead of starting every conversation from scratch, Claude can work directly with your actual information.
- Connectors let Claude read information and perform actions on your behalf. Depending on the connector and the permissions you grant, Claude can search your files, retrieve documents, analyze data, create new content, update records, and execute tasks across your connected applications — all from within your conversation.
- The **Model Context Protocol (MCP)** powers connectors. Think of MCP as USB-C for AI: a universal standard that lets Claude connect to many different applications through a single, consistent interface. Because it's an open standard, developers can build connectors for any tool and those connectors work seamlessly with Claude.
- There are two types of connectors. **Web connectors** link Claude to cloud services like Google Drive, Notion, Slack, and Asana. **Desktop extensions** run locally on your computer through the Claude Desktop app, giving Claude access to local files and native applications.

### Enterprise Search

Enterprise Search adds a dedicated **"Ask {Your Org Name}"** option to your sidebar, designed specifically for finding and synthesizing knowledge buried across your company's tools and data sources. Think of it as a pre-built project for your entire organization: your company's knowledge base is already loaded, so you can jump right in and get context-aware responses.

Unlike regular chats with connectors enabled, Enterprise Search is built for information gathering, using custom instructions configured by the Anthropic team.

**What can you ask?** Enterprise Search is particularly valuable for questions that span multiple sources or require synthesizing information from across your organization.

> *Notes incomplete — add the common use cases covered in the module.*

---

## Researching with Claude

Research transforms how Claude finds and analyzes information. Instead of a single search, Claude operates **agentically** — conducting multiple searches that build on each other while determining exactly what to investigate next. It explores different angles of your question automatically and works through open questions systematically.

- **It takes longer than a usual search** — a few minutes or more, depending on the question. That's because it isn't one lookup: Claude can send out many searches at once, sometimes across hundreds of sources, and pull what they find into one answer. That work isn't instant.
- **It works with Thinking**, so Claude can plan its approach before it searches. It breaks a complex request into manageable pieces, then gathers what each piece needs.
- **Citations make verification easy.** Research delivers thorough answers complete with easy-to-check citations, so you can trust Claude's findings and quickly verify sources yourself.

---

## Claude in action: use cases by role

### General professional use

These use cases apply across many roles and industries.

- **Generate project status reports** — keep stakeholders informed with clear, consistent updates.
- **Analyze patterns in user feedback** — extract insights from customer comments and survey responses.
- **Package your brand guidelines in a skill** — create a reusable Claude skill that applies your brand standards.

### Sales

Accelerate deal preparation, create compelling materials, and stay on top of competitive intelligence.

- **Build a battle card library** — create competitive intelligence resources that help your team win deals.
- **Prepare for sales deals** — research prospects and organize your talking points before important meetings.
- **Create sales reports** — turn pipeline data into clear, actionable reports.

### Marketing

Analyze performance data and efficiently repurpose content across channels.

- **Analyze campaign performance** — extract insights from campaign metrics to inform your strategy.
- **Adapt content across platforms** — efficiently repurpose content for different channels and audiences.

### Finance

Build models, draft documents, and make sense of complex spreadsheets.

- **Build financial models** — create and refine financial projections with Claude's help.
- **Draft investment memos** — structure and write investment analyses more efficiently.
- **Understand and extend an inherited spreadsheet** — decode complex spreadsheets and add new functionality.

### HR

Create better onboarding experiences and documentation.

- **Create new hire onboarding guides** — develop comprehensive onboarding materials tailored to different roles.

### Legal

Track complex timelines and manage discovery processes.

- **Track discovery timelines and analyze patterns** — organize case timelines and identify key patterns in legal documents.

### Research

Plan literature reviews and verify data analysis.

- **Plan your literature review** — organize your approach to reviewing academic sources.
- **Verify statistics from raw data** — double-check calculations and statistical analyses.

### Explore more

These examples are just the beginning. Visit the Use Case Gallery to browse the full collection and find inspiration for how Claude can help with your specific work.

---

## What's next

In the final module, you'll meet a few more ways to work with Claude — **Claude Code**, **Claude Tag**, **Claude Design** (which also works right inside your conversations), **Claude for Microsoft 365**, and **Claude in Chrome** — each tailored to where the work actually happens.
