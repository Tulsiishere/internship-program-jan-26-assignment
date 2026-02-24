# GenAI Assignment:

**Evaluation Criteria**

We will score your submission on:

* Clarity and practicality of architecture
* Robust JSON schema design
* Prompt quality (zero-shot, reliable, minimal hallucination risk)
* Handling of ambiguity + user review flow
* Bulk generation thinking (errors, naming, report)

## Problem 1: **Proposal for “Video-to-Notes”**

We have a local folder of long videos (3–4 hours each, 200MB+). Watching them fully is slow. We need an automated way to generate a “summary package” per video: **Summary.md** + highlight clips + screenshots, all organized per video. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

### **Task**

Prepare a **pre-processed solution proposal** comparing  **three approaches** **:**

1. **Online/Cloud-Based (Already Available Solutions)**
2. **Build Our Own Using LLM APIs (Hybrid: local media processing + cloud LLM)**
3. **Build Fully Offline Using Open-Source Models (Local transcription + local LLM + pipeline)**

No code required. We want a **clear, practical proposal** with architecture and tradeoffs.

### Your Solution for problem 1:
## Objective:
**The objective here is to design a scalable system that processes long videos (3-4 hours, 200MB++) from a local folder and automatically generates:**
* Structured **Summary.md**
* Timestamped highlights
* Highlight video clips
* Screenshots aligned to highlights
* Organized per-video output structure
* Batch processing support
**The solution must be:**
* Scalable
* Deterministic
* Robust to LLM hallucinations
* Suitable for long-form media

## Approach Comparison:
### **Approach 1: Online/Cloud-Based (Already Available Tools) -**

**Exapmle Platforms**
* Notion AI + manual upload
* Descript
* Otter.ai
* Fireflies.ai
* Loom AI
  ### Architecture
  ```
  Video Upload
      ↓
  Cloud Platform
      ↓
  Cloud Transcription
      ↓
  Built-in Summarizer
      ↓
  Export Notes 
  ```
**Pros -**
1. No engineering required
2. Clean UI
3. High-quality and reliable transcription
4. Managed infrastructure
5. Fast to setup and deploy

**Limitations -**
1. Expensive for large video volums
2. Privacy concers
3. Limited control over:
     * JSON output format
     * Highlight segmentation
     * Timestamp
4. No deterministic batch processing from local folder
5. Limited customization
6. No deterministic clip generation pipeline

**Verdict -**

Good for quick MVP, small teams or quick prototypes.

However, it does not offer:
* Schema-level output control
* Custom review flows
* Deep automations
Hence, not suitable for bulk automated large-scale internal processing.

### **Approach 2: Hybrid (Local media + Cloud LLM APIs) -**

This is the architecture I implemented practically.

The approach combines:
* Local media processing (heavy tasks)
* Cloud LLM intelligence (semantic reasoning)
As per my opinion, this is the most practical and production-ready approach.
  ### Architecture
  ```
  Batch Video Folder
        ↓
  Audio Extraction (FFmpeg)
        ↓
  Local Transcription (Whisper)
        ↓
  Transcript Chunking Layer
        ↓
  LLM API (Gemini / GPT)
        ↓
  Strict JSON Validation
        ↓
  Clip Generator (FFmpeg)
        ↓
  Screenshot Extractor
        ↓
  Markdown Builder
        ↓
  Organized Output Folder
  ```

Large videos (3-4 hours) are expensive to upload and process in the cloud.

Instead if used a Hybrid model:
* Heavy processing (audio extraction, clipping, screenshots) stays local.
* Only text (transcript chunks) is sent to LLM.
* This reduces cost and improves performance.

### Component Breakdown (Based on the project I built) -

**1. Media Processing (Local)**
  * FFmpeg for:
      * Audio extraction
      * Highlight clipping
      * Screenshot extraction
  * Handles 200MB+ files reliably
  * Avoids uploading heavy video to cloud
  * Deterministic timestamp alignment

**2. Transcription (Using Whisper)**
  * Whisper base model
  * Produces timestamped segments
  * OpenAI Whisper (base/medium)
  * Works well for long-form audio
  * No API cost
  * No privacy issues

This is to ensure that:
  * Accurate start_time_seconds
  * Precise highlight boundaries

**3. Chunking Strategy**

Long transcripts exceed token limits
    
Soluntion:
    * Chunk transcript by N-minute windows
    * Preserve timestamps
    * Merge intelligently in prompt
    This reduces hallucination and prevents context overflow.

**4. LLM Layer (Cloud API)**

Using Gemini / GPT APIs
This layer is for:
  * Extracting structured highlights
  * Generating video summary
  * Generating start and end timestamps
  * Assigning confidence scores

The LLM does sematic reasoning, not media processing.

**5. JSON Schema**

One major faliure mode of LLM pipelines is invalid structure.

To solve this, I designed a strict schema validated using Pydantic.
```
{
  "video_summary": "string",
  "highlights": [
    {
      "title": "string",
      "start_time_seconds": 0,
      "end_time_seconds": 0,
      "why_important": "string",
      "key_points": [],
      "action_items": [],
      "confidence_score": 0.0
    }
  ],
  "overall_takeaways": []
}
```
This ensures:
* No malformed output
* No missing timestamps
* No hallucinated structure
* Automatic rejection of invalid responses

And directly aligns with the evaluation criterion:
  Robust JSON schema design

**6. Prompt Design Strategy**

My prompt enforces:
* JSON-only output
* No markdowns
* No explanations
* Timestamps must exist in transcript
* Reduce confidence if unsure
* Do not invent facts

This is to reduce hallucination risk significantly.

And aligns with:
  Prompt quality (reliable, minimal hallucination risk)

**7. Screenshot and Clip Alignment**

After validation, for each highlight:
* Cut clips using start_time_seconds and end_time_seconds
* Extract screenshot at midpoint timestamp

This is to guarantee:
* Visual assets match summary
* No drift between summary and video



Example:
## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

You need to put your solution here.

## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:

You need to put your solution here.

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

You need to put your solution here.
