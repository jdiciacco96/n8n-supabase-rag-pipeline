# Andrews Construction AI Knowledge Assistant

An n8n-based RAG chatbot for asking natural-language questions about the Andrews Construction knowledge base.

## Architecture

The project uses two connected n8n workflows: one to index source material and one to answer questions from that indexed content.

```text
DOCUMENT INGESTION

Google Drive file
       |
       v
Default Data Loader --> Recursive Character Text Splitter
       |                              |
       +------------------------------+
                                      |
                                      v
                           OpenAI Embeddings
                                      |
                                      v
                         Supabase Vector Store

CHAT ASSISTANT

Chat message --> AI Agent --> Response
                    |  |  |
                    |  |  +--> Supabase Vector Store (retrieval)
                    |  |           |
                    |  |           +--> OpenAI Embeddings
                    |  |
                    |  +----> Postgres Chat Memory
                    |
                    +-------> OpenAI Chat Model
```

### Ingestion workflow

1. Downloads the knowledge-base document from Google Drive.
2. Extracts document text with the Default Data Loader.
3. Splits the text into smaller chunks using a Recursive Character Text Splitter.
4. Creates OpenAI embeddings for each chunk.
5. Stores the chunks and embeddings in Supabase for semantic search.

### Chat workflow

1. Receives a question through the n8n chat trigger.
2. Sends the question to an AI Agent backed by an OpenAI Chat Model.
3. Retrieves relevant knowledge-base chunks from Supabase.
4. Uses Postgres Chat Memory to preserve conversational context.
5. Returns an answer based on the retrieved information.

## Tech stack

| Technology | Role |
| --- | --- |
| [n8n](https://n8n.io/) | Workflow orchestration, ingestion, and chat interface |
| [OpenAI](https://openai.com/) | Chat model and embedding generation |
| [Supabase](https://supabase.com/) | Vector database and semantic retrieval |
| PostgreSQL | Conversation memory |
| Google Drive | Knowledge-base document storage |

## How to use

### 1. Configure services

Create credentials in n8n for Google Drive, OpenAI, Supabase, and PostgreSQL. Store keys and passwords in n8n's credential manager or environment variables.

### 2. Import the workflows

Import the project’s exported n8n workflow JSON files into your n8n instance. Attach the appropriate credentials to every node.

### 3. Index the knowledge base

Upload the source document to Google Drive, select it in the ingestion workflow, and run the workflow. This loads, chunks, embeds, and saves the document to Supabase.

### 4. Start the chat assistant

Open the chat workflow and send a question. The AI Agent searches the Supabase vector store before responding.

### 5. Update content

When the source document changes, run the ingestion workflow again so the vector store contains the latest content.

## Example questions

```text
What services does Andrews Construction provide?
What project types does the company support?
Tell me about the company's safety priorities.
Where is Andrews Construction located?
```

## Production considerations

- Do not commit API keys, database passwords, or other credentials.
- Add source citations and a human handoff path before using the assistant with customers.
- Restrict access to the chat workflow and vector store if the knowledge base contains private content.

Built by [Jenna DiCiacco](https://github.com/jdiciacco96)
