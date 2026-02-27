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

**Cons -**
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

**8. Handling Ambiguity and Review Flow**

Ambiguity could occur if:
* Transcript quality is poor
* Topic shift are unclear
* Highlight overlaps occur

This could be solved by implementing:
* Confidence score per highlight and flag the ones with low-confidence items (<0.6)
* Log raw LLM output
* JSON validation with error raising
* Retry mechanism possible
* Option to review low-confidence highlights, manually.

This would show:
  Handling of ambiguity + user review flow

**9. Batch Processing and Error Isolation**

The system processes all videos in folder:
  ```
  for video in input/videos:
      try:
          process(video)
      except:
          log error
          continue
  ```
Features:
* Continues even if one video fails.
* Logs error per file.
* Structured folder per video.
* Deterministic naming.

This is to align **Bulk generation thinking**.

Output structure:
  ```
  output/
  video_name/
    Summary.md
    transcript.json
    highlights.json
    clips/
    screenshots/
  ```

**Pros -**
* Best balance of cost + quality
* Cloud LLM intelligence
* Local heavy processing
* Scalable
* Customizable
* Controlled JSON schema
* Good for production

**Cons -**
* API cost
* Internet dependency
* Requires error handling for LLM instability

### **Approach 3: Fully Offline (Open Source Only)**
  ### Architecture
  ```
  Video
    ↓
  FFmpeg Whisper (local)
    ↓
  Local LLM (Llama/Mistral)
    ↓
  JSON Parser
    ↓
  Clip Generator
    ↓
  Markdown
  ```

**Requirements**
* GPU recommended
* 16–32GB RAM minimum
* Local LLM (Llama 3 / Mistral 7B+)
* Quantized model

**Pros**
* No API cost
* Full privacy
* Offline capability
* Fully controllable environment

**Cons**
* Lower summarization quality
* Higher hallucination risk
* Complex setup
* Hardware heavy
* Slower inference
* Maintenance burden

**Verdict**

Good for:
  * Enterprise privacy use cases
  * Air-gapped environments
Not ideal for:
  * Fast deployment
  * High-quality summarization

### Final Recommendation

After practical implementation and evaluation, I recommend the **Hybrid Architecture**. As,

It provides:
  * High-quality semantic reasoning (LLM APIs)
  * Local control over media
  * Structured JSON validation
  * Reliable timestamp alignment
  * Scalable batch processing
  * Production-level extensibility

And, 

It balances:
  * Cost
  * Quality
  * Privacy
  * Engineering complexity

This architecture is the most practical and scalable solution for the given constraints.

### Final Note

This proposal prioritizes reliability, scalability, and structured output control, essential qualities when building GenAI systems intended for long-form content processing at scale.

## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

This prompt handles structured content generation only. Scheduling, timezone normalization, and publishing are handled by backend services after explicit user approval, ensuring separation of concerns and production reliability.

The architecture for Scheduling the post might look something like:

   **Architecture**
   ```
  LLM
   ↓
  Generate drafts
   ↓
  User
   ↓  
  Select draft
   ↓
  Backend
   ↓
  Decide publish_now OR schedule
   ↓
  Scheduler
   ↓
  Execute
  ```
