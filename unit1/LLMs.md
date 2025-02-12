
## Large Language Models (LLMs)
An **LLM** is a Transformer-based AI model designed for **understanding and generating language**.

### Types of Transformers
1. **Encoders:** Convert input into dense representations (e.g., BERT)
2. **Decoders:** Generate text token by token (e.g., Llama)
3. **Seq2Seq (Encoder-Decoder):** Used for translation, summarization (e.g., T5, BART)

### Popular LLMs
| Model      | Provider  |
|-----------|-----------|
| Deepseek-R1 | DeepSeek |
| GPT-4      | OpenAI   |
| Llama 3    | Meta     |
| SmolLM2    | Hugging Face |
| Gemma      | Google   |
| Mistral    | Mistral  |

### Tokenization
- LLMs process **tokens** instead of full words.
- Example: *interest + ing = interesting*
- Special tokens mark sequence start/end.

### How LLMs Generate Text
1. **Autoregressive:** Predicts next token step by step.
2. **Decoding Strategies:**
   - Greedy search (highest probability token)
   - Beam search (optimizes sequence score)
3. **EOS Token:** Marks end of generation.

### Importance of Attention Mechanism
- Helps LLMs **focus on key words** in a sentence.
- Enables models to handle **longer context lengths**.

### Training of LLMs
- **Pretraining:** Predicts next token from massive text data.
- **Fine-tuning:** Adapts to specific tasks (e.g., conversation, code generation).

### Importance of Prompting
- **Well-structured prompts** guide LLMs toward desired outputs.
- Example: **“Summarize this in 3 sentences”** vs. **“Explain this”**.

---
### 📌 Key Takeaways
- **Agents = AI Model + Tools (for executing actions).**
- **LLMs predict the next token using Transformers.**
- **Proper tool integration makes AI more useful.**
- **Effective prompting improves AI responses.**


## LLMs: Messages, Special Tokens, and Chat Templates

**Key Concepts:**

*   **Chat Messages are an Abstraction:**  When you use ChatGPT, you see messages back and forth. But under the hood, the entire conversation history is combined into one big prompt that's fed to the LLM.

*   **Chat Templates are Translators:** They convert your conversation into a format the specific LLM can understand. Different models use different formatting.

*   **Special Tokens are Delimiters:** Like end-of-sequence (EOS) tokens, special tokens mark the start and end of user and assistant turns within the formatted prompt.

**Messages:**

*   **System Messages (System Prompts):**  Define the model's behavior (e.g., "You are a helpful assistant").  They are persistent instructions.

*   **User/Assistant Messages:** The back-and-forth conversation turns. The order is maintained using Chat Templates

**Base Models vs. Instruction Models:**

*   **Base Model:** Trained to predict the next token from lots of raw text.
*   **Instruct Model:** Fine-tuned to follow instructions and have conversations.

To make a base model act like an instruct model, you need to format the prompt with a chat template so the model understands the input

**Chat Templates:**

*   **Purpose:** Structure conversations for specific LLMs.
*   **Function:**
    *   Format the conversation into a single prompt string with special tokens.
    *   Handles multi-turn conversations and context.
    *   Implemented using Jinja2 code that describes how to transform the ChatML into a textual representation of the system-level instructions, user messages and assistant responses that the model can understand.
*   **Usage:**
    1.  Structure messages as a list of dictionaries with "role" (system, user, assistant) and "content".
    2.  Load the model's tokenizer.
    3.  Use `tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)` to convert the messages into a correctly formatted prompt.
