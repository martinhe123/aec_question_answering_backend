# Prompt Log

This file summarizes the AI tools and key prompts that shaped the AEC
question-answering project. The entries below are concise reconstructions from
the development conversation and project documentation; they are not presented
as complete verbatim transcripts.

No API keys, credentials, or private configuration values are included.

## AI Tools and Models

### Development tool

- **OpenAI Codex:** Used as a coding assistant to review requirements, inspect
  and modify project files, test the backend, audit the implementation, improve
  documentation, and prepare the repositories for submission.

The exact underlying Codex model was not recorded in the project files, so it is
not claimed here.

### Runtime model

- **OpenAI `gpt-4o-mini`:** Called by the FastAPI backend to classify and answer
  user questions. The backend requests a structured result containing an AEC
  category and an answer.

## Key Prompts and Requests

### 1. Define the initial backend and frontend

**Prompt summary:**

> Build a small chatbot that answers architecture, engineering, and
> construction questions. Use a Python FastAPI backend on Render and a static
> HTML/CSS/JavaScript frontend on GitHub Pages. The frontend should send a
> message to `POST /chat` and display the returned answer.

**Resulting decisions:**

- FastAPI handles the backend request and response flow.
- The frontend and backend communicate through JSON over HTTPS.
- CORS allows the deployed GitHub Pages origin and approved local development
  origins.

### 2. Keep the OpenAI key on the backend

**Prompt summary:**

> Keep the OpenAI API key out of frontend JavaScript. Use a Render environment
> variable in production and allow a local secret file only as a development
> fallback.

**Resulting decisions:**

- Production reads `OPENAI_API_KEY` from the environment.
- Local development can fall back to an ignored `secrets.txt` file.
- The frontend never receives or uses the OpenAI key.

### 3. Validate requests and handle failures clearly

**Prompt summary:**

> Reject empty input and messages longer than 200 words, handle OpenAI API
> failures, return appropriate HTTP status codes, and show understandable error
> messages in the frontend.

**Resulting decisions:**

- The backend validates empty and oversized messages.
- The frontend performs the same basic validation before sending.
- The API returns distinct validation, missing-conversation, rate-limit, and
  upstream-service errors.
- The frontend removes its Thinking indicator and restores its controls after
  either success or failure.

### 4. Add structured AEC classification

**Prompt summary:**

> Use one structured OpenAI response to both classify and answer each question.
> Support categories for codes, safety, architecture, structures, energy,
> building systems, construction, materials, sustainability, and general AEC.
> Also identify clearly off-topic questions and ambiguous AEC-related terms
> that need clarification.

**Resulting decisions:**

- `ModelResult` contains an `AECCategory` and answer.
- The runtime model selects one category based on the user's primary intent.
- `not_aec` produces a short fixed response.
- `needs_clarification` produces a fixed request for more context.
- The prompt includes named-entity examples such as architecture firms and
  ambiguous company or person names.

### 5. Add safe, curated related resources

**Prompt summary:**

> For valid AEC answers, display up to three useful links. Do not let the model
> invent URLs; keep all resource URLs in a backend-owned mapping. Use HTTPS links
> and open them safely in a new tab.

**Resulting decisions:**

- Every valid AEC category maps to exactly three curated resources.
- Off-topic and clarification responses contain no resources.
- The frontend filters for HTTPS URLs and uses `noopener noreferrer` on
  new-tab links.

### 6. Add short-term conversational memory

**Prompt summary:**

> Add short-term memory using the 10 most recent messages. Generate a unique
> conversation ID on the backend, return it to the frontend, and send it with
> follow-up questions. Preserve user and assistant roles and keep conversations
> isolated from one another.

**Resulting decisions:**

- The first successful response returns a backend-generated UUID.
- The frontend reuses that UUID during the current page session.
- The backend supplies up to 10 previous messages to the model, followed by the
  current user message.
- Memory is temporary and process-local rather than database-backed.
- The New conversation button clears the displayed conversation and stops using
  the previous ID.

### 7. Keep the implementation small and maintainable

**Prompt summary:**

> Keep the project minimal. Do not add authentication, a database, LangChain,
> agents, or unrelated abstractions. Separate the backend into understandable
> modules when doing so improves maintainability.

**Resulting decisions:**

- Routes and deployment configuration remain in `main.py`.
- OpenAI request assembly and response creation live in `chat_service.py`.
- Schemas, prompts, resources, and conversation storage have focused modules.
- Conversation history uses a small in-memory store.

### 8. Test the API contract and memory behavior

**Prompt summary:**

> Add repeatable tests for AEC and off-topic responses, input validation,
> resource mappings, clarification behavior, conversation ordering, the
> 10-message limit, conversation isolation, malformed IDs, and failed model
> calls.

**Resulting decisions:**

- Model responses are mocked in automated tests.
- The suite verifies the backend contract without making paid OpenAI calls.
- At the time this prompt log was created, all 11 backend tests passed.

### 9. Audit the implementation and overview

**Prompt summary:**

> Audit the project folder, paying particular attention to the overview. Check
> whether the documentation accurately describes the implementation and identify
> important limitations.

**Resulting decisions:**

- The Scope 1 overview was corrected to match the actual architecture and
  endpoints.
- The documentation now describes temporary memory, concurrency limitations,
  request-size limitations, missing timeouts, and current automated test
  coverage.
- A proposed image-diagram Scope 2 was deferred so the submission could focus on
  the completed chatbot.

### 10. Prepare the assignment submission

**Prompt summary:**

> Compare the completed Scope 1 chatbot with the assignment requirements. Create
> a turn-in checklist, separate the frontend and backend into different GitHub
> repositories, and add the required backend README and prompt log.

**Resulting decisions:**

- The existing repository remains the GitHub Pages frontend repository.
- A separate backend repository contains the FastAPI application at its root.
- The backend README documents endpoints, frontend communication, local setup,
  Render deployment, testing, and secret handling.
- The final checklist also covers the portfolio link, demonstration video, and
  Google form submission.

## Runtime System Prompt

The full runtime system prompt is maintained in `prompts.py`. Its main
instructions are to:

- Classify the user's primary intent into exactly one supported category.
- Treat legal requirements as codes and immediate worker hazards as safety.
- Avoid assuming an unfamiliar proper name is unrelated to AEC.
- Return `not_aec` for clearly unrelated questions.
- Return `needs_clarification` for plausibly AEC-related but ambiguous terms.
- Answer valid AEC questions professionally and clearly.
- Avoid adding website links to the model-generated answer because the backend
  supplies curated resources separately.

## Human Review

AI assistance was used throughout development, but the project owner remained
responsible for selecting the project scope, reviewing changes, testing the
deployed application, protecting credentials, and preparing the final
submission.
