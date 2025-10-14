
# Functional Requirements Specification (FRS)

## **1. System Overview**

The AI Customer Support System enables businesses to integrate a  Arabic (Egyptian dialect, Franco-Arabic) AI chatbot into their **websites**, **mobile apps**, or **Meta platforms** (WhatsApp, Messenger, Instagram).
It supports **text** (MVP) and **voice** (planned feature) communication and provides an **admin dashboard** for businesses to manage settings, knowledge base, and analytics.

---

## **2. Core Actors**

| Actor                                     | Description                                                                                               |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **End User (Customer)**                   | A visitor or client interacting with the business chatbot via widget or Meta integration.                 |
| **Business Admin**                        | Registered business staff managing chatbot configurations, viewing analytics, and editing knowledge base. |
| **AI System**                             | Orchestrator responsible for intent detection, emotion analysis, RAG retrieval, and response generation.  |
| **Backend API (FastAPI)**                 | Middleware managing requests between user, AI models, and the database (PostgreSQL).                      |
| **Database (Neon PostgreSQL + pgvector)** | Stores all persistent entities: users, sessions, messages, knowledge base, and analytics reports.         |

---

## **3. Functional Requirements**

### **3.1 User Interaction (Text and Voice)**

* **FR1:** The system shall allow users to send **text messages** through the chat widget or Meta channels.
* **FR2:** The system shall support **voice input** (speech-to-text) and **voice output** (text-to-speech) via Whisper and TTS APIs (future).
* **FR3:** The system shall store each message / Voice in `ChatMessage` / `VoiceInteraction` with metadata like (AI model used, detected emotion, timestamp , Content or transcription_text & voice url , sender_type ).
* **FR4:** The system shall detect user **emotion** from the text or voice tone using an emotion detection layer.
* **FR5:** The chatbot shall **adapt its tone or response style** based on the detected emotion and business tone configuration.

---

### **3.2 Widget & Meta Integration**

* **FR6:** The system shall provide an **embeddable chat widget** using a unique `api_key` from `BusinessAuth`.
* **FR7:** The widget shall support **real-time messaging**, **typing indicators**, and **session persistence**.
* **FR8:** The system shall verify the `api_key` before initializing any chat session.
* **FR9:** The system shall integrate with **WhatsApp**, **Messenger**, and **Instagram APIs** (Planned feature) for businesses to connect their customer channels.
* **FR10:** The widget and integrations shall automatically switch to **offline mode** outside business hours (`operating_hour_start` / `operating_hour_end`).

---

### **3.3 Knowledge Base and AI Response**

* **FR11:** The business admin shall be able to upload or input **FAQs, text, or files** into the `KnowledgeBase` (text or file-based).
* **FR12:** The AI pipeline shall generate **embedding vectors** for each entry using AI "LangChain > AI framework" (still under review) and store them in the `embedding_vector` field.
* **FR13:** When a user message is received, the system shall perform **contextual retrieval** (RAG) to fetch top-matching knowledge chunks.
* **FR14:** The AI controller layer shall choose the most confident model output between NileChat, Gemini, or Qwen based on similarity threshold.
* **FR15:** The system shall format AI responses based on business rules from `Business Settings`consistently using a **response formatting layer**.

---

### **3.4 Business Registration and Authentication**

* **FR16:** Businesses can register an account providing `name`, `email`, and `domain_url`.
* **FR17:** A default `BusinessSetting` and active `BusinessAuth` record shall be automatically generated upon registration.
* **FR18:** Each business can manage multiple `BusinessAdmin` accounts with roles (`Owner`, `staff`).
* **FR19:** Authentication shall be done through secure login using email and password.
* **FR20:** The system shall allow owners to **generate, rotate, or deactivate API keys** without deleting the business record.

---

### **3.5 Dashboard Management**

* **FR21:** The business admin shall access an **admin dashboard** via web (React.js).
* **FR22:** The dashboard shall allow management of:

  * Knowledge Base entries (CRUD)
  * Bot tone style and response mode (auto/manual)
  * Operating hours
  * Viewing chat history and feedback
* **FR23:** The dashboard shall show analytics reports such as total chats, average response time, and user satisfaction (from `AnalyticsReport`).
* **FR24:** The dashboard shall include a **manual reply mode**, allowing admins to respond directly to ongoing chats if AI auto-reply is disabled.

---

### **3.6 Chat Session Management**

* **FR25:** When a user starts chatting, the system shall create a new `ChatSession` linked to the `EndUser` and `Business`.
* **FR26:** All messages within the same chat shall link to that session ID for history tracking.
* **FR27:** When a chat ends, the system shall mark the session as `closed` or `expired`.
* **FR28:** The user shall be prompted to submit **feedback** (rating and comment), stored in the `Feedback` table.

---

### **3.7 Analytics and Reporting**

* **FR29:** The system shall automatically aggregate chat statistics and feedback to generate `AnalyticsReport` entries.
* **FR30:** Reports shall include total chats, average response time, and average user satisfaction.
* **FR31:** Reports shall be available for download or viewing in the dashboard.
* **FR32:** Analytics shall update periodically (e.g., daily or weekly background job).

---

### **3.8 Voice Interaction (Future Expansion)**

* **FR33:** The system shall store voice recordings and transcription text in the `VoiceInteraction` table.
* **FR34:** The system shall detect emotion from voice tone and integrate results with AI response generation.
* **FR35:** The chatbot shall be able to **reply with customized voice** based on user preference or platform type (e.g., WhatsApp voice message).

---

### **3.9 AI Pipeline Orchestration**

* **FR36:** The AI Orchestrator shall manage the workflow between input message → emotion detection → RAG → model selection → formatting → output.
* **FR37:** The orchestrator shall fallback between models if the primary fails.
* **FR38:** Each step of the pipeline shall log its operation (model used, latency, confidence score).

---

### **3.10 System Administration**

* **FR39:** The system shall maintain logs of authentication attempts, API usage, and integration events.
* **FR40:** The system shall support environment-based configurations for development, staging, and production.
* **FR41:** The system shall notify admins of integration errors (e.g., API key invalid, channel disconnected).

---
