# AEC Question-Answering Backend

This repository contains the FastAPI backend for an architecture, engineering,
and construction (AEC) question-answering chatbot. The backend validates user
input, sends questions and recent conversation context to the OpenAI API,
returns a structured answer and category, and attaches curated AEC resource
links.

The backend is designed to run on Render and communicate with a separate static
frontend hosted on GitHub Pages.

## Endpoints

### `GET /`

Basic health check.

Response:

```json
{
  "status": "ok"
}
```

### `POST /chat`

Accepts an AEC question and an optional conversation ID.

First request:

```json
{
  "message": "What is mass timber?"
}
```

Follow-up request:

```json
{
  "conversation_id": "generated-conversation-id",
  "message": "What about its fire resistance?"
}
```

Fields:

- `message`: required string containing the user's question; limited to 200
  words.
- `conversation_id`: optional UUID returned by an earlier successful request.
  Supplying it continues that conversation.

Example response:

```json
{
  "conversation_id": "generated-conversation-id",
  "response": "Mass timber products are engineered wood systems...",
  "category": "materials",
  "resources": [
    {
      "title": "NIST Buildings and Construction",
      "url": "https://www.nist.gov/buildings-and-construction"
    },
    {
      "title": "USDA Forest Products Laboratory",
      "url": "https://www.fpl.fs.usda.gov/"
    },
    {
      "title": "American Concrete Institute",
      "url": "https://www.concrete.org/"
    }
  ]
}
```

The response contains:

- `conversation_id`: backend-generated UUID used for follow-up questions.
- `response`: answer or fixed off-topic/clarification message.
- `category`: the structured AEC classification selected for the question.
- `resources`: up to three curated HTTPS links for valid AEC categories. This
  list is empty for off-topic questions and clarification requests.

The endpoint can return:

- `400` for empty messages or messages longer than 200 words.
- `404` for an unknown conversation ID.
- `422` for malformed request data, including an invalid UUID.
- `429` when the in-memory request limit is exceeded.
- `502` when the OpenAI request fails or returns an unusable result.

## How the Frontend Uses the Backend

The separate GitHub Pages frontend calls `POST /chat` when the user submits the
chat form. It sends the user's message and, after the first successful request,
the active `conversation_id`.

The frontend displays the returned answer, category, and related resource
links. It keeps the conversation ID in browser memory for follow-up questions.
The New conversation button clears the displayed conversation and stops sending
the previous ID. The frontend also displays validation and server errors to the
user.

The deployed frontend is configured to call:

```text
https://aec-question-answering.onrender.com/chat
```

## Conversational Memory

The backend keeps up to 10 previous user and assistant messages for each
conversation. This memory is temporary and process-local:

- It is lost when the Render process restarts.
- It is not shared between multiple backend processes.
- Conversation records do not currently expire before a process restart.
- The browser does not persist the conversation ID across page reloads.

No database or long-term user profile is used.

## Local Setup

Requirements:

- Python 3.10 or newer
- An OpenAI API key

Create and activate a virtual environment, then install the dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Set the OpenAI API key for the current PowerShell session:

```powershell
$env:OPENAI_API_KEY = "your-openai-api-key"
```

Run the backend from the repository root:

```powershell
uvicorn main:app --reload
```

The local service will normally be available at `http://127.0.0.1:8000`.
FastAPI's interactive API documentation will be available at
`http://127.0.0.1:8000/docs`.

For local development only, the application also supports a `secrets.txt` file
in the repository root when `OPENAI_API_KEY` is not set. `secrets.txt` is
excluded by `.gitignore` and must never be committed.

## Running the Tests

Set `OPENAI_API_KEY` to any nonempty test value before importing the application;
the model calls are mocked by the tests.

```powershell
$env:OPENAI_API_KEY = "test-key"
python -m unittest discover -s . -p "test*.py" -v
```

## Render Deployment

Create a Render web service connected to this repository and use:

- Build command: `pip install -r requirements.txt`
- Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
- Environment variable: `OPENAI_API_KEY`

If the repository files are at the repository root, leave Render's Root
Directory setting blank.

After deployment, verify the health endpoint and test a real request through
the frontend.

## Secrets and Security

The OpenAI API key is used only by the backend. It is not included in frontend
JavaScript or returned to the browser. In production, the key is stored as a
Render environment variable.

The `/chat` endpoint is public and does not use user authentication. It has a
basic in-memory rate limit of 10 requests per minute per observed client IP.
CORS permits the deployed GitHub Pages origin and the approved local development
origins.

Never commit API keys, credentials, `.env`, `secrets.txt`, virtual environments,
or generated cache files.

## Main Files

- `main.py`: FastAPI routes, validation, CORS, rate limiting, and error handling.
- `chat_service.py`: conversation assembly, structured OpenAI request, and
  response creation.
- `conversation_store.py`: temporary short-term conversation storage.
- `schemas.py`: request, response, category, and resource models.
- `prompts.py`: system prompt and fixed user-facing responses.
- `resources.py`: application-owned curated resource mappings.
- `test_main.py`: backend API and conversation-memory tests.
