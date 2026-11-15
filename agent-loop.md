# Agent loop

An **agent loop** is the cycle that turns a model that answers once into an agent that acts. You send the model a request and the tools it may use. If it asks to use a tool, your code runs the tool, feeds the result back, and asks again. This repeats until the model replies without asking for a tool.

```python
messages = [user_request]
while True:
    reply = model(messages, tools)          # model answers or requests tool calls
    messages.append(reply)
    if not reply.tool_calls:
        break                               # no more tools needed, so it's done
    for call in reply.tool_calls:
        result = run_tool(call)             # e.g. read a file, run bash
        messages.append(result)             # the model sees the result next turn
```

## Example

A coding agent asked to "fix this failing test" might go around the loop like this:

1. it reads the test file
2. it runs the test
3. it reads the error and edits the code
4. it runs the test again
5. it reports back

Each step is one trip around the loop, and the model decides the next step from what the previous one returned.

## The loop is the easy part

The loop is about 20% of a code agent (see the [README](README.md#caveats)). The hard parts are around it:

* **when to stop** — for example, tau and pi let a tool end the run, but only if every tool in the batch agrees (see [tau.md](tau.md))
* **running tools safely** — sandboxing and permission prompts before `bash` runs
* **long runs** — compacting the history so it still fits in the model's context window
* **showing progress** — streaming events so a UI can display what's happening

## Where the loop lives

Where the loop lives decides the [layers](README.md#layers-of-using-llm):

* **layer 1** — you write the loop yourself, or use a provider's mini version such as Anthropic's Tool Runner (`client.beta.messages.tool_runner`)
* **layers 3–4** — a harness or code agent runs it for you, e.g. tau's `tau_agent` or pi's `pi-agent-core`
