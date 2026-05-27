# Chapter 4 — Glossary

A complete glossary of the terms used throughout the design documents, covering
both the **Android shopping app** (frontend) and the **UniGoal navigation
backend**. Terms are grouped by area for readability.

## A. Shopping-app domain terms

| Term | Definition (explanation) | Notes |
|------|--------------------------|-------|
| **Shopping companion** | The overall information system: an Android app that plans, budgets, advises, and navigates a grocery trip. | Frontend + backend together. |
| **Shopping item** | One thing the user intends to buy: name, quantity, price, checked state, store, location, timestamps. | Modelled by `ShoppingItem`. |
| **Shopping list** | A dated collection of shopping items with a status (in-progress / done). | Modelled by `ShoppingList`. |
| **Checked / purchased** | A shopping item marked as bought; it then counts toward spending. | `isChecked = true`. |
| **Category** | The classification of an item (e.g. food, daily goods). | Auto-assigned by the category classifier. |
| **Budget** | The user's monthly spending limit, against which purchases are compared. | Drives the *預算* (budget) tab. |
| **Spending by category** | Total money spent grouped by item category. | Powers budget charts. |
| **Receipt scan** | Photographing a paper receipt so the app can extract its line items automatically. | Uses PaddleOCR + Groq. |
| **Line item** | One product row parsed from a receipt: name, price, quantity. | Modelled by `GeminiLineItem`. |
| **Diet record** | A logged food entry with nutrition data (calories, carbs, protein, fat, etc.). | Modelled by `DietRecord`. |
| **Nutrition label scan** | Photographing a food/nutrition label so the app can read its values. | Uses PaddleOCR. |
| **User profile** | The user's personal/health data: name, gender, birthday, height, weight, allergies, disease, activity level. | Modelled by `UserProfile`; personalises AI advice. |
| **Shopping context** | A single JSON snapshot aggregating inventory + budget + health, sent to the AI with every query. | Modelled by `ShoppingContext`; enables cross-module answers. |
| **AI shopping assistant** | The in-app chat that answers questions using the shopping context. | Powered by the Groq LLM. |
| **Nearby store** | A shop near the user's GPS location returned by the maps service. | Found via Google Places. |
| **AR overlay** | An augmented-reality camera view that points toward nearby stores. | `NearbyStoresAr` screen. |

## B. Navigation domain terms

| Term | Definition (explanation) | Notes |
|------|--------------------------|-------|
| **UniGoal** | The Python vision backend that provides in-store visual navigation. | Called by the app over REST. |
| **Indoor navigation** | Guiding a person to a destination inside a building where GPS is unreliable. | Uses vision, not GPS/beacons. |
| **Goal** | The free-text destination the user wants to reach, e.g. "find the refrigerator". | Set when a navigation session starts. |
| **Goal object** | A single detectable item derived from the goal (e.g. `refrigerator`). | Produced by *goal decomposition*. |
| **Goal decomposition** | Turning the free-text goal into a short list of goal objects. | Done by the VLM; keyword fallback if it fails. |
| **Session** | One navigation attempt for one goal, holding all per-attempt state. | 8-character id. |
| **Turn** | One round: the user uploads a photo (or answers) and the system replies with an action. | A session is a sequence of turns. |
| **Action** | The system's decision for a turn: `MOVE`, `ASK`, or `ARRIVED`. | Modelled by `VLMAction`. |
| **MOVE / ASK / ARRIVED** | Walk this way / I need a clarification / you have reached the goal. | The three possible actions. |
| **Guidance** | The natural-language instruction returned each turn. | e.g. "Turn right toward the back wall." |
| **Topological map** | A graph of places: nodes are visited locations, edges are the movements between them. | No metric coordinates. |
| **Node / Edge** | A location in the map / a directed movement labelled with its action. | Stored in a NetworkX `DiGraph`. |
| **Zone** | A cluster of photos representing one coherent area (batch mapping). | Built via Jaccard similarity. |
| **Shortest path** | Fewest-edge route between two nodes; powers step-by-step directions. | Computed with NetworkX. |
| **Arrival gate** | A safety check that confirms `ARRIVED` only with real evidence (a detected goal object or a matching sign); otherwise downgrades to a confirm `ASK`. | Prevents false "you've arrived" claims. |
| **Annotated photo** | The uploaded photo redrawn with detection boxes, OCR text, and a guidance banner. | Returned to the user. |

