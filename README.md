# ResuListic 🚀

An AI-powered Career Intelligence and Recommendation Platform. ResuListic uses Machine Learning (NLP, OCR) and Generative AI to parse resumes, accurately extract candidate skills, and orchestrate dynamic, AI-evaluated technical assessments.

---

## 🌟 Key Features

* **Intelligent Resume Parsing:** Extracts text from PDFs, DOCX, and image files. Automatically falls back to **Tesseract OCR** for scanned documents.
* **NLP Entity & Skill Extraction:** Utilizes a custom **SpaCy** pipeline to classify resume sections, extract contact information, and map raw text against a database of 170+ standardized tech skills.
* **Confidence Scoring:** Intelligently categorizes skills into "High Confidence" (found in both Skills & Experience sections) and "Medium Confidence".
* **Dynamic Skill Testing:** Generates a custom 10-question technical exam on the fly:
  * **5 AI MCQs:** Generated dynamically using **Google Gemini 2.5 Flash**.
  * **3 Moderate Coding Tasks:** Pulled randomly from the database.
  * **2 Hard Coding Tasks:** Served via a mathematical **Circular Linked List** algorithm to guarantee consecutive, non-repeating questions on test retakes.

---

## 🛠️ Tech Stack

* **Backend Framework:** FastAPI (Python)
* **Database:** MongoDB Atlas (NoSQL)
* **AI & NLP:** Google GenAI (Gemini), SpaCy (`en_core_web_md`)
* **OCR & Extraction:** Tesseract OCR, Poppler, `pdfplumber`, `pdf2image`, `pytesseract`
* **Frontend:** React.js (Pending Implementation)

---

## ⚙️ Local Development Setup

### Prerequisites
Before you begin, ensure you have the following installed on your machine:
* Python 3.12+
* [Tesseract OCR](https://github.com/UB-Mannheim/tesseract/wiki) (For Windows developers)
* [Poppler](https://github.com/oschwartz10612/poppler-windows/releases) (For Windows developers)
* A MongoDB Atlas cluster and a Google Gemini API Key.

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/ResuListic.git
cd ResuListic
```

### 2. Set up a Virtual Environment
```bash
# Create the virtual environment
python -m venv venv

# Activate it (Windows)
venv\Scripts\activate

# Activate it (macOS/Linux)
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt

# Download the required SpaCy NLP model
python -m spacy download en_core_web_md
```

### 4. Environment Variables
Create a `.env` file in the root directory and add the following:
```env
# MongoDB Connection
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net"

# Google Gemini API
GEMINI_API_KEY="your_api_key_here"

# ONLY FOR WINDOWS LOCAL DEVELOPMENT (Leave blank for Cloud/Linux deployment)
TESSERACT_CMD="C:\Program Files\Tesseract-OCR\tesseract.exe"
POPPLER_PATH="C:\path\to\poppler\bin"
```

### 5. Run the Server
Start the FastAPI server with hot-reloading:
```bash
uvicorn main:app --reload
```
The API will be available at `http://localhost:8000`. You can view the interactive Swagger API documentation at `http://localhost:8000/docs`.

---

## 📂 Project Structure

```text
ResuListic/
├── main.py                     # FastAPI Application Entry Point
├── requirements.txt            # Dependency Definitions
├── docs/                       # Detailed System Documentation
├── temp_uploads/               # Ephemeral storage for incoming files
└── ml-services/
    ├── builder/                # Assembles final JSON for MongoDB
    ├── extraction/             # SpaCy NLP, Contact, and Section classifiers
    ├── parser/                 # OCR, PDF, and DOCX routers
    ├── schema/                 # Pydantic strict data models
    └── skill_test/             # Test generation, Circular logic, AI orchestrators
```

---

## ☁️ Cloud Deployment
ResuListic is designed to be cloud-native. When deploying to a Linux-based Docker environment (e.g., Google Cloud Run, AWS ECS), the application automatically detects the OS. `Poppler` and `Tesseract` will utilize the system's global `PATH`, ignoring the Windows-specific `.env` overrides.

