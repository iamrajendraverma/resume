## MCP Overview

Model Context Protocol (MCP) is a communication layer that provides Claude with context and tools without requiring you to write a bunch of tedious integration code. Instead of building every tool function yourself, MCP shifts that burden to specialized servers that handle the heavy lifting.

## Who Creates MCP Servers

Anyone can create an MCP Server implementation. Often , servive providers themselves will create official MCP implementaion. For example , AWS might release their own offcial MCP server with tools for their various services

You can also create your own MCP Server to wrap access to any service you need to integrate with.

## How is using an MCP Server different from calling a service's API directly?
MCP Servers provide tool schemas and functions already defined for you. If you call an API directly, you'll be writing those tool definitions yourself. MCP saves you that implementation work.

## Aren't MCP Servers and tool use the same thing?

This is a common misconception. MCP Servers and tool use are complementary but different concepts. MCP Servers provide pre-built tool schemas and functions, while tool use is about how Claude actually calls those tools. MCP is really about who does the work of creating and maintaining the tool implementations.

The key benefit is that MCP Servers give you access to sophisticated integrations without having to build and maintain all that code yourself. You get the power of tool use with much less development overhead.



## MCP client 

The MCP client serves as the **communication bridge** between your server and MCP servers. Think of it as your access point to all the tools that an MCP server provides. When you need to use external functionality, the client handles all the **message passing** and **protocol details** for you.

## Transport Agnostic Communication

One of **MCP**'s key strengths is being **transport agnostic** - a fancy way of saying the client and server can talk to each other using **different communication methods**. The most common setup runs both the MCP client and server on the same machine, where they communicate through **standard input/output**.

But you're not limited to that approach. MCP clients and servers can also connect over:

- HTTP
- WebSockets
- Various other network protocols

## Message Types 

**ListToolsRequest/ListToolsResult:** The client asks the server "what tools do you provide?" and gets back a complete list of available functionality.

**CallToolRequest/CallToolResult:** The client tells the server "run this specific tool with these arguments" and receives the execution results.

## Real-World Example Flow

Let's walk through a complete example to see how all these pieces work together. Imagine a user asks "What repositories do I have?" - here's the entire communication chain:


1. The **User** asks Claude: "What repositories do I have?"

2. Claude sends a **ListToolsRequest** to the **MCP client** to discover available tools.
3. MCP Server responds with list of tools
4. Claude analyzes the tool descriptions and determines that the "list_repositories" tool can answer the user's question.
5. Claude sends a **CallToolRequest** to the MCP client, specifying the "list_repositories" tool and providing any required arguments.
6. The MCP client relays the request to the MCP Server, which executes the underlying function (e.g., makes an API call to GitHub).
7. The MCP Server returns the results (e.g., a list of repository names) to the MCP client.
8. The MCP client forwards the results to Claude.
9. Claude processes the results and generates a natural-language response for the user: "You have 3 repositories: 'repo1', 'repo2', and 'repo3'."   

## Setting Up the MCP Server
The Python MCP SDK makes server creation straightforward. You can initialize a server with just one line:

```python 
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DocumentMCP", log_level="ERROR")
```
This creates a fully functional MCP server that can handle tool definitions, client connections, and message routing.

## Building the Document Reader Tool

The first tool reads document contents by ID. It takes a document identifier and returns the corresponding content from our in-memory dictionary:

```python 
@mcp.tool(
    name="read_doc_contents",
    description="Read the contents of a document and return it as a string."
)
def read_document(
    doc_id: str = Field(description="Id of the document to read")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    
    return docs[doc_id]
    ```
    The function includes basic error handling to catch requests for non-existent documents. When Claude calls this tool with a valid document ID, it receives the full document content as a string.

## Creating the Document Editor Tool

The second tool performs simple find-and-replace operations on documents. It requires three parameters: the document ID, the text to find, and the replacement text:

```python
@mcp.tool(
    name="edit_document",
    description="Edit a document by replacing a string in the documents content with a new string."
)
def edit_document(
    doc_id: str = Field(description="Id of the document that will be edited"),
    old_str: str = Field(description="The text to replace. Must match exactly, including white space."),
    new_str: str = Field(description="The new text to insert in place of the old text.")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    
    docs[doc_id] = docs[doc_id].replace(old_str, new_str)

```
This implementation uses Python's built-in string replace method, which requires exact matches including whitespace. The tool modifies the document in place within our dictionary.

## Key Benefits of the SDK Approach

- No manual JSON schema writing required
- Type hints provide automatic parameter validation
- Field descriptions help Claude understand tool usage
- Error handling integrates naturally with Python exceptions
- Tool registration happens automatically through decorators

The MCP Python SDK transforms tool creation from a complex schema-writing exercise into straightforward Python function definitions. Your tools become more maintainable and easier to test, while Claude gets all the metadata it needs to use them effectively.

## The server inspector 


When building MCP servers, you need a way to test your functionality without connecting to a full application. The Python MCP SDK includes a built-in browser-based inspector that lets you debug and test your server in real-time.

Starting the Inspector


```python

mcp dev mcp_server.py
```

