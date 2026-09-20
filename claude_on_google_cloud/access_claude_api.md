# Claude Notes

**19 September 2026**

> Study notes — Claude Certified Architect (Foundations), Anthropic Academy

**Contents**
- [System Prompts](#system-prompts)
- [Temperature](#temperature)
- [Code Reference](#code-reference)
- [Streaming](#streaming)
- [Controlling Model Output](#controlling-model-output)
- [Prompt Evaluation](#prompt-evaluation)
- [Prompt Engineering](#prompt-engineering)
- [Be Clear and Direct](#be-clear-and-direct)

---

## System Prompts

System prompts are a powerful way to customise how Claude responds to user input. Instead of getting generic answers, you can shape Claude's **tone**, **style**, and **approach** to match your specific use case.

### What is a System Prompt?

A **system prompt** (often called *system instructions* or *system role*) is a foundational set of instructions, guidelines, or constraints given to a Large Language Model (LLM) **before** a user begins interacting with it.

> Think of it as the *"director's brief"* — or the backstage setup for an actor.

While regular user prompts change with every message, the system prompt remains active in the background, shaping the AI's core behavior, persona, knowledge boundaries, and output formatting **across the entire session**.

### Why is a System Prompt Important?

System prompts are critical for transforming a raw, general-purpose AI into a reliable, safe, and specialized tool.

| # | Purpose | What it does |
|---|---|---|
| 1 | **Establishes Persona and Tone** | Defines *how* the AI communicates — formal and professional, casual and witty, or a specific character (a technical architect, a patient tutor) |
| 2 | **Sets Guardrails and Safety Rules** | Restricts the AI from discussing harmful topics, sharing sensitive data, or generating inappropriate content |
| 3 | **Enforces Output Formats** | Dictates strict formatting — always responding in a specific JSON structure, using Markdown tables, avoiding certain phrasing. Vital for developer integrations |
| 4 | **Defines Domain Expertise** | Anchors knowledge to a specific context — mobile development, IoT hardware, data analysis — and prioritizes relevant frameworks or tools |
| 5 | **Maintains Consistency** | Without one, an LLM may drift in style or forget its objective during long conversations. The system prompt is a permanent anchor |

---

## Temperature

**Temperature** is a powerful parameter that controls how *predictable* or *creative* Claude's responses will be. Understanding how to use it effectively can dramatically improve your AI applications.

### How Claude Processes Text

When you send text to Claude, it goes through three main steps:

1. **Tokenisation**
2. **Prediction**
3. **Sampling**

### What Temperature Does

Temperature is a decimal value between `0` and `1` that directly influences these selection probabilities.

> It's like adjusting the **"creativity dial"** on Claude's responses.

### Temperature Ranges & Use Cases

| Range | Level | Best for |
|---|---|---|
| `0.0` – `0.3` | **Low** | Factual responses · Coding assistance · Data extraction · Content moderation |
| `0.4` – `0.7` | **Medium** | Summarisation · Educational content · Problem solving · Creative writing with constraints |
| `0.8` – `1.0` | **High** | Brainstorming · Creative writing · Marketing content · Joke generation |

<details>
<summary>Expanded view</summary>

**Low temperature (0.0 – 0.3)**
1. Factual responses
2. Coding assistance
3. Data extraction
4. Content moderation

**Medium temperature (0.4 – 0.7)**
1. Summarisation
2. Educational content
3. Problem solving
4. Creative writing with constraints

**High temperature (0.8 – 1.0)**
1. Brainstorming
2. Creative writing
3. Marketing content
4. Joke generation

</details>

---

## Code Reference

A helper that wraps `messages.create()` with an optional system prompt and configurable temperature:

```python
def chat(messages, system=None, temperature=1.0):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }

    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

**Notes on the snippet**
- `system` is a **top-level parameter**, not a message in the `messages` list
- It is only added to `params` when provided, so the call stays valid without it
- `temperature` defaults to `1.0` here — lower it for factual or code-generation work

---

## Streaming

### The Problem with Standard Responses

The main problem with the standard response is that it is **time consuming and not real time** — you wait for the entire generation before seeing anything.

With streaming enabled, Claude immediately sends back an initial response indicating it has received your request and is starting to generate text.

### Understanding Stream Events

When you enable streaming, Claude sends back several types of events:

| Event | Meaning |
|---|---|
| `message_start` | A new message is being sent |
| `content_block_start` | Start of a new block containing text, tool use, or other content |
| `content_block_delta` | Chunks of the actual generated text |
| `content_block_stop` | The current block has been completed |
| `message_delta` | Top-level updates to the message (e.g. `stop_reason`, `usage`) |
| `message_stop` | End of the message — the stream is finished |

### Basic Streaming Implementation

```python
messages = []
add_user_message(messages, "Write a 1 sentence description of a fake database")

stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True
)

for event in stream:
    print(event)
```

---

## Controlling Model Output

Beyond crafting better prompts, there are two powerful techniques for controlling Claude's output: **prefilled assistant messages** and **stop sequences**. These methods give you precise control over *how* Claude responds and *when* it stops generating text.

### Message Prefilling

Prefilling means starting the assistant's turn for it — Claude continues from where you left off, which steers the shape of the answer.

| Prefill | Resulting behaviour |
|---|---|
| *(none)* — user asks "Is tea or coffee better at breakfast?" | Claude gives a **balanced** response covering both tea and coffee |
| `"Tea is better because"` | Claude responds **only about tea**, committed to that position |

### Stop Sequences

Stop sequences are strings that tell Claude to **stop generating text**.

Uses of stop sequencing:
- Its main usage is to **control the output** of the model
- Stop the model from generating **irrelevant or unwanted** text
- Stop the model from generating **repetitive** text
- Stop the model from generating **long** responses

---

## Prompt Evaluation

### Draft a Prompt — Three Options

| Option | Approach | Trade-off |
|---|---|---|
| **1** | Test the prompt once and decide it's good enough | Carries a significant risk of breaking in production when users provide unexpected inputs |
| **2** | Test the prompt a few times and tweak it to handle a corner case or two | Better than Option 1, but users will often provide very unexpected inputs that you haven't considered |
| **3** | Run the prompt through an **evaluation pipeline** to score it, then iterate based on objective data | Requires more work and cost upfront, but gives much more confidence in the prompt's reliability |

### Prompt Eval Workflow

1. Draft a prompt
2. Create an eval dataset
3. Feed through Claude
4. Feed through a grader
5. Change prompt and repeat

### Grade Range

`1` to `10`

- **1** — very low quality output
- **10** — very high quality output

### Types of Graders

| Grader | Flexibility | Cost / Speed |
|---|---|---|
| **Code grader** | Limited to programmatic checks | Fast and cheap |
| **Model grader** | High — judges subjective qualities | Extra API call per eval |
| **Human grader** | Highest — any criteria imaginable | Time-consuming and tedious |

**Code graders** let you implement any programmatic check you can imagine. Common uses include:
- Checking output length
- Verifying output does or doesn't contain certain words
- Syntax validation for JSON, Python, or regex
- Readability scores to ensure appropriate reading levels

**Model graders** offer tremendous flexibility by using an additional API call to evaluate outputs. They're useful for assessing:
- Response quality
- Quality of instruction following
- Completeness
- Helpfulness
- Safety

**Human graders** provide the most flexibility but come with significant downsides. While humans can evaluate responses for any criteria imaginable, the process is time-consuming and tedious.

> **In combination:** a code grader checks format and valid syntax, while a model grader checks the user's task and whether the generated code meets the user's requirement.

---

## Prompt Engineering

1. Set a goal
2. Write an initial prompt
3. Evaluate the prompt
4. Apply prompt engineering techniques
5. Re-evaluate

### Worked Example — Meal Planning for Athletes

**Sample Input**

| Attribute | Value |
|---|---|
| Weight | 75 kg |
| Height | 180 cm |
| Sex | Male |
| Goal | Maintain weight and improve power |
| Dietary Restrictions | Lactose intolerance |

**Prompt Output**

> Here is a one-day meal plan for an athlete aiming to maintain weight and improve power.

**Daily Caloric Total:** approximately **2,800 calories**
*(calculation based on height, weight, and goal)*

**Macronutrient Breakdown**

| Macro | Amount | Share |
|---|---|---|
| Protein | 150 g | 20% |
| Carbs | 325 g | 50% |
| Fats | 89 g | 30% |

**Meal Plan**

| Meal | Time | Foods |
|---|---|---|
| **Meal 1** | 7:00 AM | Oatmeal 80 g (dry) · Lactose-free protein powder 30 g · Berries 100 g |
| **Meal 2** | 10:00 AM | Banana 1 medium · Almond butter 2 tbsp · Rice cakes 2 |
| **Meal 3** | 1:00 PM | Chicken breast 150 g · Quinoa 100 g · Steamed vegetables 200 g |

> **TODO** — the original notes flagged calculating calories, protein, carbs and fats *per meal*. Those figures weren't captured in the source notes, so the per-meal macro columns are still to be filled in.

---

## Be Clear and Direct

When crafting that crucial first line, focus on two key principles: **clarity** and **directness**. This means using simple language that leaves no room for ambiguity about what you want Claude to do.

### Clear Communication

Being **clear** means:
- Use simple language that anyone can understand
- State exactly what you want without beating around the bush
- Lead with a straightforward statement of Claude's task

### Direct Instructions

Being **direct** focuses on how you structure your request:
- Use **instructions, not questions**
- Start with direct action verbs like *Write*, *Create*, or *Generate*

❌ Rather than asking: *"I was reading about renewable energy and geothermal energy sounds neat…"*

### Be Specific

**Not great**

```python
prompt = """
Write a short story about a character who discovers a hidden talent
"""
```

**Better**

```python
prompt = f"""
Write a short story about a character who discovers a hidden talent

Guidelines:
1. Keep the story under 1,000 words
2. Include a clear action that reveals the character's talent
3. Include at least one supporting character
"""
```

### Two Types of Guideline

There are two main approaches to being specific in your prompts, and you will often see them used together in professional applications.

#### 1. Quality Guidelines

The first type focuses on listing **qualities that your output should have**. These guidelines control attributes like:

- **Length constraints** — keep under 1,000 words
- **Structural requirements** — include a clear action that reveals the character's talent
- Outline a pivotal scene that reveals that talent
- Brainstorm 3 supporting character types that could increase the impact of this discovery
- **Inclusion/exclusion criteria** — must include X, must not include Y

Example of quality guidelines:

```python
prompt = f"""
Write a short story about a character who discovers a hidden talent

Guidelines:
1. Keep the story under 1,000 words
2. Include a clear action that reveals the character's talent
3. Include at least one supporting character
"""
```

#### 2. Format Guidelines

The second type focuses on the **structure and format** of your output. These guidelines control how Claude should structure its response.

**Why structure matters in a prompt**

Consider a prompt where you need to analyze 20 pages of sales records. Without clear boundaries, Claude might have trouble distinguishing between your instruction and the actual data you want analyzed.

**Providing structure**

Use **XML tags** to separate distinct portions of the prompt:
- Most useful when including a lot of context
- Helps serve as delimiters for Claude

**Example**

```xml
Write a one page decision report to troubleshoot why a sales team's numbers
have dropped 30% last quarter.

Here are the last 20 pages of our sales records:

<sales_records>
{sales_records}
</sales_records>

Follow these steps:

1. Compare current vs previous market metrics
2. Identify relevant industry changes
3. Analyze individual team member performance
4. Consider recent organizational changes
5. Review customer feedback
```