## C. Perception, OCR & AI terms

| Term | Definition (explanation) | Notes |
|------|--------------------------|-------|
| **GroundingDINO** | An open-vocabulary object detector that finds objects named by free text. | Lets UniGoal detect any goal object without retraining. |
| **Open-vocabulary detection** | Detection where the classes are given as text at run time, not fixed at training. | Why "fire hydrant" can be detected on demand. |
| **Detection** | One detected object: label, bounding box, confidence score. | Top 5 kept per photo. |
| **Bounding box** | Rectangle `[x1,y1,x2,y2]` (pixels) enclosing a detected object. | Converted from center-format. |
| **Confidence score** | A 0–1 value of how sure a model is about a result. | Below-threshold results are dropped. |
| **OCR** | Optical Character Recognition — reading text out of an image. | Used for signs, labels, receipts. |
| **EasyOCR** | The backend OCR engine (English + Traditional Chinese). | Reads in-store signs. |
| **PaddleOCR** | The cloud OCR service the app uses for receipts and nutrition labels. | Returns recognised text. |
| **ML Kit text recognition** | Google's on-device OCR used inside the app. | Fast local text reads. |
| **Sign match** | An OCR string judged to indicate the goal's section/aisle (substring or fuzzy, incl. bilingual synonyms). | A strong arrival signal. |
| **VLM (Vision-Language Model)** | A model that takes an image + text and returns text; here it chooses the next navigation action. | Served by Ollama. |
| **Ollama** | The local service hosting the navigation VLM over HTTP. | External supporting actor. |
| **`llama3.2-vision`** | The vision model Ollama serves for navigation. | Configurable. |
| **Groq** | A cloud LLM service (`llama-3.3-70b`) used by the app for chat and receipt parsing. | External supporting actor. |
| **LLM (Large Language Model)** | A text model used for the assistant and structured receipt parsing. | Groq-hosted. |
| **Warm-up / Fallback** | A startup priming call / a safe default response when an external call fails or is unparseable. | Keeps the system responsive and robust. |
| **Spatial position** | Coarse human-readable location of a box: left/center/right, top/middle/bottom, near/far. | Grounds the VLM's directions. |

## D. Implementation & modelling terms

| Term | Definition (explanation) | Notes |
|------|--------------------------|-------|
| **Jetpack Compose** | Android's declarative UI toolkit used to build every screen. | Each tab is a `@Composable`. |
| **Composable / Screen** | A UI function that renders part of the app (e.g. `HomeScreen`, `AIScreen`). | Compose building block. |
| **OkHttp / Retrofit** | The HTTP client / typed REST layer the app uses to call services. | Backs `NavigatorApi`, `GroqApiService`. |
| **`NavigatorApi`** | The app's client object for the UniGoal backend (`startSession`, `uploadPhoto`, `answer`). | Bridges app ↔ backend. |
| **FastAPI** | The Python web framework exposing UniGoal's REST endpoints. | Hosts `/session`, `/photo`, etc. |
| **REST endpoint** | One HTTP operation (method + path) a client can call. | e.g. `POST /session/{id}/photo`. |
| **Pydantic model** | A typed request/response schema in the backend. | e.g. `StartSessionRequest`. |
| **Session store** | The backend registry that creates and looks up sessions by id. | One per running server. |
| **Batch mapper** | The offline tool that detects objects across a folder of photos and builds a map. | Run from the command line. |
| **Jaccard similarity** | A 0–1 measure of overlap between two sets (here, two photos' object sets). | Decides zone boundaries. |
| **Fuzzy matching** | Approximate string matching tolerating typos/partial words. | "fridge" matches "refrigerator". |
| **Knowledge base** | The static description of an environment: rooms, landmarks, professor offices (EN + ZH). | `RoomInfo` records. |
| **`RoomInfo` / `LocationMatch`** | A described room/zone / the ranked result of a location query. | Drive `find_product`. |
| **Actor** | An external participant (person or system) that interacts with the system. | User, Mapping Operator, cloud services. |
| **Use case** | A unit of useful behaviour the system performs for an actor. | Rounded shape in the diagrams. |
| **`«include»` / `«extend»`** | A use case always performed as part of another / optional conditional behaviour. | Stereotyped dashed arrows. |
