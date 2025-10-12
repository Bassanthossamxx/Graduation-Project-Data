# Kalamna - AI Customer Support Egyptian Service ERD V1
## **Overview**
The AI Customer Support Egyptian Service enables Business to integrate a smart AI chatbot into their websites or apps through an embeddable web widget.
 It supports text (and future voice) communication in Egyptian Arabic, allowing businesses to automate responses based on their knowledge base, while admins can manage settings, analyze reports, and respond manually when needed.

---

## **1. **Business
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| name | String | Business name |
| email | String | Contact email |
| description | Text | Business overview |
| industry | String | Business industry type |
| domain_url | String | Website domain where widget is integrated |
| bot_name | String | Custom chatbot name |
| created_at | Timestamp | Record creation date |
| updated_at | Timestamp | Last update timestamp |
**Notes:**

- Each Business represents one client using the service.
- The Business owns its chatbot, knowledge base, and analytics.
---

## **2. Business Admin**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| name | String | Admin’s full name |
| email | String | Login email (unique) |
| password | String | Encrypted password |
| role | String | “owner” or “staff” |
| business_id | FK → Company | Belongs to one company |
| created_at | Timestamp | Account creation date |
**Notes:**

- Admins manage chatbot setup, upload data, and view reports.
- A Business can have multiple admins.
---

## **3. End User**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Company | The company this user interacts with |
| external_id | String | For WhatsApp or Messenger integration (optional) |
| created_at | Timestamp | When user first interacted |
| updated_at | Timestamp | Last interaction timestamp |
**Notes:**

- Represents a real customer chatting with the Business bot.
- Data may be minimal since users don’t need to register. SO optional to add name , phone , email  fields
---

## **4. Chat Session**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Company | Chat belongs to one company |
| end_user_id | FK → EndUser | Chat belongs to one user |
| session_token | String | Unique session identifier |
| status | String | “active”, “closed”, or “expired” |
| started_at | Timestamp | When chat started |
| ended_at | Timestamp | When chat ended |
| feedback_id | FK → Feedback | Optional feedback link |
**Notes:**

- Represents a single conversation between user and bot.
- One user can have multiple sessions over time.
---

## **5. Chat Message**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| session_id | FK → ChatSession | Belongs to a chat session |
| sender_type | String | “user”, “bot”, or “admin” |
| content | Text | Message content |
| ai_model_used | String | Model name used for reply (e.g., NileChat, Gemini) "auto added with AI layer" |
| emotion_detected | String | <p>Optional emotion tag </p><p>"auto added with AI layer"</p> |
| timestamp | Timestamp | Message timestamp |
**Notes:**

- Every message exchanged is stored here.
- Helps with analytics, AI improvement, and conversation replay.
---

## **6. Feedback**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| session_id | FK → ChatSession | Related chat session |
| rating | Integer | Rating from 1 to 5 |
| comment | Text | User feedback message |
| submitted_at | Timestamp | Feedback submission time |
**Notes:**

- Captures end-user satisfaction after each chat.
- Used in analytics reports.
---

## **7. Knowledge Base**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Company | Belongs to one company |
| base_type | String | “text” or “file” |
| content_json | JSON  | Structured data (FAQs, policies, etc.) >> used for text only |
| file_url | String | Path to uploaded PDF, doc, or image  >> null if type is text  |
| file_type | String | File MIME type >> only when needed  |
| embedding_vector | Vector | Auto-generated embedding for RAG |
| created_at | Timestamp | Upload or creation date |
**Notes:**

- Knowledge is used by the AI to generate accurate answers.
- Embeddings are created automatically using LangChain and stored in the database.
---

## **8. Analytics Report**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| business_id | FK → Company | The company owning the report |
| total_chats | Integer | Number of sessions in the period |
| avg_response_time | Float | Average bot response time |
| user_satisfaction | Float | Calculated from feedback ratings |
| report_period_start | Date | Start date of report range |
| report_period_end | Date | End date of report range |
| generated_at | Timestamp | When the report was generated |
**Notes:**

- Used by admins for performance tracking and decision-making.
- Generated automatically by background analytics jobs. 
- example for output : In October, Company {{company_id}} handled {{total_chats}} chats, responded in ~1.8s on average, and received an average satisfaction rating of 4.3/5.
---

## **9. Voice Interaction (Future Feature)**
| Field | Type | Description |
| ----- | ----- | ----- |
| id | UUID | Primary key |
| session_id | FK → ChatSession | Belongs to one chat session |
| audio_file_url | String | Stored path for the voice file |
| transcription_text | Text | Speech-to-text output |
| detected_emotion | String | Optional emotion tag |
| created_at | Timestamp | Timestamp of voice input |
**Notes:**

- Supports future voice-based communication (STT & TTS).
- Linked directly to existing chat sessions.
---

## ** Relationships Summary**
| Relationship | Type | Description |
| ----- | ----- | ----- |
| Business → Business Admin | 1 → Many | One Business can have multiple admins. |
| Business → End User | 1 → Many | Many users can interact with one Business bot. |
| Business → Knowledge Base | 1 → Many | Each Business has multiple data sources. |
| Business → Analytics Report | 1 → Many | Reports are generated per company. |
| End User → Chat Session | 1 → Many | A user can have multiple chat sessions. |
| Chat Session → Chat Message | 1 → Many | A chat session holds many messages. |
| Chat Session → Feedback | 1 → 1 | Each session can have one feedback entry. |
| Chat Session → Voice Interaction | 1 → Many | Each chat may have multiple voice exchanges. |
---

## ** Data & AI Flow Summary**
1. **Registration:** Business admin registers → new Business and admin record created.
2. **Integration:** Business embeds chat widget on their site (auto-authenticated by Business token).
3. **Chat Start:** End user opens widget → new chat session + user created.
4. **Conversation:**
    - User sends messages → stored in `ChatMessage` .
    - AI retrieves company-specific data from `KnowledgeBase` .
    - Response returned and logged (with model name).

5. **Feedback:** After session ends, user can rate and comment → stored in `Feedback` .
6. **Analytics:** System aggregates chat data into `AnalyticsReport` .
7. **Future Support:** Voice data captured in `VoiceInteraction`  for AI processing.
---


