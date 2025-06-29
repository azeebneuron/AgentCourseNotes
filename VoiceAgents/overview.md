
**What are AI Voice Agents?**
* AI systems that enable real-time, human-like conversations by combining speech with Large Language Models (LLMs).

**Core Components:**
* **Speech-to-Text (STT):** Transcribes spoken audio into text.
* **Large Language Model (LLM):** Processes the text to generate intelligent responses.
* **Text-to-Speech (TTS):** Converts the LLM's text response back into audible speech.
* **Voice Activity Detection (VAD):** Detects when a person starts and stops speaking.

**How to Build Voice Agents:**
* **Single Speech-to-Speech API:** A simpler approach using one service for all components.
* **Pipeline Approach:** A more flexible method using separate STT, LLM, and TTS components, which is the main focus of the course.

**Major Challenge: Latency**
* Minimizing the delay between when a user speaks and the agent responds is crucial for a natural conversation.
* Platforms like LiveKit can help reduce latency.

**Other Key Considerations:**
* Choosing the right API providers.
* Optimizing for speed.
* Handling speech disfluencies (e.g., "um", "ah").
* Developing for multilingual applications.