### Prompt:
```
You are a senior LinkedIn content strategist and ghostwriter.

Your task is to generate THREE distinct LinkedIn post drafts based on:

1. User Persona Configuration
2. A Topic
3. Optional Context, Audience and Goal

The three drafts must:
- Be clearly different in structure, rhythm, and delivery style
- Preserve the exact voice and constraints of the persona
- Address the same core topic
- Be LinkedIn-ready
- Avoid repetition between drafts

Return output in the EXACT structured format shown below.
Do NOT add commentary.
Do NOT add explanations.
Do NOT use markdown formatting.
Do NOT wrap anything in code blocks.

---------------------------------------------------
INPUTS
---------------------------------------------------

PERSONA_CONFIGURATION:
{{persona_configuration}}

TOPIC:
{{topic}}

OPTIONAL_CONTEXT:
{{optional_context}}

TARGET_AUDIENCE:
{{target_audience}}

POST_GOAL:
{{post_goal}}

---------------------------------------------------
CRITICAL REQUIREMENTS
---------------------------------------------------

1) PERSONA LOCK
- Match tone, communication style, vocabulary, and professional maturity.
- Follow do/don’t guidelines strictly.
- Do not exaggerate experience level.
- Do not invent achievements.
- Avoid generic motivational fluff unless persona prefers it.
- Maintain consistency across all 3 drafts.

2) STYLE DIFFERENTIATION
Instead of fixed templates, generate three stylistically distinct formats that feel naturally different. Examples of variation include:

- Contrarian perspective
- Personal reflection
- Mini-framework
- Data-driven breakdown
- Thought-provoking question thread
- Tactical how-to
- Industry observation
- Lessons learned
- Myth-busting
- Strategic insight

Each draft must:
- Feel structurally different
- Use different opening hooks
- Use different pacing and flow
- Avoid repeating the same sentences or phrasing

3) LINKEDIN OPTIMIZATION
- Use natural short paragraphs
- Use spacing for readability
- 0–3 relevant emojis only if persona allows
- 3–5 relevant hashtags
- No clickbait
- No engagement bait (“comment YES”, etc.)
- No spam tone
- No policy-violating content

4) LENGTH
Each draft: 150–300 words.

---------------------------------------------------
OUTPUT FORMAT (STRICT)
---------------------------------------------------

=== DRAFT 1 ===
STYLE: <describe style in 3-5 words>
TITLE: <internal working title>

<LinkedIn post content>

--- END DRAFT 1 ---


=== DRAFT 2 ===
STYLE: <describe style in 3-5 words>
TITLE: <internal working title>

<LinkedIn post content>

--- END DRAFT 2 ---


=== DRAFT 3 ===
STYLE: <describe style in 3-5 words>
TITLE: <internal working title>

<LinkedIn post content>

--- END DRAFT 3 ---


FINAL_CHECK:
Persona Alignment Confidence: <0–100>
Style Distinction Confidence: <0–100>
Policy Risk Level: low | medium | high

After generating drafts, ensure each draft:
- Is ready for direct publishing without modification
- Contains no placeholders
- Contains no dynamic references to time (“today”, “this morning”) unless context requires
- Does not depend on publishing time

Generate the response now.
```

## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:

**Building a GenAI-powered Smart Document Templating System that:**
1. Converts any uploaded DOCX into a reusable template.
2. Uses GenAI (OpenAI/Gemini) to detect editable fields automatically.
3. Allows single document generation via a form.
4. Enables bulk document generation using Excel/Google Sheets.
5. Preserves formatting and outputs DOCX and/or PDF.

The core intelligence of this system is a LLM-based template field detection and schema generation, while document rendering remains deterministic and reliable.

### System Architecture
  ```
  User Upload (DOCX)
          ↓
  DOCX Parser (structure extraction)
          ↓
  GenAI Field Detection Engine
          ↓
  Field Schema + Validation Rules
          ↓
  Template Storage (DB + File Storage)
          ↓
  Single or Bulk Data Input
          ↓
  Document Rendering Engine
          ↓
  DOCX/PDF Output + ZIP + Report
  ```

### Technicals
* **Backend:**
  * Python (FastAPI / Flask)
  * DOCX parsing: python-docx
  * Templating: docxtpl
  * PDF conversion: LibreOffice / docx2pdf
  * Background jobs: Celery / RQ
  * Storage: S3-compatible (AWS/GCP/MinIO)

* **GenAI:**
    * OpenAI (GPT-4 / GPT-4.1)
    * Google (Gemini 1.5)

**LLM will be used only for:**
* Template field detection
* Field type inference
* Schema generation
* Optional conditional block detection

### Template Creation
* **Step 1: DOCX Structure Extraction**
  Parse -
    * Paragraphs
    * Tables
    * Headers/footers
    * Text runs

Convert into structured JSON:
  ```
  {
    "paragraphs": [...],
    "tables": [...],
    "headers": [...],
    "footers": [...]
  }
  ```
