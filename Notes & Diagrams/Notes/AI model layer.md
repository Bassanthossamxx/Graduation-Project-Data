

#  1. NileChat Guide  (Base Model)

### What it is

* An **open-source / research model** fine-tuned for **Egyptian Arabic + Franco-Arabic**.
* Runs locally or on your cloud (Docker, GPU, or CPU fallback).
* Acts as your **always-available dialect-first agent**.

###  Deployment Options

* **Local Deployment:**

    * Pull from HuggingFace or repo → run on server with `transformers` or `vLLM`.
    * Package into Docker for reproducibility.
* **Cloud Deployment:**

    * Run on **Railway, Render, Koyeb, or HuggingFace Spaces**.
    * Expose as a REST API (FastAPI wrapper).

---

### Tokens / Limits

* Since you **self-host** NileChat → **no API token limit**.
* The only "tokens" are **LLM input/output tokens**:

    * e.g., if prompt = 200 tokens and answer = 100 tokens → 300 tokens processed.
    * Your limit = **GPU/CPU capacity**, not billing.
* Bottlenecks:

    * GPU RAM (e.g., 7B model needs ~16GB VRAM).
    * Latency (CPU runs slower, maybe 5–10 sec per reply).

---

## 2. Gemini API (Fallback Model)

### What it is

* Google’s **cloud LLM API** (Gemini Pro / gwmini).
* Strong for **complex reasoning, long context, open-domain answers**.
* You call it with **API key (token)**.

### Integration

* Runs only via **API calls** (no self-hosting).
* Deployed by Google, you don’t manage infra.
* You just manage your **FastAPI connector** to send requests.

### Tokens / Limits

* Here, **tokens = billing units** (like OpenAI GPT).
* Each request consumes input tokens (prompt + RAG chunks) and output tokens (model response).
* Google sets **free tier** (e.g., Gemini Pro 2.5 gives some free tokens/month) + **paid tier** (after quota).
* Example (not final numbers):

    * Free: ~60 requests/min, 32k tokens max per request.
    * Paid: ~$0.00025 per 1k input tokens, ~$0.0005 per 1k output tokens.

---

## 3. How They Work Together in Your System

* **NileChat (self-hosted)**

    * Always available, no billing limit.
    * Strong Egyptian Arabic fluency.
    * Limited reasoning power vs Gemini.
* **Gemini API (fallback)**

    * Pay-per-use, limited by quotas.
    * Used only when NileChat confidence is low or query is complex.
    * Ensures system doesn’t fail on hard cases.

---

## 4. Deployment Strategy

* **NileChat:** Deploy as Docker container (on cloud VM, or HuggingFace Spaces).
* **Gemini API:** No deployment → just API integration.
* **Controller Layer:** In FastAPI, decide which model to call.

---

So in simple terms:

* **NileChat = free unlimited, but resource-heavy** (runs on your machine/cloud).
* **Gemini = scalable but metered**, limited by Google’s quotas + costs.
---
# 2. Gemini API Integration Guide (fallback model)

##  What Gemini Brings to the System

* Acts as the **fallback model**.
* Stronger in **reasoning, complex queries, and open-domain knowledge**.
* Cloud-hosted by Google → you don’t run it locally.
* Accessed through **API calls** (using an API key).

---

##  How Gemini Integration Works (Step by Step)

1. **User query arrives**

    * User sends message via frontend → FastAPI middleware receives it.

2. **Controller checks NileChat’s response**

    * If NileChat is confident → return answer.
    * If NileChat is *not confident* → forward query to Gemini.

3. **RAG injection (optional but powerful)**

    * Before sending to Gemini, system retrieves **knowledge base chunks** (FAQs, docs).
    * These are inserted into the Gemini prompt as **context**, so answers are business-specific, not just generic.

4. **Gemini API call**

    * Query + context are sent to **Google’s Gemini endpoint**.
    * Gemini processes input tokens and generates a response.

5. **Response post-processing**

    * Apply emotion-aware tone (angry, happy, polite, etc.).
    * Format the output to match the same style as NileChat (so users don’t notice switching).

6. **Send response back**

    * FastAPI returns formatted answer → frontend shows to the user.

---

##  Flow Diagram (Conceptual)

```
User → FastAPI Middleware
     ├─ Emotion Detection
     ├─ RAG Retrieval (KB context)
     └─ Controller Layer
          ├─ NileChat (Base Model) → confident → Reply
          └─ Gemini API (Fallback)
                 • Inject KB chunks into prompt
                 • Call Gemini cloud model
                 • Get response
     → Tone & Formatting Layer
     → Return to User
```

---

##  Key Characteristics of Gemini Integration

