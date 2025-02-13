**I. Serverless API with Hugging Face InferenceClient:**

*   **Convenience:**  Enables easy inference on Hugging Face models without local installation or deployment.
*   **Access:** Requires a Hugging Face token.
*   **Basic Usage (text\_generation):**
    *   Can lead to unexpected behavior (repetition) if the correct chat template and special tokens for the model are not used.
    *   Needs special tokens to identify when the model should stop.
*   **Recommended Usage (chat.completions.create):**
    *   Handles chat templates more reliably, simplifying model transitions.

**II. Building a Dummy Agent from Scratch:**

*   **Core Concept:** Appending information (tool descriptions, instructions) to a system prompt.
*   **System Prompt Structure:**
    *   Tool information (e.g., `get_weather`).
    *   Cycle instructions: Thought -> Action -> Observation.
    *   Format requirements (JSON blob for tool calls, `Final Answer:`).
*   **Prompt Engineering:**
    *   Needs careful construction, including `<|begin_of_text|>`, `<|start_header_id|>system<|end_header_id|>`, `<|eot_id|>`, `<|start_header_id|>user<|end_header_id|>`, and `<|start_header_id|>assistant<|end_header_id|>` tokens
    *   The HF tokenizer `apply_chat_template` method helps with creating this prompt.
*   **Execution Loop:**
    1.  **Initial Prompt:** Question within the formatted system/user prompt.
    2.  **Model Generation:**  Model predicts an "Action" (JSON blob) and "Thought."
    3.  **Stopping Criteria:**  Crucial to stop generation *before* the model hallucinates the tool's response, so using `stop=["Observation:"]` is key.
    4.  **Tool Execution:** Parse the JSON blob and call a tool.
    5.  **Observation:**  The result of the tool is added as an "Observation" to the prompt.
    6.  **Iterate:**  The updated prompt is fed back to the model for further reasoning.
*   **Hallucination Mitigation:**  Stopping generation at "Observation:" prevents the model from inventing results.
*   **Manual Concatenation:**  Building the new prompt by concatenating the original, model output, and tool observation.
*   **Final Answer:**  The model ultimately provides a final answer based on the gathered observations.
*   **Agent Libraries:** Simplify the process of building Agents