This is to preserve formatting so that only text content is analyzed.

* **Step 2: Field Detection Using GenAI**
  Here, the extracted text will be sent to the LLM with a structured prompt:

  The main Objective here is to detect -
    * Repeated variable patterns.
    * Candidate-specific placeholders.
    * Dynamic entities, like Name, Date, Amount, etc.
    * Content-based variable suggestions.

Example Prompt that can be used for OpenAI/Gemini:
```
You are a document schema detection system.

Given the following Word document text, identify:
1. Fields that are likely to change per document.
2. Assign a clean field name.
3. Infer type (text, date, currency, number).
4. Suggest validation rules.
5. Suggest example values.

Return ONLY JSON in this format:
{
  "fields": [
    {
      "original_text": "...",
      "field_name": "...",
      "type": "...",
      "required": true/false,
      "validation": "...",
      "example": "..."
    }
  ]
}
```

**An example to understand the output of this prompt:**
Input - 
  This offer letter is for Mr. Rahul Sharma joining as a Software Engineer with a salary of
  ₹12,00,000 per annum, effective from 10 March 2026.

LLM Output - 
  ```
  {
    "fields": [
      {"field_name": "candidate_name", "type": "text"},
      {"field_name": "role", "type": "text"},
      {"field_name": "salary", "type": "currency"},
      {"field_name": "joining_date", "type": "date"}
    ]
  }
  ```

* **Step 3: Filed Confirmation UI**
  The user only sees:
    * Suggested fields
    * Editable field names
    * Type dropdown
    * Required toggle
    * Optional conditional blocks (advanced)

  The user can then confirm, and the template is saved.

### Template Stoage

We have to store:

**1. Original DOCX**

**2. Template Metadata (DB)**
```
{
  "template_id": "offer_letter_v1",
  "fields": [
    {
      "name": "candidate_name",
      "type": "text",
      "required": true
    }
  ],
  "created_at": "...",
  "owner_id": "..."
}
```
### Generation Flow for Single Documents
1. User selects template.
2. Dynamic from auto-generated from schema.
3. Field validation happens in the backend.
4. Template rendered using docxtpl
5. Output generated:
     * DOCX
     * PDF
   File naming pattern:
```<CandidateName>_<TemplateName>_<Date>.pdf```

### Generation Flow for Documents in Bulk

Let's suppose the system provides a downloadable Excel format:

| candidate_name | role | salary | joining_date |
| -------------- | ---- | ------ | ------------ |

* **Step 1: Upload the Sheet**
    This would .xlsx for Excel upload, and secure OAuth for Google Sheets API
  
* **Step 2: Bulk Processing**
  For each row:
  * Validate fields
  * Render document
  * Log success/failure
  * Continue (no full-job crash)
  Use background queue workers.

* **Step 3: Final Output**
  The user will receive:
  * ZIP file of document
  * Generate report
  
  | Row | Status  | Error          |
  | --- | ------- | -------------- |
  | 1   | Success | —              |
  | 3   | Failed  | Missing salary |

### How to Validate and make the model Reliable

**Validation**
  * Required field check
  * Date format validation
  * Currency format normalization
  * Regex rules
**Large Batch Handling**
  * Streaming row processing
  * Worker queues
  * Memory-efficient file writing
  * Temporary storage cleanup

### To Preserve Formatting

**A Key Constraint here is:**
  Must preserve original Word formatting.

**Solution:**
* We NEVER rebuild document structure.
* We replace text placeholders only.
* Headers/footers processed separately.
* Tables maintained as-is.

### Security Considerations
* Encrypted file storage
* Temporarily signed URLs
* Sheet access via OAuth (not storing credentials)
* Auto-deletion policy for generated docs
* Role-based access control

### Final Vision

A user uploads a normal Word document once.

The system intelligently:
* Detects editable fields
* Builds a reusable schema
* Allows instant form-based generation
* Scales to thousands of documents in bulk

All this while preserving formatting and generating clean DOCX/PDF outputs with structured reports.

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

You need to put your solution here.
