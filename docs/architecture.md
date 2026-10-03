# ResuListic (ResuMate) - System Architecture

## 1. High-Level System Overview
ResuListic follows a modern, decoupled client-server architecture:
- **Frontend:** React.js (Handles UI, File Uploads, Exam Interface, Auth).
- **Backend:** FastAPI (Python 3.12). Serves as the central orchestrator for ML pipelines and API requests.
- **Database:** MongoDB Atlas (NoSQL) for flexible document storage.
- **AI/ML Layer:** Google Gemini 2.5 Flash (Generative AI) + SpaCy (NLP) + Tesseract (OCR).

---

## 2. Component Architecture

### A. Parser Engine (`ml-services/parser/`)
Responsible for converting unstructured binary files into raw text.
- **`text_extractor.py`**: Master router.
- **`pdf_extractor.py`**: Uses `pdfplumber` for native PDF text extraction.
- **`image_extractor.py`**: Uses `pytesseract` and OpenCV/Pillow for OCR fallback.
- **`docx_extractor.py`**: Uses `python-docx`.

### B. NLP Extraction Engine (`ml-services/extraction/`)
Responsible for understanding the raw text.
- **`section_classifier.py`**: Splits text into Experience, Education, Projects, etc.
- **`contact_extractor.py`**: Regex-based extraction of emails, phones, and GitHub/LinkedIn links.
- **`skills_extractor.py`**: Uses SpaCy (`en_core_web_md`) to map text against a dictionary of 170+ standardized tech skills. Separates them into High and Medium confidence.

### C. Skill Test Orchestrator (`ml-services/skill_test/`)
Responsible for assembling the dynamic assessments.
- **`ai_question_generator.py`**: Prompts Gemini API to generate 5 Easy MCQs using Pydantic Structured Outputs.
- **`circular_tracker.py`**: Manages the modulo-based pointer system for deterministic Hard questions.
- **`question_selector.py`**: Assembles the final 10-question test session (5 Easy MCQs, 3 Mod Coding, 2 Hard Coding).

---

## 3. Database Schema Design (MongoDB)

### `resumes` Collection
Stores the parsed candidate profile.
```json
{
  "_id": "user_id_from_auth",
  "status": "raw",
  "profile": { "name": "...", "headline": "..." },
  "contact": { "emails": [], "phones": [], "links": {} },
  "sections": { "summary": "...", "experience": "..." },
  "enrichment": {
    "skills": { "high": [], "medium": [] },
    "skill_test_result": null
  }
}
```

### `questions_moderate` & `questions_hard` Collections
Stores the predefined coding questions.
```json
{
  "skill": "Python",
  "index": 42,
  "title": "Reverse Linked List",
  "description": "Write a function to...",
  "source": "manual",
  "tags": ["dsa", "lists"]
}
```

### `user_test_state` Collection
Tracks the Circular Linked List pointer for hard questions.
```json
{
  "_id": { "user_id": "user_123", "skill": "Python" },
  "hard_pointer": 12,
  "test_count": 2,
  "last_test_at": "2026-10-03T12:00:00Z"
}
```

---

## 4. Infrastructure & Deployment Architecture
- **Containerization:** The FastAPI backend will be containerized using Docker. `Tesseract` and `Poppler-utils` will be installed at the OS level within the Debian-based Python image.
- **Hosting (Proposed):** Google Cloud Run or AWS App Runner for auto-scaling serverless containers.
- **Storage:** Ephemeral local storage `/temp_uploads` is used for processing resumes before immediate deletion.
