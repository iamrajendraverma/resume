# Tool Use with Claude

**19 September 2026**

> Study notes — Claude Certified Architect (Foundations), Anthropic Academy

**Contents**
- [What is a Tool Function?](#what-is-a-tool-function)
- [Example Tool Function](#example-tool-function)
- [Describing the Tool to Claude](#describing-the-tool-to-claude)
- [Adding Type Safety](#adding-type-safety)

---

## What is a Tool Function?

A **tool function** — officially referred to as **tool use** or **function calling** in Anthropic's Claude API — is a capability that allows Claude to securely connect and interact with external systems such as databases, APIs, and other services.

It enables Claude to:
- Access **real-time data**
- Trigger **actions**
- Integrate with **existing workflows**

This extends Claude's capabilities **beyond its training data**.

---

## Example Tool Function

```python
from datetime import datetime


def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)
```

The above tool function can be used by Claude to get the current date and time in the specified format.

---

## Describing the Tool to Claude

You'll need to create a **JSON schema** that describes this function to Claude.

> **The easy way:** let Claude write your schema for you.

### ⚠️ Key detail — the schema key is `input_schema`

Claude's Messages API expects the JSON Schema under **`input_schema`**. The `parameters` key belongs to OpenAI-style function calling and will be **rejected by the Claude API**.

| Wrong (OpenAI shape) | Correct (Anthropic shape) |
|---|---|
| `"parameters": { ... }` | `"input_schema": { ... }` |

### Correct tool definition

```json
{
  "name": "get_current_datetime",
  "description": "Returns the current date and time formatted according to the specified format string.",
  "input_schema": {
    "type": "object",
    "properties": {
      "date_format": {
        "type": "string",
        "description": "A strftime-style format string used to format the current date and time (e.g., '%Y-%m-%d %H:%M:%S').",
        "default": "%Y-%m-%d %H:%M:%S"
      }
    },
    "required": [],
    "additionalProperties": false
  }
}
```

**Anatomy of a tool definition**

| Field | Purpose |
|---|---|
| `name` | The function name Claude will call |
| `description` | What the tool does — Claude uses this to decide *when* to call it |
| `input_schema` | JSON Schema for the arguments Claude must supply |
| `strict` *(optional)* | Set `true` to guarantee arguments validate exactly against the schema. Requires `additionalProperties: false` and a `required` list |

---

## Adding Type Safety

For better type checking, import and use the `ToolParam` type from the Anthropic library:

```python
from anthropic.types import ToolParam

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format string.",
    "input_schema": {
        "type": "object",
        "properties": {
            "date_format": {
                "type": "string",
                "description": "A strftime-style format string (e.g., '%Y-%m-%d %H:%M:%S').",
                "default": "%Y-%m-%d %H:%M:%S"
            }
        },
        "required": [],
        "additionalProperties": False
    }
})
```

> Note the Python casing: `False`, not JSON's `false`.
