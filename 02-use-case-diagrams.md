# Chapter 5 — Use Case Diagrams

The information system provides **eight functions**. Each is presented below as
its own use case diagram. A labelled box is the **system boundary**, rounded
shapes are **use cases**, and lines connect **actors** to the use cases they
take part in. Dashed arrows carry the `«include»` and `«extend»` stereotypes.

**Legend**

```mermaid
flowchart LR
    A["👤 Actor"]
    UC(["Use Case"])
    A --- UC
    UC -. "«include»" .-> UC2(["Always-performed sub use case"])
    UC -. "«extend»" .-> UC3(["Optional extending use case"])
```

---

## Function 1 — Manage Shopping List

The user builds and maintains a shopping list: add, edit, delete, and check off
items. Items are auto-categorised, and checking an item feeds the budget.

```mermaid
flowchart LR
    User["👤 User"]

    subgraph S1["Shopping App — List Management"]
        UC1(["Manage Shopping List"])
        UC1a(["Add Item"])
        UC1b(["Edit / Delete Item"])
        UC1c(["Check Off Item as Purchased"])
        UC1d(["Auto-Categorise Item"])
    end

    User --- UC1
    UC1 -. "«include»" .-> UC1a
    UC1 -. "«include»" .-> UC1b
    UC1 -. "«include»" .-> UC1c
    UC1a -. "«include»" .-> UC1d
```

---

## Function 2 — Scan Receipt & Track Budget

The user photographs a receipt; the app reads it with cloud OCR, asks the LLM to
parse it into line items, adds them to the budget, and shows spending vs. the
monthly budget by category.

```mermaid
flowchart LR
    User["👤 User"]
    OCRSvc["🌐 PaddleOCR Service"]
    Groq["🤖 Groq AI Service"]

    subgraph S2["Shopping App — Budget"]
        UC2(["Scan Receipt & Track Budget"])
        UC2a(["Capture Receipt Photo"])
        UC2b(["Recognise Receipt Text"])
        UC2c(["Parse Text into Line Items"])
        UC2d(["Update Budget & Spending"])
        UC2e(["Set Monthly Budget"])
    end

    User --- UC2
    UC2 -. "«include»" .-> UC2a
    UC2 -. "«include»" .-> UC2b
    UC2 -. "«include»" .-> UC2c
    UC2 -. "«include»" .-> UC2d
    UC2 -. "«extend»" .-> UC2e
    UC2b --- OCRSvc
    UC2c --- Groq
```

---

## Function 3 — Log Diet & Nutrition

The user scans a food/nutrition label (or enters data manually); the app reads
the values, stores a diet record, and totals calories and macros for the day.

```mermaid
flowchart LR
    User["👤 User"]
    OCRSvc["🌐 PaddleOCR Service"]

    subgraph S3["Shopping App — Diet Analysis"]
        UC3(["Log Diet & Nutrition"])
        UC3a(["Scan Nutrition Label"])
        UC3b(["Recognise Label Text"])
        UC3c(["Enter Record Manually"])
        UC3d(["Save Diet Record"])
        UC3e(["Summarise Daily Calories"])
    end

    User --- UC3
    UC3 -. "«include»" .-> UC3d
    UC3 -. "«include»" .-> UC3e
    UC3 -. "«extend»" .-> UC3a
    UC3 -. "«extend»" .-> UC3c
    UC3a -. "«include»" .-> UC3b
    UC3b --- OCRSvc
```

---

## Function 4 — Consult AI Shopping Assistant

The user chats with the assistant. Each question is sent to the LLM together
with a snapshot of the shopping context (inventory + budget + health), so the AI
can answer cross-module questions.

```mermaid
flowchart LR
    User["👤 User"]
    Groq["🤖 Groq AI Service"]

    subgraph S4["Shopping App — AI Assistant"]
        UC4(["Consult AI Shopping Assistant"])
        UC4a(["Build Shopping Context Snapshot"])
        UC4b(["Send Question to LLM"])
        UC4c(["Show Answer"])
    end

    User --- UC4
    UC4 -. "«include»" .-> UC4a
    UC4 -. "«include»" .-> UC4b
    UC4 -. "«include»" .-> UC4c
    UC4b --- Groq
```

