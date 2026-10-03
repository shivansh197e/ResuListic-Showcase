# ResuListic System Design Document

*(Note: If you need this as a PDF, install the "Markdown PDF" extension in VS Code, right-click this file, and select "Export to PDF".)*

## 1. Introduction
This document outlines the system design for ResuListic, detailing the structural choices, data models, and ML pipelines designed to scale an AI-powered technical recruitment platform.

## 2. System Context
- **Users:** Candidates uploading resumes and taking technical tests.
- **External Systems:** 
  - Google Gemini API (LLM for text generation and evaluation).
  - MongoDB Atlas (Persistent Storage).
  - Auth Provider (e.g., Firebase).

## 3. Core Subsystems

### 3.1. Parsing & Ingestion
To handle the unpredictability of user-uploaded resumes, the system uses a tiered fallback approach:
1. **Primary:** `pdfplumber` / `python-docx` for native text extraction (Fast, O(1) per page).
2. **Secondary:** If no native text is found, the file is converted to a high-res image (Lanczos resampling, Contrast enhancement 2.5x) and passed through `pytesseract` OCR (Slower, CPU intensive).

### 3.2. ML Entity Extraction
- Uses **SpaCy** (`en_core_web_md`) for Named Entity Recognition (NER).
- A custom matching pipeline parses 170+ skills.
- **Design Decision:** Skills are tiered. "High Confidence" skills must appear in contextually relevant sections (e.g., "Experience" or "Projects"), reducing false positives from keyword stuffing at the bottom of a resume.

### 3.3. Test Orchestration (The Exam Engine)
The Exam Engine must balance real-time performance with deterministic behavior.
- **AI MCQs (Real-time):** Gemini generates 5 MCQs on the fly. This ensures the questions are never exactly the same. We use *Structured Outputs* (Pydantic schemas mapped to Gemini response schemas) to guarantee the frontend never receives broken JSON.
- **Database Coding Questions (Pre-computed):** Generating complex coding problems in real-time is prone to hallucinations. Therefore, Moderate and Hard questions are pre-scraped and stored in MongoDB.
- **Circular Linked List:** A stateful modulo algorithm `(pointer + count) % size` ensures a candidate never receives the same Hard question on immediate retakes, cycling fairly through a pool of 50 questions.

### 3.4. Evaluation System (Phase 5 Plan)
- Raw user code will be submitted alongside the `reference_solution`.
- The LLM acts as an "Expert Grader", prompted to evaluate purely on Logic, Time Complexity, and Edge Cases, returning a structured score between 1-10.

## 4. Scalability & Constraints
- **Stateless API:** The FastAPI backend is completely stateless. The `user_test_state` is offloaded to MongoDB. This allows horizontal scaling (e.g., spinning up multiple Docker containers).
- **OCR Bottleneck:** OCR is CPU-heavy. In a production environment, `/api/parse-resume` should ideally be offloaded to a background task queue (like Celery + Redis), but is currently synchronous for simplicity.
- **Database Constraints:** 
  - `_id` fields use the external Auth UUID (e.g., Firebase ID) rather than Mongo's default ObjectIds to ensure easy cross-referencing between Frontend and Backend without secondary lookups.
