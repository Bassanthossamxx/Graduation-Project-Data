# Kalamna - AI Customer Support Egyptian Service ERD V2
## **Overview**
The AI Customer Support Egyptian Service enables Business to integrate a smart AI chatbot into their websites or apps through an embeddable web widget.
 It supports text (and future voice) communication in Egyptian Arabic, allowing businesses to automate responses based on their knowledge base, while admins can manage settings, analyze reports, and respond manually when needed.

---

## **1. Business**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| name | String | Registered business name |
| email | String | Contact or registration email |
| description | Text | Overview of the company |
| industry | String | Industry category |
| domain_url | String | Website domain where the widget is embedded |
| created_at | Timestamp | Record creation timestamp |
| updated_at | Timestamp | Last update timestamp |
**Notes:**

- Represents each client business using the platform.
- Acts as a parent entity for admins, settings, authentication keys, users, and analytics.
---

## **2. BusinessSetting**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| bot_name | String | Optional; defaults to system name but can be customized |
| tone_style | String | Predefined choice (e.g., formal, friendly, neutral) |
| auto_reply_enabled | Boolean | Determines if AI auto-replies or waits for human admin input |
| operating_hour_start | Time | Business operating start time |
| operating_hour_end | Time | Business operating end time |
| business_id | FK → Business | One-to-one link to owning company |
| created_at | Timestamp | Record creation timestamp |
**Notes:**

- Controls chatbot personality, response mode, and business availability.
- Separated to isolate configuration logic from business metadata.
---

## **3. BusinessAuth**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Business | Linked to one business |
| api_key | String (hashed) | Unique key used to authenticate API or widget requests |
| is_active | Boolean | Determines whether the key is valid for use |
| created_at | Timestamp | Record creation timestamp |
**Notes:**

- Provides secure integration for external APIs or widgets.
- Rotatable and can be deactivated without affecting the main business record.
---

## **4. BusinessAdmin**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| name | String | Admin’s full name |
| email | String | Unique login email |
| password | String | Encrypted password |
| role | String | Role indicator choices (“owner” or “staff”) |
| business_id | FK → Business | Associated to business |
| created_at | Timestamp | Account creation timestamp |
**Notes:**

- Each business can have multiple admins.
- Owners manage authentication keys, configurations, and analytics access.
---

## **5. EndUser**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Business | Company this user interacts with |
| external_id | String | Optional (e.g., WhatsApp, Messenger ID) |
| name | String (optional) | End-user display name |
| created_at | Timestamp | First interaction timestamp |
| updated_at | Timestamp | Last interaction timestamp |
**Notes:**

- Represents real customers interacting with the chatbot.
- Data remains minimal for privacy; no login is required.
---

## **6. ChatSession**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Business | Chat belongs to a specific company |
| end_user_id | FK → EndUser | Linked user |
| session_token | String | Unique identifier for the chat session |
| status | String | Choices > “active”, “closed”, or “expired” |
| started_at | Timestamp | Start time of chat |
| ended_at | Timestamp | End time of chat |
| feedback_id | FK → Feedback | Optional feedback link |
**Notes:**

- Tracks each conversation between a business and an end user.
- Enables reloading previous sessions or conversation histories.
---

## **7. ChatMessage**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| session_id | FK → ChatSession | Belongs to a session |
| sender_type | String | “user”, “bot”, or “admin” |
| content | Text | Message body |
| ai_model_used | String | AI model used for the response (e.g., Gemini, NileChat) |
| emotion_detected | String | auto-tagged emotion (from emotion detection layer) |
| time | Timestamp | Message timestamp |
**Notes:**

- Core unit of conversation storage.
- Enhances analytics and AI fine-tuning accuracy.
---

## **8. Feedback**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| enduser_id | FK → Enduser | Associated to enduser |
| session_id | FK → ChatSession | Associated session |
| rating | Integer | Numeric rating (1–5) |
| comment | Text | User feedback text |
| submitted_at | Timestamp | Feedback submission timestamp |
**Notes:**

