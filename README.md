# charcotgpt

A RAG (Retrieval-Augmented Generation) system for medical consultation assistance, built with local LLM orchestration.

## Architecture

The system consists of three main components:

1. **Engine (LLM):** Llama 3 8B model running via Ollama
2. **Archive (Vector Database):** ChromaDB for storing embedded document chunks
3. **Orchestrator (Python Script):** LangChain-based script connecting user input to retrieval and generation

## Key Behaviors

### Data Ingestion
- Monitors `/prontuarios/` (PDFs, TXTs, MED files) and `/agenda/` (ICS calendar)
- Splits documents into 500-word chunks
- Embeds chunks using nomic-embed-text model
- Stores vectors in local ChromaDB

### Read-Only Queries
- Vectorizes user questions
- Retrieves top 5 similar chunks
- Augments LLM prompt with context
- Generates factual responses based only on stored data

### Action Queries (Tool Use)
- LLM can call tools like `agendar_consulta` for scheduling
- Orchestrator executes actions and confirms via LLM

## Technology Stack

- **LLM:** Ollama with Llama-3-8B-Instruct-Q5_K_M.gguf
- **Vector DB:** ChromaDB
- **Orchestrator:** Python 3.11+ with LangChain
- **Libraries:** ollama-python, pdfplumber, icalendar, Flask/FastAPI for web interface

## Development Steps

### 1. Setup Environment
- Install Ollama on Arch Linux: `pacman -S ollama`
- Pull Llama-3-8B-Instruct-Q5_K_M.gguf model: `ollama pull llama3:8b-instruct-q5_K_M`
- Create Python virtual environment: `python -m venv venv && source venv/bin/activate`
- Install dependencies: `pip install langchain ollama chromadb pdfplumber icalendar fastapi uvicorn`

### 2. Create Orchestrator Structure
- Create main script `orchestrator.py` with modules for:
  - `ingestion.py`: Handles document loading and embedding
  - `retrieval.py`: Manages vector search and context retrieval
  - `generation.py`: Interfaces with Ollama for response generation
  - `tools.py`: Defines action tools like appointment scheduling
- Set up configuration in `config.py` for paths, model settings, and DB location

### 3. Implement Data Ingestion Pipeline
- Use `pdfplumber` to extract text from PDFs in `/prontuarios/`
- Use `icalendar` to parse calendar data from `/agenda/calendar.ics`
- Split texts into 500-word chunks using LangChain's `RecursiveCharacterTextSplitter`
- Initialize ChromaDB client and collection
- Embed chunks with `nomic-embed-text` via Ollama embeddings
- Store vectors with metadata (source file, chunk index) in ChromaDB

### 4. Implement RAG Query Pipeline
- Create query endpoint that vectorizes user input using same embedding model
- Perform similarity search in ChromaDB to retrieve top 5 relevant chunks
- Construct augmented prompt with system instructions, retrieved context, and user query
- Send prompt to Ollama via `ollama-python` for generation
- Return LLM response to user

### 5. Add Tool Use Functionality
- Define tool schemas in LangChain (e.g., `agendar_consulta` with parameters: nome_paciente, data_hora)
- Modify generation to use tool-calling capable prompt format
- Parse LLM output for tool calls (JSON format)
- Execute tools: for scheduling, modify `.ics` file or integrate with calendar API
- Send tool execution results back to LLM for final response generation

### 6. Build Web Interface
- Use FastAPI to create REST API endpoints for chat and ingestion
- Create HTML/CSS/JS frontend with chat interface (input field, message history)
- Implement WebSocket or polling for real-time conversation updates
- Add file upload functionality for manual document ingestion
- Serve static files and handle CORS for local development

### 7. Test Complete System
- Create sample documents in `/prontuarios/` and `/agenda/`
- Run ingestion script to populate ChromaDB
- Test read-only queries: verify accurate retrieval and generation
- Test action queries: confirm tool execution and scheduling
- Test web interface: full conversation flow from browser
- Performance testing: response times, memory usage on RTX 5060

Project created with Odin.