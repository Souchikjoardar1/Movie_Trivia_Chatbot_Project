# MovieBot | Movie Trivia Chatbot

MovieBot is a local question-answering app built with Python, LangChain, Chroma, Ollama, and Gradio. It uses retrieval-augmented generation (RAG) to answer questions about directors, casts, release years, and awards from a bundled movie trivia PDF.

The project covers the full flow from PDF ingestion and semantic search to prompt construction and a browser chat interface. The language model and embedding model run through Ollama. A hosted API key is optional and used only for remote tracing when enabled.

## How it works

1. **Index the source.** PyPDF extracts text from [the movie trivia PDF](movies_trivia.pdf). A recursive splitter breaks it into overlapping passages, which Ollama's Granite embedding model converts into vectors for Chroma.
2. **Retrieve context.** LangChain searches for up to four relevant passages using the question and recent conversation as the query.
3. **Generate an answer.** Ollama runs Mistral by default. The prompt asks the model to use the retrieved passages and acknowledge when they lack an answer.
4. **Maintain the conversation.** Gradio displays the responses. Each browser session gets its own in-memory history, and the chain uses up to six previous exchanges. **Clear History** starts a fresh conversation.

The bundled PDF is a small reference corpus, so coverage depends on what it contains and which passages are retrieved. The interface displays the generated answer without page citations.

## Run locally

Install Python 3.12 and [Ollama](https://ollama.com/). With Ollama running, download the chat and embedding models.

```sh
ollama pull mistral
ollama pull granite-embedding:latest
```

Clone the repository and create a virtual environment.

```sh
git clone https://github.com/Souchikjoardar1/Movie_Trivia_Chatbot_Project.git
cd Movie_Trivia_Chatbot_Project
python -m venv .venv
```

Activate it with `source .venv/bin/activate` on macOS or Linux, or `.\.venv\Scripts\Activate.ps1` in Windows PowerShell. Then install the dependencies and start the app from the repository root.

```sh
python -m pip install -r requirements.txt
python movie_chatbot.py
```

Open the local URL printed by Gradio, typically `http://127.0.0.1:7860`. The app loads and indexes the PDF before the interface opens.

## Try the demo

| Prompt | What to look for |
| --- | --- |
| Who directed *The Shawshank Redemption*? | The PDF identifies Frank Darabont. |
| Who starred in it? | A follow-up that relies on the preceding question. The PDF lists Tim Robbins and Morgan Freeman. |
| What was the production budget of *The Shawshank Redemption*? | The PDF entry lacks a budget. The prompt asks the model to acknowledge missing information. |

These are checks against the source material. Wording can vary with the model and retrieved passages.

## Configuration and project files

Copy [`.env.example`](.env.example) to `.env` to override the defaults.

| Setting | Default | Purpose |
| --- | --- | --- |
| `PDF_PATH` | `movies_trivia.pdf` | Trivia PDF to index. Relative paths are resolved from the working directory. |
| `LLM_MODEL` | `mistral` | Ollama chat model. Pull the model in Ollama before switching. |
| `LANGCHAIN_TRACING_V2` | Unset | Set to `true` and configure a LangSmith API key to send traces to LangSmith. Traces can include questions, retrieved text, and model responses. |

- [`movie_chatbot.py`](movie_chatbot.py) contains PDF ingestion, retrieval, prompt construction, session handling, and the Gradio interface.
- [`movies_trivia.pdf`](movies_trivia.pdf) is the runtime knowledge base.
- [`requirements.txt`](requirements.txt) lists the Python dependencies.
- The app writes the local Chroma index under `.chroma/movie_trivia/`. To replace the source PDF, stop the app and clear that generated directory before restarting so the collection reflects the new source.

If Ollama refuses the connection, start it with `ollama serve`. If a model is missing, check `ollama list` and rerun the corresponding pull command.
