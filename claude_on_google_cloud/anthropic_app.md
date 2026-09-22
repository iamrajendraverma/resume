document path to markdown is a toll by which a user can can connect claude code with Own MCP server.

use cases 

figma -context-mcp : Expose Figma design to claude. 
mcp-atlassian : Expose jira and confluence to claude. 
firecrawl-mcp-server : Adds web scraping capabilities to Claude
slack-mcp: Allow claude to post messages or reply to specific threads 


Automate Debugging : 

Discover -> Design -> Build -> Deploy -> Support & Scale 

Computer use : 

Computer use is a Claude feature that lets it directly control a desktop — taking screenshots to see what's on screen, and then moving the mouse, clicking, and typing to operate apps just like a person would.

How it works: Claude captures a screenshot, figures out what's on screen, and decides the next action (click a button, type text, open a menu, switch apps), repeating this loop step by step until the task is done.

## What are workflow and agents in claude ?

### Workflows

A workflow, in Claude's context, is the repeatable pattern by which an agent gets things done: gather context → take action (via tools) → verify the result → loop until finished. Claude Code calls this the "agentic loop." It adapts to the task — a simple question might only need context-gathering, while a bug fix could cycle through the loop many times, editing files, running commands, and checking results.

At a higher level, "workflow" also refers to the common, repeatable patterns of use — things like fixing a bug, refactoring code, writing tests, or creating a pull request — that Claude Code is designed to handle well as a developer assistant, whether run in the terminal, an IDE, the desktop app, or CI/CD.

### Agents (Subagents)

Subagents are specialized AI assistants Claude can delegate focused tasks to. Key traits:

- **Own context window** : a subagent works in its own isolated context, so it doesn't clutter or "pollute" the main conversation with search results, logs, or file contents that don't need to stick around.
- **Custom instructions & tool access** : each can have its own system prompt and restricted set of tools.
- **Independent execution** : Claude delegates a matching task to the right subagent, which works on it independently and returns just the summary/result.
- **Built-in vs. custom** : Claude Code ships with built-in subagents like Explore, Plan, and general-purpose, and you can also define your own (as markdown files in .claude/agents/) for tasks you find yourself repeating with the same instructions.

In short: a workflow is the loop/pattern Claude follows to complete a task; an agent (subagent) is a specialized helper Claude spins off to handle one piece of that work independently, then reports back.



## The Parallelization pattern 
This approch follows a general pattern called parallelization workflow: 

1. **Split** a single complex task into multiple specialized sub tasks. 
2. **Run** the sub-tasks in parallel (simultanously)
3. **Aggregate**  the results togethr in a final step.

The key insight is that the parallelized sub-tasks don't need to be identical. Each can have a specialized prompt, different tools, or unique approaches tailored to its specific purpose.

## Benefits of Parallelization

This workflow pattern offers several advantages:

1. **Focused attention**: Claude can concentrate on one specific analysis at a time instead of juggling multiple complex considerations
2. **Easier optimization**: You can improve and test the prompt for each sub-task independently
3. **Better scalability**: Adding new material types or criteria doesn't complicate existing sub-tasks
4. **Faster execution**: Since the sub-tasks run in parallel, the total time is often less than a sequential approach

## When to Use This Pattern

Parallelization works well when you have a complex task that can be broken down into independent sub-problems. Look for situations where you're asking Claude to consider multiple options, perform several types of analysis, or handle different aspects of the same problem simultaneously.

The pattern is especially useful when each sub-task benefits from specialized prompting or when you want to ensure thorough coverage of different possibilities without overwhelming the model with too much complexity at once.

## What is Chaining ? 

A chaining workflow breaks down one large task into smaller, sequential subtasks. Instead of asking Claude to handle everything at once, you split the work across multiple focused requests.

Here's a practical example: imagine building a social media marketing tool that creates and posts videos. Rather than one massive prompt, you could chain together these steps:

1. Find related trending topics on Twitter
2. Select the most interesting topic (using Claude)
3. Research the topic (using Claude)
4. Write a script for a short format video (using Claude)
5. Use an AI avatar and text-to-speech to create a video
6. Post the video to social media

## Benefits of Chaining

1. split large tasks into smaller, non-parallelizable subtasks 
2. Optinally do non-LLM processing between each task
3. Keep claude focused on one aspect of the overall task


## The Real-World Problem Chaining Solves

Here's where chaining becomes invaluable: dealing with constraint violations in complex prompts.