- Tracks satisfaction and sentiment at the end of each chat.
- Used for business performance analytics.
---

## **9. KnowledgeBase**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Business | Linked business |
| base_type | String | “text” or “file” |
| content_json | JSON | Structured content for text entries |
| file_url | String | Path to uploaded files (null if text type) |
| file_type | String | MIME type for uploaded files |
| embedding_vector | Vector | Generated vector representation for RAG |
| created_at | Timestamp | Creation timestamp |
**Notes:**

- Represents structured business knowledge used for retrieval-augmented generation.
- Embeddings generated automatically through LangChain and stored in the database.
- will normalize it to be have other table for vector in small size
---

## **10. AnalyticsReport**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Business | Business that owns this report |
| total_chats | Integer | Total sessions during report period |
| avg_response_time | Float | Mean bot response latency |
| user_satisfaction | Float | Average feedback score |
| report_period_start | Date | Start of reporting window |
| report_period_end | Date | End of reporting window |
| generated_at | Timestamp | Timestamp when report was generated |
**Notes:**

- Automatically generated by background analytics workers.
- Used to evaluate customer satisfaction and agent performance.
---

## **11. VoiceInteraction (Future)**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| session_id | FK → ChatSession | Linked chat session |
| audio_file_url | String | Path to audio input |
| transcription_text | Text | Speech-to-text conversion |
| detected_emotion | String | Optional detected emotional tone |
| created_at | Timestamp | Record timestamp |
**Notes:**

- Extends chat capabilities to voice input/output.
- Will integrate with STT/TTS APIs for hybrid voice-text interaction.
---

## **12. Relationships Summary**
| Relationship | Type | Description |
| ----- | ----- | ----- |
| Business → BusinessAdmin | 1 → Many | Multiple admins can manage one business. |
| Business → BusinessSetting | 1 → 1 | Each business has a single configuration profile. |
| Business → BusinessAuth | 1 → Many | One business can have multiple API keys (rotatable). |
| Business → EndUser | 1 → Many | Multiple users can chat with the same business bot. |
| Business → KnowledgeBase | 1 → Many | Each business maintains several data sources. |
| Business → AnalyticsReport | 1 → Many | Reports generated per business. |
| EndUser → ChatSession | 1 → Many | Each user can have multiple chat sessions. |
| ChatSession → ChatMessage | 1 → Many | Each chat session contains multiple messages. |
| ChatSession → Feedback | 1 → 1 | Optional feedback per session. |
| ChatSession → VoiceInteraction | 1 → Many | Multiple voice messages can belong to a chat. |
---

## **13.  Data & AI Flow Summary**
1. **Registration Phase**
    - Admin registers → new `Business` , `BusinessSetting` , and `BusinessAuth`  created.
    - Default settings applied automatically.

2. **Widget Integration**
    - Website embeds the widget using the unique API key from `BusinessAuth` .
    - The API key is verified for active status before initiating a session.

3. **Chat Initialization**
    - End user opens chat → new `EndUser`  + `ChatSession`  created.
    - System checks business `operating_hour_start`  and `auto_reply_enabled`  from `BusinessSetting` .

4. **Conversation Flow**
    - Messages stored in `ChatMessage` .
    - AI orchestrator retrieves contextual data from `KnowledgeBase`  via embeddings (RAG pipeline).
    - Model response stored with metadata (`ai_model_used` , `emotion_detected` ).

5. **Feedback Capture**
    - Once chat ends, user submits feedback → saved to `Feedback` .

6. **Analytics & Monitoring**
    - Background job aggregates sessions, feedback, and timing metrics → stores results in `AnalyticsReport` .

7. **Future Voice Support**
    - Voice input saved in `VoiceInteraction` , transcribed and analyzed for emotion and intent.

---

## **14. Summary**
This version introduces **clear modular separation** and **secure authentication control**, resulting in:

- Stronger **security model** via per-business API keys.
- Independent **configuration management** (tone, hours, auto-reply).
- Scalable **AI data pipeline** ready for real-world deployment.
- Future-ready **voice and emotion** integration support.
---