* **Cloud-managed** → no deployment, you only integrate via API.
* **Token-based usage** → cost + quota based on input/output tokens.
* **Strong fallback** → ensures chatbot can always handle complex or unknown queries.
* **Consistency required** → unify tone/style so users don’t feel “two personalities.”

---

##  Why Gemini is Secondary (Not Base)

* **Cost:** Calling Gemini for *every* query would be expensive.
* **Language Gap:** NileChat is more fluent in Egyptian Arabic slang/Franco.
* **Performance:** Gemini adds latency for simple FAQs that NileChat could handle instantly.

So → Gemini is the **safety net** that guarantees reliability when NileChat isn’t enough.

---
# 3. NileChat Deployment Guide

##  1. Running NileChat Locally

NileChat is just a HuggingFace-compatible model (like GPT-J, LLaMA, etc.).
You can:

* Download it from HuggingFace Hub.
* Run inference with `transformers` or `vLLM`.
* Expose it with **FastAPI** so your frontend/backend can call it.

Example FastAPI wrapper (simplified):

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

app = FastAPI()

class ChatRequest(BaseModel):
    message: str

# Load NileChat model (from HuggingFace Hub)
model_name = "NileChat/model"   # Replace with actual repo name
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name).to("cuda")

@app.post("/nilechat")
def chat(req: ChatRequest):
    inputs = tokenizer(req.message, return_tensors="pt").to("cuda")
    outputs = model.generate(**inputs, max_new_tokens=200)
    reply = tokenizer.decode(outputs[0], skip_special_tokens=True)
    return {"reply": reply}
```

* Save this as `app.py`.
* Run with `uvicorn app:app --reload`.
* Now you have a local REST API endpoint at `http://localhost:8000/nilechat`.

---

## 2. Deploying on HuggingFace Spaces

HuggingFace Spaces is like “free hosting” for ML demos.

* You push your code + model → it runs automatically in a container.
* Supports **Gradio, Streamlit, or FastAPI** backends.
* Free tier has limited GPU/CPU, but you can upgrade with **GPU Spaces** if needed.

Steps:

1. Create a new Space → choose **Docker / Gradio / FastAPI template**.
2. Push your model + `requirements.txt` + `app.py` (like above).
3. HuggingFace builds it → gives you a URL like:

   ```
   https://huggingface.co/spaces/<team>/<nilechat-service>
   ```
4. That becomes your NileChat endpoint, callable from your backend.

Example `requirements.txt`:

```
transformers
torch
fastapi
uvicorn
```

---

## 3. Deploying as Docker Container (Recommended for Control)

If you don’t want HF Spaces limits, you can deploy on any cloud (Railway, Render, AWS, Azure, etc.):

1. **Dockerfile Example**:

   ```dockerfile
   FROM python:3.10-slim

   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt

   COPY app.py .

   CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
   ```

2. **Build & Run**:

   ```bash
   docker build -t nilechat-api .
   docker run -p 8000:8000 nilechat-api
   ```

3. Now you have a NileChat API container → deploy it on:

    * **Railway.app** (student-friendly free tier).
    * **Render.com** (free plan, scalable).
    * **HuggingFace Spaces (Docker mode)** if you want hybrid.

---

## 4. Connection with Your System

Once deployed (HF Space URL or Docker service URL), your **FastAPI middleware** just calls:

```python
import httpx

async def call_nilechat(query: str) -> str:
    async with httpx.AsyncClient() as client:
        resp = await client.post("https://nilechat.space/api/chat", json={"message": query})
        return resp.json()["reply"]
```

So it acts exactly like Gemini API, but **self-hosted** and unlimited.

---

##  5. Key Trade-offs

* **HF Spaces (Quick, Free)** → easy MVP demo, but limited resources, may sleep when idle.
* **Docker + Cloud VM (Flexible)** → full control, can scale, but you pay for GPU.
* **Local GPU (Free but limited)** → only for dev/test, not production.
---
# 4. Controller Layer Guide
##  Overview

When using **two models** (NileChat + Gemini), you need a **controller layer** in your FastAPI middleware to decide **which model to call** based on the user query.
* **NileChat (base)** → fast, cheap, dialect fluent.
* **Gemini (fallback)** → stronger reasoning, open-domain, but costly/limited.

Without a controller layer, you’d either:

* Always hit NileChat (but risk nonsense answers for complex queries).
* Always hit Gemini (but waste money and lose dialect fluency).

---

##  Why Controller Layer is Needed

1. **Efficiency**

    * Saves cost (Gemini API isn’t free for unlimited calls).
    * Keeps latency low (simple FAQs answered instantly by NileChat).

2. **Quality Control**

    * Prevents wrong or low-quality answers from NileChat.
    * Ensures fallback to Gemini when the query is outside NileChat’s domain.

