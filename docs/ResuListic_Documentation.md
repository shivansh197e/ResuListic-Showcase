# ResuListic — Complete Project Documentation

## 1. Project Overview
**ResuListic** is an AI-powered Career Intelligence Platform. It takes raw resumes (PDF, DOCX, Images), extracts structural data and skills using OCR and NLP, and dynamically assesses the candidate's proficiency through a Gemini AI-powered adaptive test engine.

### Core Modules
1. **Resume Parser Engine:** Parses multi-format files, utilizing OCR as a fallback for scanned documents.
2. **NLP & Entity Extractor:** Classifies text into sections and maps raw text to 170+ standardized tech skills using SpaCy.
3. **Skill Test Engine:** Orchestrates dynamic technical assessments blending AI-generated MCQs and deterministic Database coding questions.

---

## 2. Local Setup & Installation

### Prerequisites
- **Python 3.12+**
- **Tesseract OCR** (For image-based PDF parsing)
- **Poppler** (For PDF-to-Image conversion)
- **MongoDB Atlas** Account
- **Google Gemini API Key**

### Installation Steps
1. **Clone & Virtual Environment:**
   ```bash
   git clone <repo-url>
   cd ResuListic
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```
2. **Install Dependencies:**
   ```bash
   pip install -r requirements.txt
   python -m spacy download en_core_web_md
   ```
3. **Environment Variables (`.env`):**
   ```env
   # Database
   MONGODB_URI="mongodb+srv://<user>:<pass>@<cluster>.mongodb.net"

   # AI Key
   GEMINI_API_KEY="your_google_gemini_key"

   # Local Windows Dependencies (Leave blank on Linux/Cloud)
   TESSERACT_CMD="C:\Program Files\Tesseract-OCR\tesseract.exe"
   POPPLER_PATH="C:\poppler\Library\bin"
   ```
4. **Run the Server:**
   ```bash
   uvicorn main:app --reload
   ```

---

## 3. Project Structure

```text
ResuListic/
├── main.py                     # FastAPI Application Entry Point
├── requirements.txt            # Dependency Definitions
├── docs/                       # Project Documentation
├── temp_uploads/               # Ephemeral storage for incoming files
└── ml-services/
    ├── api/                    # (Planned) Separated API routers
    ├── builder/
    │   └── resume_builder.py   # Assembles final MongoDB document
    ├── extraction/
    │   ├── contact_extractor.py
    │   ├── section_classifier.py
    │   └── skills_extractor.py # SpaCy logic (170+ skills)
    ├── parser/
    │   ├── text_extractor.py   # Master router for file parsing
    │   ├── pdf_extractor.py
    │   ├── docx_extractor.py
    │   └── image_extractor.py  # OCR Engine
    ├── schema/
    │   ├── __init__.py
    │   └── question_schema.py  # Pydantic data contracts
    └── skill_test/
        ├── ai_question_generator.py # Gemini GenAI logic
        ├── circular_tracker.py      # Modulo-based Hard Question Engine
        ├── data_builder.py          # Database seeding scripts
        └── question_selector.py     # Exam Assembly Orchestrator
```

---

## 4. API Reference

### 4.1 Upload Resume
* **Endpoint:** `POST /api/parse-resume`
* **Format:** `multipart/form-data`
* **Inputs:** `file` (PDF/DOCX/IMG), `user_id` (String)
* **Description:** Runs OCR/NLP pipeline and saves the extracted candidate profile to the `resumes` MongoDB collection.

### 4.2 Fetch Extracted Skills (Pending Implementation)
* **Endpoint:** `GET /api/user-skills/{user_id}`
* **Description:** Retrieves the high/medium confidence skills extracted from the user's resume. 

### 4.3 Start Skill Test (Pending Implementation)
* **Endpoint:** `POST /api/start-test`
* **Format:** `application/json`
* **Inputs:** `user_id` (String), `selected_skills` (List of Strings, optional)
* **Description:** Orchestrates a 10-question dynamic exam (5 Easy AI MCQs, 3 Random Moderate Coding, 2 Deterministic Hard Coding).

### 4.4 Submit Test (Pending Implementation)
* **Endpoint:** `POST /api/submit-test`
* **Format:** `application/json`
* **Inputs:** `session_id`, `user_id`, `answers` (List of dicts)
* **Description:** Submits raw user inputs for final AI evaluation.

---

## 5. Architectural Deep Dives

### The Parser Fallback Strategy
To guarantee maximum text extraction, `text_extractor.py` features a smart fallback system:
1. It attempts to read PDF metadata via `pdfplumber`. 
2. If text is extracted (and length > 10 characters), it considers the PDF a text-document.
3. If no text is found, it converts the PDF into high-resolution images via `pdf2image` and runs Tesseract OCR (`image_extractor.py`) across all pages.

### The Skill Test "Circular Linked List"
To ensure candidates do not receive the exact same "Hard" coding questions on retakes, a Circular Tracker algorithm is employed:
- Hard questions are stored sequentially (0 to 49).
- A pointer is maintained in the `user_test_state` collection.
- Upon a test request, the engine serves questions at `pointer` and `pointer+1`.
- The pointer is advanced via modulo math: `new_pointer = (old_pointer + 2) % 50`.

### Cloud Deployment & Dependencies
The codebase is designed to be cloud-agnostic. While `POPPLER_PATH` and `TESSERACT_CMD` are required for local Windows development, they fallback gracefully to the system `PATH` via `os.environ.get()` when deployed on Linux-based Docker environments (e.g., Google Cloud Run, AWS ECS).
