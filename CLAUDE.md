# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Course Materials RAG (Retrieval-Augmented Generation) System** - a full-stack web application that enables users to query course materials and receive intelligent, context-aware responses using semantic search and AI-powered responses.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
**IMPORTANT: Always use `uv` for all dependency management - never use `pip` directly**

```bash
# Install all dependencies
uv sync

# Add new dependency
uv add package-name

# Add development dependency
uv add --dev package-name

# Remove dependency
uv remove package-name

# Update all dependencies
uv sync --upgrade

# Run Python commands with uv
uv run python script.py
uv run uvicorn app:app --reload --port 8000
```

### Environment Setup
Required `.env` file in root directory:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

### Core RAG Pipeline Flow
```
User Query → FastAPI → RAG System → AI Generator → Claude API
                                        ↓
                                   Tool Manager → Search Tool → Vector Store → ChromaDB
                                        ↓
Claude API ← AI Generator ← Tool Results ← Search Results ← Formatted Documents
```

### Key Components

**Backend Orchestration (Python/FastAPI):**
- `rag_system.py` - Main orchestrator that coordinates all components
- `ai_generator.py` - Manages Claude API interactions and tool execution  
- `search_tools.py` - Tool-based search system with CourseSearchTool
- `vector_store.py` - ChromaDB integration with semantic search
- `document_processor.py` - Processes course documents into structured chunks
- `session_manager.py` - Maintains conversation history across requests

**Frontend (Vanilla JS):**
- `frontend/script.js` - Handles user interaction, API calls, and response rendering
- Uses marked.js for markdown rendering

**Data Models:**
- `Course` - Represents a complete course with lessons
- `Lesson` - Individual lessons within courses  
- `CourseChunk` - Text chunks for vector storage with metadata

### Document Processing Pipeline

1. **Expected Format**: Course documents follow structured format:
   ```
   Course Title: [title]
   Course Link: [url]
   Course Instructor: [instructor]
   
   Lesson 0: Introduction
   Lesson Link: [lesson_url]
   [content...]
   ```

2. **Processing Steps**:
   - Extract course metadata (title, link, instructor)
   - Parse lessons by "Lesson X:" markers
   - Create sentence-based chunks with overlap (800 chars, 100 overlap)
   - Add context to chunks: "Course [title] Lesson [number] content: [chunk]"

3. **Storage**: ChromaDB collections:
   - `course_catalog` - Course metadata for name resolution
   - `course_content` - Chunked content for semantic search

### Tool-Based Search Architecture

The system uses **Claude's tool calling** rather than traditional RAG retrieval:

1. **Query Processing**: Claude decides whether to use search tool based on query
2. **Tool Execution**: `CourseSearchTool` performs semantic search with filters
3. **Context Assembly**: Search results formatted with course/lesson context
4. **Response Generation**: Claude generates final response using search results

### Configuration

All settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 characters for text chunks
- `CHUNK_OVERLAP`: 100 characters overlap between chunks  
- `MAX_RESULTS`: 5 maximum search results
- `MAX_HISTORY`: 2 conversation messages remembered
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2" for sentence transformers
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"

### API Endpoints

- `POST /api/query` - Main query endpoint that processes user questions
- `GET /api/courses` - Returns course statistics and titles
- `GET /` - Serves static frontend files

### Session Management

- Each conversation gets unique session ID
- Conversation history maintained for context
- History limited to `MAX_HISTORY` exchanges to control token usage

### Key Development Considerations

- **Two-stage Claude calls**: First for tool decision, second for final response
- **Source tracking**: Sources flow through entire pipeline for UI display
- **Context preservation**: Course and lesson metadata maintained through chunking
- **Error handling**: Graceful fallbacks throughout the search pipeline
- **Performance**: Sentence-based chunking optimized for semantic coherence

### Adding New Course Documents

Place `.txt`, `.pdf`, or `.docx` files in `docs/` directory. The system automatically:
1. Detects new documents on startup
2. Processes them through the document pipeline
3. Stores chunks in ChromaDB
4. Makes them available for search

### Frontend Architecture

- **Session persistence**: Maintains conversation across page reloads
- **Real-time updates**: Loading animations and progressive response rendering
- **Markdown support**: AI responses rendered with syntax highlighting
- **Collapsible sources**: Source citations displayed in expandable sections