**Picture this scenario:** you're using Claude to write technical articles. You start with a simple prompt, but the output isn't quite right. Claude might mention it's an AI, use too many emojis, or write in a cringey tone. So you add constraints to your prompt.

Over time, your prompt grows into a long list of "DO NOT" instructions. But no matter how many constraints you add, Claude sometimes still violates them - using emojis, mentioning it's an AI, or maintaining that unprofessional tone.

## The Chaining Solution

Instead of fighting this in one massive prompt, use a two-step chaining approach:

**First request:** Send your original prompt with all constraints, accepting that you'll get an imperfect article
**Second request:** Ask Claude to revise the article with specific, focused instructions

Your follow-up prompt might look like:

`Revise the article provided below. Follow these steps to rewrite the article: 1. Identify any location where the text identifies the author as an AI and remove them 2. Find and remove all emojis 3. Locate any cringey writing and replace it with text that would be written by a technical writer`

This approach works because it allows Claude to focus on one specific aspect at a time. Even if the initial response doesn't satisfy all your requirements, the follow-up prompt gives Claude a clear, focused task for improvement.

## When to Use Chaining

Chaining workflows are particularly useful when:

1. You have a complex task with many constraints
2. Claude isn't consistently following all your requirements
3. You want to process or validate outputs between steps
4. You need to maintain focus on specific aspects of a larger task

While it might seem like extra work, chaining often produces more reliable results than trying to cram everything into a single, complex prompt. It's a pattern you'll find yourself reaching for regularly as you build more sophisticated Claude-powered applications.

## Environment Inspection

Environment inspection is the process of gathering information about the user's system or application before taking action. This includes things like screen layout, currently active application, and system settings. This is particularly useful when building tools that interact with desktop applications.

For example, if you want to build a tool that automates repetitive tasks on a user's computer, you would first need to inspect the environment to understand the current state and layout of the desktop.

## Practical Applications

Environment inspection becomes especially valuable in complex workflows. Consider an agent that creates videos and posts them to social media. The agent might need to:

1. Generate video content using various tools
2. Verify the output quality and timing
3. Check that audio and visual elements align correctly
4. Confirm successful posting to social platforms

## System Prompts for Inspection

You can guide Claude to inspect its environment through system prompts. For a video creation agent, you might include instructions like:

-   Use the bash tool to run whisper.cpp and generate caption files with timestamps to verify dialog placement
-   Use FFmpeg to extract screenshots from the video at regular intervals to confirm visual quality
-   Check file sizes and formats before attempting uploads

These inspection steps help Claude catch errors early and ensure the final output meets expectations. By building environment inspection into your agents, you create more reliable and self-correcting systems that can handle unexpected results gracefully.

**Remember**: every action an agent takes should be followed by some form of verification or inspection to confirm the desired outcome was achieved.

## Workflows vs agents

When building AI-powered applications, you'll need to choose between two main architectural patterns: workflows and agents. Each has distinct advantages and trade-offs that make them suitable for different scenarios.

## Workflows
Workflows are a predefined series of calls to Claude designed to solve a known problem or set of problems. Think of them as a recipe - you know exactly what ingredients you need and what steps to follow.

You'll want to use workflows when you can picture the flow of steps ahead of time. The key insight is breaking down a big task into much smaller, more specific subtasks.

### Benefits of Workflows
-   Claude can focus on one subtask at a time, generally leading to higher accuracy
-   Far easier to evaluate and test, since you know each exact step
- More predictable and reliable execution

### Downsides of Workflows

- Far less flexible - dedicated to solving specific types of tasks
- Generally more constrained user experience - you need to know the exact inputs to the flow

## Agents

With agents, Claude is given a set of basic tools and we expect it to formulate a plan to use these tools to complete a task. Instead of following a predetermined path, Claude creatively figures out how to handle challenges.

### Benefits of Agents

- Allow for more flexible user experience
- Far more flexible task completion - Claude can combine tools in unexpected ways to complete a wide variety of tasks
- Can create their own inputs based on user queries and ask for more input when needed

### Downsides of Agents

- Lower successful task completion rate compared to workflows
- More challenging to instrument, test, and evaluate since you often don't know what series of steps the agent will execute

### Choosing the Right Approach

While agents are really interesting from a technical perspective, remember that your primary goal as an engineer is to solve problems reliably. Users probably don't care that you've built a fancy agent - they want a product that works 100% of the time.

The general recommendation is to always focus on implementing workflows where possible, and only resort to agents when they are truly required. Workflows give you the predictability and reliability that most production applications need, while agents provide flexibility for scenarios where the exact solution path can't be predetermined.

 


