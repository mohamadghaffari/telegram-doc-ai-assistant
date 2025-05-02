# 🤖 Telegram AI Assistant for Your Documents (n8n + Supabase + Gemini)

This project turns a **Telegram bot** into your personal AI assistant — capable of answering questions about any document you upload. It’s powered by **Google Gemini**, **Supabase vector embeddings**, and built entirely in **n8n** with a no-code approach.

Any PDF document, upload it to the bot and chat with it like a knowledgeable assistant. No coding required.

---

## 🎯 Use Case

Two simple scenarios:

### 1. Chatbot Interface (User → Telegram → Answers)
- Users send questions to the bot.
- The bot answers using LLM (Google Gemini) and optionally fetches **weather data**.
- It can reference previously uploaded documents to give contextual responses.

### 2. PDF Upload (User → Upload → Embedding)
- Users upload a PDF.
- The flow parses the content, generates embeddings via Gemini, and stores them in a **Supabase vector table**.
- After upload, users can ask the bot questions about that document — all via chat.

---
## 🧠 Key Features

- ✅ **No-code** setup using [n8n](https://n8n.io)
- 📄 **PDF upload** and parsing
- 🧠 **Google Gemini** for embeddings + responses
- 🗂 **Supabase** as a vector database for document memory
- 🧹 **HTML post-processing**: cleans Gemini responses, removes unsupported tags
- 📤 **Smart chunking**: splits long answers into 4096-character-safe Telegram messages
- 🌦️ **Weather integration** via OpenWeatherMap (optional)
---

## 🛠 Setup Instructions

### 1. Clone or Download This Repo

This repo contains:

- `telegram-pdf-ai-assistant.json`: The full n8n flow export
- `README.md`: You’re reading it!

### 2. Import Workflow into n8n

- Open your local or hosted [n8n](https://n8n.io) instance
- Click `Import` → `Upload Workflow` → select `telegram-pdf-ai-assistant.json`

### 3. Configure Your Accounts

Create credentials for the following services:

| Service          | Use                          |
|------------------|-------------------------------|
| Telegram API     | To receive/send messages      |
| Google Gemini    | For embeddings + responses    |
| Supabase         | Store vector data from PDFs   |
| OpenWeatherMap   | (Optional) weather data       |

### 4. Set Up Supabase for Embeddings

Create a vector-enabled table (`user_knowledge_base`) and ensure it supports 768-dimension embeddings:

Follow guide here:

``` sql
-- Enable the pgvector extension to work with embedding vectors
create extension vector;

-- Create a table to store your documents
create table user_knowledge_base (
  id bigserial primary key,
  content text, -- corresponds to Document.pageContent
  metadata jsonb, -- corresponds to Document.metadata
  embedding vector(768) -- 768 works for Gemini embeddings, change if needed
);

-- Create a function to search for documents
create function match_documents (
  query_embedding vector(768),
  match_count int default null,
  filter jsonb DEFAULT '{}'
) returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (user_knowledge_base.embedding <=> query_embedding) as similarity
  from user_knowledge_base
  where metadata @> filter
  order by user_knowledge_base.embedding <=> query_embedding
  limit match_count;
end;
$$;
```
---

## 💬 How Users Interact

```
[Telegram Bot]
     ↓
[User sends PDF or asks question]
     ↓
[n8n parses & routes input]
     ↓
[Gemini + Supabase → generates reply]
     ↓
[Bot sends answer]
```

It’s that simple — seamless interaction between chat, document parsing, embeddings, and retrieval.

---

## 🖼 Workflow Preview

![Workflow Screenshot](workflow-screenshot.png)

---

## 📚 Tech Stack

- [n8n](https://n8n.io) – Automation and flow management
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Supabase](https://supabase.com/) + [LangChain Integration](https://supabase.com/docs/guides/ai/langchain?queryGroups=database-method&database-method=sql)
- [Google Gemini API](https://ai.google.dev/)
- [OpenWeatherMap API](https://openweathermap.org/api)

---

## 📄 License

MIT License — free to use, modify, and build on.

---

## 🙌 Inspired By

This project is part of an exploration of no-code AI agent development using open tools and public APIs.