---

## Function 5 — Find Nearby Stores

The user opens the nearby-stores view; the app reads GPS, queries the maps
service for nearby shops of a chosen category, lists them, and can open
directions or an AR overlay.

```mermaid
flowchart LR
    User["👤 User"]
    Places["🌐 Google Places Service"]

    subgraph S5["Shopping App — Nearby Stores"]
        UC5(["Find Nearby Stores"])
        UC5a(["Get Current Location"])
        UC5b(["Search Nearby by Category"])
        UC5c(["Open Directions in Maps"])
        UC5d(["Show AR Overlay"])
    end

    User --- UC5
    UC5 -. "«include»" .-> UC5a
    UC5 -. "«include»" .-> UC5b
    UC5 -. "«extend»" .-> UC5c
    UC5 -. "«extend»" .-> UC5d
    UC5b --- Places
```

---

## Function 6 — In-Store Visual Navigation

The core integration. The user states a goal, then walks and uploads photos; the
backend detects objects, reads signs, asks the VLM for the next action, gates
any arrival claim, and replies `MOVE` / `ASK` / `ARRIVED`. The user can answer
questions and view the map.

```mermaid
flowchart LR
    User["👤 User"]
    Backend["🌐 UniGoal Backend"]
    VLM["🤖 Ollama VLM Service"]

    subgraph S6["Shopping App + UniGoal — Navigation"]
        UC6(["In-Store Visual Navigation"])
        UC6a(["Start Navigation Session"])
        UC6g(["Decompose Goal"])
        UC6b(["Capture & Upload Photo"])
        UC6c(["Detect Objects"])
        UC6d(["Read Signs with OCR"])
        UC6e(["Decide Next Action"])
        UC6f(["Verify Arrival"])
        UC6h(["Answer Clarification Question"])
        UC6i(["View Topological Map"])
        UC6j(["Annotate Photo"])
    end

    User --- UC6
    UC6 -. "«include»" .-> UC6a
    UC6 -. "«include»" .-> UC6b
    UC6 -. "«extend»" .-> UC6h
    UC6 -. "«extend»" .-> UC6i
    UC6a -. "«include»" .-> UC6g
    UC6b -. "«include»" .-> UC6c
    UC6b -. "«include»" .-> UC6d
    UC6b -. "«include»" .-> UC6e
    UC6b -. "«include»" .-> UC6f
    UC6b -. "«extend»" .-> UC6j
    UC6a --- Backend
    UC6b --- Backend
    UC6e --- VLM
    UC6g --- VLM
```

---

## Function 7 — Build Environment Map (offline)

A mapping operator points the batch mapper at a folder of survey photos; the
system detects objects in every photo, clusters them into zones, builds the
graph, and renders a topological map image.

```mermaid
flowchart LR
    Operator["👤 Mapping Operator"]

    subgraph S7["UniGoal Backend — Batch Mapping"]
        UC7(["Build Environment Map"])
        UC7a(["Detect Objects in Each Photo"])
        UC7b(["Cluster Photos into Zones"])
        UC7c(["Build Graph Edges from Shared Objects"])
        UC7d(["Save Detections JSON"])
        UC7e(["Render Map Image"])
    end

    Operator --- UC7
    UC7 -. "«include»" .-> UC7a
    UC7 -. "«include»" .-> UC7b
    UC7 -. "«include»" .-> UC7c
    UC7 -. "«include»" .-> UC7d
    UC7 -. "«extend»" .-> UC7e
```

---

## Function 8 — Query Product Location on Map

On a pre-built map, the user (or operator) asks where a product, room, or
professor's office is. The system fuzzy-matches the query against the knowledge
base and returns step-by-step directions via shortest path.

```mermaid
flowchart LR
    User["👤 User"]

    subgraph S8["UniGoal Backend — Location Query"]
        UC8(["Query Product Location"])
        UC8a(["Fuzzy-Match Query to Knowledge Base"])
        UC8b(["Compute Shortest Path"])
        UC8c(["Build Step-by-Step Directions"])
    end

    User --- UC8
    UC8 -. "«include»" .-> UC8a
    UC8 -. "«include»" .-> UC8b
    UC8 -. "«include»" .-> UC8c
```