3. **Unified Experience**

    * Controller enforces **response formatting**, so the user doesn’t notice a “switch.”

---

##  How to Decide (Thresholding Methods)

Since NileChat won’t give a built-in “confidence score,” you need to approximate confidence. Common strategies:

### 1. **Keyword/Intent Match**

* If user’s query matches business KB / FAQ directly → NileChat is trusted.
* If query has **unknown intent** → fallback.

Example:

```python
if similarity(query, kb_top_chunk) < 0.6:
    use_gemini()
else:
    use_nilechat()
```

---

### 2. **Semantic Similarity (Embedding-based)**

* Generate embedding for **user query** and **retrieved KB chunks**.
* Use cosine similarity to measure closeness.
* If similarity < threshold (e.g., 0.7), NileChat is not reliable → fallback.

---

### 3. **Answer Quality Heuristics**

* After NileChat responds, check:

    * Is it empty or very short (e.g., <10 words)?
    * Does it contain fallback phrases like “I don’t know”?
* If yes → send to Gemini.

---

### 4. **Hybrid Scoring (Best Practice)**

Combine multiple signals:

```
Final Confidence = 0.5 * (KB similarity) + 0.3 * (answer length score) + 0.2 * (intent match)
```

* If `Final Confidence >= 0.7` → accept NileChat.
* Else → fallback to Gemini.

---

##  Visual

```
User → FastAPI Middleware
   └─ Controller Layer
        ├─ Check KB similarity + NileChat response
        ├─ If Conf ≥ Threshold → NileChat reply
        └─ Else → Gemini fallback
```

---

⚡ In short:

* The **controller** = router + judge.
* The **threshold** = “minimum trust” in NileChat’s answer (based on similarity, length, heuristics).
* Keeps system **cheap, fluent, but reliable**.

---
# 5. Prompting / formatting layer Guide
##  Why You Need a Formatting Layer

* NileChat = strong in dialect, but maybe less structured.
* Gemini = structured and smart, but maybe too formal (MSA or English).
* Without formatting → users feel like they’re chatting with **two different personalities**.

So the **formatting layer** enforces:

* Consistent **tone** (friendly, polite, brand-specific).
* Consistent **style** (short sentences, emojis, or bullet points).
* Consistent **language variant** (Egyptian Arabic dialect + Franco where needed).

---

##  How It Works

1. **Model generates answer (NileChat or Gemini).**

    * E.g., Gemini says: *“Your order will be delivered within 2–3 business days.”*
    * NileChat says: *“التوصيل بياخد من يومين لتلاتة جوه القاهرة.”*

2. **Formatting Layer intercepts answer.**

    * Wraps it in a second prompt with clear instructions:

      ```
      Instruction: 
      Reformat the answer to match the company’s chatbot style:
      - Always reply in Egyptian Arabic dialect.
      - Keep sentences short, friendly, with emojis when appropriate.
      - Add a closing phrase: "هل تحب تعرف أكتر؟"
 
      Original Answer: "Your order will be delivered within 2–3 business days."
      Reformatted Answer:
      ```
    * Sends this to a **reformatter model** (could even be NileChat again).

3. **Reformatter model outputs the final consistent reply.**

    * *“طلبك هيوصلك خلال يومين لتلاتة إن شاء الله 🚚. هل تحب تعرف أكتر؟”*

---

##  Options for the Formatting Layer

* **Use NileChat as reformatter (always).**

    * Advantage: forces everything into Egyptian Arabic dialect.
    * Even Gemini answers get “Egyptianized.”
* **Use a lightweight instruction-tuned model** just for formatting.

    * Keeps separation clean, but adds complexity.
* **Inline formatting rules** (string replacements, templates).

    * Faster, cheaper, but limited flexibility.

---

##  Flow With Formatting Layer

```
User → FastAPI Middleware
   ├─ Emotion Detection
   ├─ RAG Retrieval
   ├─ Controller Layer
   │     ├─ NileChat (Base) 
   │     └─ Gemini (Fallback)
   ├─ Formatting Layer
   │     ├─ Send raw answer + style instructions
   │     └─ Output unified response (Egyptian Arabic tone/style)
   → Return to User
```

---

##  Example Rule

* If **answer from Gemini** → pass it to NileChat (as reformatter).
* If **answer from NileChat** → still pass it again through NileChat (or a formatting prompt) to unify tone.
* This way, *all final answers come out of one “voice.”*

---

⚡ In short:

* Formatting layer = the **“editor”** of the chatbot.
* Ensures **all replies sound Egyptian, friendly, and brand-consistent**.
* Technically, it’s just a **prompting step after the main answer**, possibly reusing NileChat.

---