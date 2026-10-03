# Project Roadmap

This document outlines the strategic plan for completing ResuListic and moving it to a production-ready state.

---

## 🎯 Milestone 1: Core Backend Completion (Current Phase)

### Phase 4: Skill Test Generation APIs
- [ ] Add `GET /api/user-skills/{user_id}` to `main.py`.
- [ ] Add `POST /api/start-test` to `main.py`.
- [ ] Add `POST /api/submit-test` to `main.py`.
- [ ] Test the full pipeline end-to-end via Swagger UI.

### Phase 5: AI Auto-Evaluation Engine
- [ ] Build the Evaluator Module (`evaluator.py`).
- [ ] Implement Gemini prompts to analyze raw user code against the `reference_solution` from the DB.
- [ ] Generate a final `Competency Score` (e.g., 8.5/10) with detailed feedback (Time Complexity, Space Complexity, Code Quality).
- [ ] Update the user's document in the `resumes` collection with their final test scores.

---

## 🎯 Milestone 2: Frontend & Integration

### Phase 6: React Frontend
- [ ] Scaffold React / Vite application.
- [ ] Implement Firebase / Auth0 authentication (to generate `user_id`).
- [ ] Build "Upload Resume" UI & Loading states.
- [ ] Build "Profile Review" UI.
- [ ] Build "Skill Selection" UI.
- [ ] Build "Test Engine" UI (MCQ Radio buttons + Monaco Code Editor).
- [ ] Build "Results & Feedback" Dashboard.

---

## 🎯 Milestone 3: Advanced Features

### Phase 7: Job Matching System
- [ ] Web Scrape or import actual Job Descriptions.
- [ ] Build a Vector Database (e.g., Pinecone or MongoDB Vector Search).
- [ ] Embed the candidate's Resume + Skill Test Score into a vector.
- [ ] Perform cosine similarity search to match candidates with the best-fit job IDs.

### Phase 8: Mock Interviewer (Stretch Goal)
- [ ] Implement real-time WebSockets.
- [ ] Use Gemini Live API for a voice-based technical interview.

---

## 🎯 Milestone 4: Production & Deployment

- [ ] Write `Dockerfile` replacing Windows-specific binaries with Linux `apt-get` equivalents (`tesseract-ocr`, `poppler-utils`).
- [ ] Setup CI/CD pipeline using GitHub Actions.
- [ ] Deploy Backend to Google Cloud Run.
- [ ] Deploy Frontend to Vercel or Netlify.
- [ ] Perform security audit (API rate limiting, CORS configuration).
