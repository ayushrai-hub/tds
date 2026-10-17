# tds - Virtual TA for the TDS course

A small FastAPI service that answers student questions about a course by searching a local knowledge base and passing the best matches to an LLM. I built it for the Tools in Data Science (TDS) course at IIT Madras Online Degree.

## How it works

1. `scrape.py` logs in to the course Discourse forum with Playwright (you sign in by hand once, the session is saved locally) and downloads topics from the TDS category within a date range set at the top of the file.
2. The forum posts and course docs are chunked into `knowledge_base.db` (SQLite, tables `discourse_chunks` and `markdown_chunks`) with an embedding stored for each chunk.
3. `app.py` embeds the question, ranks chunks by cosine similarity, pulls in neighbouring chunks for context, and asks `gpt-4o-mini` for an answer. Embeddings use `text-embedding-3-small`. Both calls go through the AI Pipe proxy (`aipipe.org`).
4. A question can include a base64 image, which is described first and then added to the query.

## API

- `POST /query` - body `{"question": "...", "image": "<optional base64>"}`; returns `{"answer": "...", "links": [{"url": "...", "text": "..."}]}`
- `GET /health` - health check

## Run locally

```bash
pip install -r requirements.txt
echo "API_KEY=your_aipipe_token" > .env
uvicorn app:app --reload
```

`API_KEY` is read from the environment and is never committed (`.env` is git-ignored). `vercel.json` is set up to deploy `app.py` on Vercel.

## Notes

- `knowledge_base.db` is about 96 MB and is checked into the repo.
- `scrape.py` only needs to be re-run to refresh the data. It expects a saved `auth.json` login state.
- CORS is open to all origins, which is fine for a course project but not for anything serious.

Licensed under MIT, see `LICENSE`.
