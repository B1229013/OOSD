Object-Oriented Software Design Final Project

> **Course:** Object-Oriented Software Design (物件導向軟體設計)
> **Project:** A Smart Shopping & Indoor-Navigation Companion

## 1. Software + Hardware Project Overview

Our information system is a **smart shopping companion** for everyday grocery
trips. It runs as an **Android app** that helps a user *plan* what to buy,
*budget and track* spending and nutrition, *get AI advice*, *find nearby
stores*, and finally *navigate inside the store* to each product using nothing
but the phone camera. The in-store navigation is powered by a separate
**Python vision backend** called **UniGoal**.

### 1.1 Hardware

| Component | Role |
|-----------|------|
| **Smartphone (Android)** | Runs the app. Its **camera** scans receipts and nutrition labels and drives in-store navigation; its **GPS** powers nearby-store search; its screen is the whole UI. |
| **Backend server (CPU or CUDA GPU)** | Runs the UniGoal navigation backend: the object detector, the OCR engine, and the vision-language model. GPU optional — the system auto-detects and falls back to CPU. |

### 1.2 Software

**Frontend — Android app** (`android-app/`)

| Area | Technology |
|------|-----------|
| Language / UI | Kotlin + Jetpack Compose (Material 3) |
| Networking | OkHttp + Retrofit (JSON over HTTP) |
| On-device vision | Google ML Kit text recognition |
| Local data | Kotlin serialization (`@Serializable` models) |

**Backend — UniGoal navigation server** (`backend/`)

| Area | Technology |
|------|-----------|
| Language / API | Python 3.12 + FastAPI |
| Object detection | GroundingDINO (open-vocabulary detector) |
| Text reading | EasyOCR (English + Traditional Chinese) |
| Reasoning | Ollama vision-language model (`llama3.2-vision`) |
| Map | NetworkX directed graph (topological map) |

**External cloud services (actors)**

| Service | Used for |
|---------|----------|
| **Groq API** (`llama-3.3-70b`) | The in-app AI assistant and receipt-to-line-item parsing. |
| **PaddleOCR cloud API** | Reading receipts and nutrition labels into text. |
| **Google Places API** | Finding nearby stores; AR overlay. |
| **UniGoal backend + Ollama VLM** | Real-time in-store visual navigation. |

### 1.3 The eight functions of the system

| # | Function | Primary actor | Where it lives |
|---|----------|---------------|----------------|
| 1 | **Manage Shopping List** | User | App — *清單* tab |
| 2 | **Scan Receipt & Track Budget** | User | App — *預算* tab (PaddleOCR + Groq) |
| 3 | **Log Diet & Nutrition** | User | App — *分析* tab (PaddleOCR) |
| 4 | **Consult AI Shopping Assistant** | User | App — *助理* tab (Groq) |
| 5 | **Find Nearby Stores** | User | App — nearby-stores sheet (Google Places) |
| 6 | **In-Store Visual Navigation** | User | App *navigation* screen + UniGoal backend |
| 7 | **Build Environment Map (offline)** | Mapping Operator | Backend — `batch_mapper` |
| 8 | **Query Product Location on Map** | User / Operator | Backend — `navigate_to_product` |

### 1.4 Actors

- **User** — the shopper using the app.
- **Mapping Operator** — the technician who builds the topological map of a new
  store offline.
- **Groq AI Service**, **PaddleOCR Service**, **Google Places Service**,
  **UniGoal Backend / Ollama VLM** — external/supporting actors the system calls.

---

## 2. Repository layout

| Path | Contents |
|------|----------|
| [`docs/`](docs/) | The five design documents (Chapters 4–8) as Markdown with Mermaid diagrams. |
| [`docs/images/`](docs/images/) | The diagrams rendered to PNG (used by the Word report). |
| [`report/`](report/) | **`UniGoal_OOSD_Report.docx`** — the full Word report combining all chapters. |
| [`android-app/`](android-app/) | The Android shopping-companion app (Kotlin / Jetpack Compose). |
| [`backend/`](backend/) | The UniGoal navigation backend (Python / FastAPI). |

---

## 3. How the documents map to the assignment

| Chapter | Deliverable | File |
|---------|-------------|------|
| **Ch. 4** | A complete **glossary** (Term / Definition / Notes). | [`docs/01-glossary.md`](docs/01-glossary.md) |
| **Ch. 5** | **N use case diagrams** — one per function (N = 8). | [`docs/02-use-case-diagrams.md`](docs/02-use-case-diagrams.md) |
| **Ch. 6** | **M use case descriptions** per diagram — one *normal* scenario + *exception* scenarios. | [`docs/03-use-case-descriptions.md`](docs/03-use-case-descriptions.md) |
| **Ch. 7** | One **activity diagram** per use case diagram. | [`docs/04-activity-diagrams.md`](docs/04-activity-diagrams.md) |
| **Ch. 8** | One **class diagram** for the whole information system. | [`docs/05-class-diagram.md`](docs/05-class-diagram.md) |

### How to read the diagrams

- **Use case diagrams**: a box is the *system boundary*; rounded shapes are *use
  cases*; lines join actors to the use cases they take part in; dashed arrows
  carry `«include»` / `«extend»`.
- **Activity diagrams**: read top-to-bottom — start node, rectangular actions,
  diamond decisions with guard labels, end nodes. Exception paths branch off the
  decisions.
- **Class diagram**: two namespaces (Android client, navigation backend) with
  key attributes, operations, and the associations/dependencies between classes.

---
