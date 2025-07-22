

## Editable Memory: Giving Agents the Power to Revise and Forget

This lecture builds upon the concept of perpetual memory by introducing **Editable Memory**. While the previous system allowed an agent to overcome context window limits by paging memory out, that memory was largely append-only. This lesson explores a more dynamic and powerful paradigm where the agent can not only add to its long-term storage but also surgically **edit** and **delete** specific memories.

### The Problem: Static Long-Term Memory

A simple append-only memory system, while powerful, has limitations that prevent agents from truly adapting and learning over time.

  * **Incorrect Information**: What if the agent stores a fact that is later proven to be incorrect? In an append-only system, the agent might add the correction, but the original, incorrect fact remains in its memory, potentially causing confusion later.
  * **Evolving Facts**: The world changes. A person's job, preferences, or goals can evolve. An agent needs a way to update its understanding of a user, rather than just accumulating a long list of chronological, and possibly contradictory, facts.
  * **Privacy and User Control**: A user may want an agent to forget a sensitive piece of information. A simple "ignore this" instruction is unreliable; true forgetting requires the ability to permanently delete the data from the agent's memory store.

### The Solution: A CRUD-Enabled Memory Store

The solution is to upgrade the agent's external context from a simple log or storage bin into a true **database** that the agent can manipulate. This is accomplished by giving the LLM a new suite of tools that correspond to **CRUD** operations (Create, Read, Update, Delete), which are the fundamental actions in database management.

  * **Create**: The ability to add a new memory. This is the `add_memory` function.
  * **Read**: The ability to search for and retrieve memories. This is the `search_memory` function.
  * **Update**: The ability to modify an existing memory. This is the new, powerful `edit_memory` function.
  * **Delete**: The ability to permanently remove a memory. This is the crucial `delete_memory` function.

By providing these functions as tools, the agent gains fine-grained control over its knowledge base, allowing it to maintain a more accurate, relevant, and secure long-term memory.

-----

### How Editable Memory Works: The Agent's Toolkit

The provided `Implementing_Editable_Memory.py` script likely implements a `Memory` class that connects to a database (like SQLite) and exposes the following functions to the LLM agent. Each memory entry is typically assigned a unique `memory_id` to allow for precise targeting.

1.  **Adding a Memory**: When new information is presented, the agent calls the `add_memory` function. The system adds the content to the database and returns the unique `memory_id`.

    *Conceptual LLM Function Call:*

    ```json
    {
      "function": "add_memory",
      "arguments": {
        "content": "User's favorite programming language is Python."
      }
    }
    ```

    *System Response:*

    ```json
    {
      "status": "success",
      "memory_id": 1
    }
    ```

2.  **Editing a Memory**: This is the core of editable memory. If the agent learns new, superseding information, it first retrieves the relevant memory (getting its `memory_id`) and then calls `edit_memory` to update it.

    *User Input:* "Actually, I've been using Rust a lot more lately and I prefer it."

    *Conceptual LLM Function Call:*

    ```json
    {
      "function": "edit_memory",
      "arguments": {
        "memory_id": 1,
        "new_content": "User's favorite programming language is Rust, they previously preferred Python."
      }
    }
    ```

3.  **Deleting a Memory**: If the user wants the agent to forget something, the agent can use the `delete_memory` function to permanently remove the entry from its database.

    *User Input:* "Please forget my favorite language, I don't want you to store that."

    *Conceptual LLM Function Call:*

    ```json
    {
      "function": "delete_memory",
      "arguments": {
        "memory_id": 1
      }
    }
    ```

This loop of reading, understanding user intent, and choosing the correct CRUD function allows the agent to act as an intelligent database administrator for its own mind.

-----

### Key Use Cases

  * **Fact Correction**: An agent can self-correct its knowledge base. If it saves "The capital of Australia is Sydney" and is later corrected, it can edit that specific entry to "The capital of Australia is Canberra."
  * **Dynamic User Profiles**: An agent can maintain an evolving profile of the user. It can update preferences, track changing goals, and maintain a current understanding of the user's state without accumulating outdated information.
  * **Task Management**: An agent managing a to-do list can add tasks (`add_memory`), update their status (`edit_memory`), and remove them upon completion (`delete_memory`).

---

Of course\! Here is a detailed explanation of the `Implementing_Editable_Memory.py` notebook. This guide breaks down each section of the code to explain how an agent with self-editing memory is built from scratch.

-----

### A Walkthrough of the Editable Memory Notebook