After clicking "Connect" to start your MCP server, you'll see a navigation bar with sections for:

*Resources
*Prompts
*Tools
*Other server capabilities

## Testing Your Tools

When you select a tool, the right panel shows its details and provides input fields for testing. For example, to test the read_doc_contents tool:

- Select the tool from the list
- Enter a document ID (like "deposition.md")
- Click "Run Tool"
- Check the results for success and expected output

## Testing Tool Interactions

You can test multiple tools in sequence to verify they work together correctly. For instance, after using the edit_document tool to modify content:

## Development Workflow

The inspector creates an efficient development loop:

- Make changes to your MCP server code
- Test individual tools with various inputs
- Verify tool interactions work as expected
- Debug issues without needing a full application setup

This browser-based testing environment is essential for MCP server development. It saves time by letting you catch issues early and verify functionality before integrating with Claude or other applications.

## The MCP client consists of two main components:

- **MCP Client** - A custom class we create to make using the session easier
- **Client Session** - The actual connection to the server (part of the MCP Python SDK)

The client session requires resource cleanup when we're done with it, which is why we wrap it in our custom class. This handles connection management and cleanup automatically.

## How the Client Fits Into Our Application

## Implementing Core Client Functions


We need to implement two essential functions:

-**list_tools** - Retrieves available tools
- **call_tool** - Executes a specific tool with arguments

## List Tools Function

This function gets all available tools from the server:

```python

    async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools

```
It's straightforward - we access our session (the connection to the MCP server), call the built-in list_tools function, and return the tools from the result.

## Call Tool Function

This function executes a specific tool on the server:


```python

    async def call_tool(
    self, tool_name: str, tool_input: dict
) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)
```

We pass the tool name and input parameters (provided by Claude) to the server and return the result.


## Testing the Client

To verify our implementation works, we can test it directly. The client file includes a testing harness that connects to the MCP server and runs commands against it.

Running `uv run mcp_client.py` should return a list of available tools with their descriptions and input schemas. You should see tools like `read_doc_contents` and `edit_document` that we defined in our server.

## Resources in MCP 

Resources in MCP allow your server to expose data that can be directly included in prompts, rather than requiring tool calls to access information. This creates a more efficient way to provide context to AI models like Claude.

## Understanding the Resource Flow

When a user wants to access resource content, the flow works like this:

1. User requests information about a resource (like "@report.pdf")
2. Your code needs a list of document names for autocomplete
3. MCP Client sends a ReadResourceRequest to the MCP Server
4. Server responds with a ReadResourceResult containing the resource data
5. Your code can then put this data directly into prompts

## Implementing Resource Reading

To read resources from your MCP client, you'll need to implement a `read_resource` function. First, add the necessary imports:

```python
    import json
from pydantic import AnyUrl

```

The core function makes a request to your MCP session and processes the response:

```python
    async def read_resource(self, uri: str) -> Any:
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]
```

This approach handles two main scenarios:

- JSON resources that need parsing
- Plain text resources that can be returned as-is

## Resource vs Tool Usage

Resources are particularly useful when:

- You have static or semi-static content that's frequently referenced
- You want to reduce the number of API calls
- The content should be immediately available in the prompt context

This differs from tools, which are better for dynamic operations or when you need the AI to decide whether to access certain information based on the conversation context.

## Defining prompts

Prompts in MCP servers let you define pre-built, high-quality instructions that clients can use instead of writing their own prompts from scratch. Think of them as carefully crafted templates that give better results than what users might come up with on their own.

## Why Use Prompts?

Let's say you want Claude to reformat a document into markdown. A user could just type "convert report.pdf to markdown" and get decent results. But they'd probably get much better output if they used a thoroughly tested, specialized prompt that you've designed specifically for document formatting.

The key insight is that while users can accomplish these tasks on their own, they'll get superior results when using prompts that have been carefully engineered and tested by the MCP server authors.

## How Prompts Work

Prompts define a set of user and assistant messages that clients can use directly. When a client requests a prompt, your server returns a list of messages that can be sent straight to Claude.


## The basic structure looks like this:

```python
@mcp.prompt(
    name="format",
    description="Rewrites the contents of a document in Markdown format",
)
def format_document(
    doc_id: str = Field(description="Id of the document to format"),
) -> list[base.Message]:
    # Return a list of messages

```

## Building a Format Command

Here's a practical example. We'll create a format command that lets users type `/format doc_id` to reformat any document into markdown syntax.
The prompt implementation includes detailed instructions for Claude:

```python
    def format_document(
    doc_id: str = Field(description="Id of the document to format"),
) -> list[base.Message]:
    prompt = f"""
Your goal is to reformat a document to be written with markdown syntax.

The id of the document you need to reformat is:

{doc_id}


Add in headers, bullet points, tables, etc as necessary. Feel free to add in structure.
Use the 'edit_document' tool to edit the document. After the document has been reformatted...
"""
    
    return [
        base.UserMessage(prompt)
    ]
```
## Prompts in the client

The final step in building our MCP client is implementing prompt functionality. This allows us to list all available prompts from the server and retrieve specific prompts with variables interpolated into them.