This [notebook](Implementing_Editable_Memory.ipynb) provides a hands-on demonstration of how to give a Large Language Model (LLM) the ability to manage its own memory. Instead of using complex, pre-built frameworks, it uses the OpenAI API's native **tool-calling** feature to create a simple yet powerful agent that can remember, update, and reason about information over multiple steps.

-----

### Section 1: Structuring the LLM's Context Window

The notebook begins by explaining the core challenge: an LLM's limited **context window**. To build a capable agent, we must intelligently manage what goes into this context.

The notebook structures the context window with two key components:

  * A **system prompt** that instructs the agent on its behavior and personality.
  * A **conversation history** that contains the ongoing dialogue.

More advanced agents, like MemGPT, also reserve space for a **core memory** section. This is a special, read-writeable part of the context that the agent can modify.

The first code blocks demonstrate how to inject a simple memory object into the system prompt. By placing `[MEMORY]` and its contents directly in the instructions, the agent can use this information to answer questions, like recalling the user's name.

```python
# The agent's memory is a simple dictionary
agent_memory = {"human": "Name: Bob"}

# The system prompt tells the agent about the memory section
system_prompt = "You are a chatbot. " \
+ "You have a section of your context called [MEMORY] " \
+ "that contains information relevant to your conversation"

# The memory is formatted and included in the API call
messages=[
    {"role": "system", "content": system_prompt + "[MEMORY]\n" + json.dumps(agent_memory)},
    {"role": "user", "content": "What is my name?"},
]
```

-----

### Section 2: Making Memory Editable with Tools

This section introduces the core concept of **self-editing memory**. Instead of just reading from a static memory block, the agent is given a **tool** to modify it.

#### Defining the `core_memory_save` Tool

A simple Python function, `core_memory_save`, is defined. This function takes a `section` ('human' or 'agent') and a `memory` string and appends the new information to a global `agent_memory` dictionary.

```python
# A global dictionary to store the agent's memory
agent_memory = {"human": "", "agent": ""}

# The function that will act as our tool
def core_memory_save(section: str, memory: str): 
    agent_memory[section] += '\n' 
    agent_memory[section] += memory
```

To make this Python function usable by the LLM, a detailed **tool schema** is created. This schema is a JSON object that tells the OpenAI API:

  * The function's `name` (`core_memory_save`).
  * A natural language `description` of what it does.
  * The `parameters` it expects (in this case, `section` and `memory`), including their types and descriptions.

#### Executing a Tool Call

When the user provides new information (e.g., "My name is Bob"), the notebook sends this message to the LLM along with the `core_memory_save` tool schema.

The LLM, understanding the user's intent, doesn't respond with a text message. Instead, it returns a **tool call**, which is a structured request to execute the `core_memory_save` function with the appropriate arguments (`section='human'`, `memory='Name: Bob'`).

Crucially, the OpenAI API **does not run the function itself**. Your code is responsible for:

1.  Parsing the `tool_calls` object from the LLM's response.
2.  Extracting the function name and arguments.
3.  Executing the actual Python function (`core_memory_save(**arguments)`).

After execution, the `agent_memory` dictionary is updated. On the next turn, when the user asks, "what is my name," the updated memory is passed in the context, and the agent correctly answers "Bob".

-----

### Section 3: Implementing an Agentic Loop for Multi-Step Reasoning

The final and most advanced part of the notebook is creating an **agentic loop**. The problem with the previous section is that the agent can only perform one action at a time: either call a tool or respond to the user. An agentic loop allows for **multi-step reasoning**, where the agent can chain multiple actions together in a single turn.

The `agent_step` function implements this loop:

1.  It takes a `user_message` and prepares the initial context, including the system prompt and the current state of `agent_memory`.
2.  It enters a `while True:` loop, which represents the agent's "thought process".
3.  Inside the loop, it calls the LLM with the current message history.
4.  **If the LLM responds with a tool call**:
      * The code prints that a tool is being called.
      * It executes the `core_memory_save` function to update the `agent_memory`.
      * It appends both the agent's tool call and a "tool response" message to the conversation history. This lets the agent know its action was successful.
      * The loop continues, allowing the agent to take another action immediately.
5.  **If the LLM responds with a regular message** (i.e., no tool call):
      * This signifies the end of the agent's reasoning process for this turn.
      * The function returns the content of the message to be displayed to the user, and the loop breaks.

This structure allows the agent to receive a message like "my name is bob," first call the `core_memory_save` tool, and then, in the same turn, generate a final response to the user like "Got it\! Your name is Bob." This mimics a more natural and intelligent conversational flow.