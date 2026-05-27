# Chapter 6 — Use Case Descriptions

For each of the eight use case diagrams, this chapter gives **M use case
descriptions (scenarios)**: one **normal** (main success) scenario plus one or
more **exception** scenarios, using a standard template.

---

## Function 1 — Manage Shopping List

### 1.0 Normal scenario — *Add an item and check it off*

| Field | Content |
|-------|---------|
| **Use case** | Manage Shopping List |
| **Primary actor** | User |
| **Precondition** | The app is open on the list (*清單*) tab. |
| **Postcondition** | The item exists in the list; when checked, it counts toward spending. |
| **Trigger** | The user adds a new item. |

**Main success scenario**

1. The user enters an item name (and optionally quantity and price).
2. The system creates a shopping item with a unique id and a creation time.
3. The system auto-categorises the item (e.g. *food*, *daily goods*).
4. The system shows the item in the list.
5. Later, the user checks the item off as purchased.
6. The system marks it purchased and records the purchase time, so it now
   counts toward budget spending.

### 1.E1 Exception — *Empty item name*

1. The user confirms with a blank name.
2. The system rejects the entry and keeps the add dialog open. No item is
   created.

### 1.E2 Exception — *Edit or delete an item*

1. The user selects an existing item and edits a field (or deletes it).
2. The system updates or removes the item and refreshes any dependent totals.

---

## Function 2 — Scan Receipt & Track Budget

### 2.0 Normal scenario — *Receipt becomes budget line items*

| Field | Content |
|-------|---------|
| **Use case** | Scan Receipt & Track Budget |
| **Primary actor** | User |
| **Supporting actors** | PaddleOCR Service, Groq AI Service |
| **Precondition** | The budget (*預算*) tab is open; API keys are configured. |
| **Postcondition** | Parsed line items are added; spending and remaining budget are updated. |
| **Trigger** | The user captures or picks a receipt photo. |

**Main success scenario**

1. The user provides a receipt photo.
2. The system sends the image to the OCR service and receives the raw text.
3. The system sends the text to the LLM with a parsing prompt.
4. The LLM returns structured line items (name, price, quantity).
5. The system adds the line items and recomputes total spent, remaining budget,
   and spending by category.
6. The system shows the updated budget summary.

### 2.E1 Exception — *OCR finds no usable text*

1. At step 2 the OCR returns empty/garbled text.
2. The system reports that the receipt could not be read and adds nothing.

### 2.E2 Exception — *LLM parsing fails or returns invalid JSON*

1. At step 4 the LLM call errors or returns unparseable output.
2. The system reports a parsing failure; no line items are added.

### 2.E3 Exception — *Set monthly budget*

1. The user opens the budget setting and enters a new monthly limit.
2. The system saves it and recomputes the remaining amount and utilisation.

---

## Function 3 — Log Diet & Nutrition

### 3.0 Normal scenario — *Scan a nutrition label*

| Field | Content |
|-------|---------|
| **Use case** | Log Diet & Nutrition |
| **Primary actor** | User |
| **Supporting actor** | PaddleOCR Service |
| **Precondition** | The diet-analysis (*分析*) tab is open. |
| **Postcondition** | A diet record is stored; today's calorie/macro totals are updated. |
| **Trigger** | The user scans a nutrition label. |

**Main success scenario**

1. The user captures a nutrition-label photo.
2. The system sends it to the OCR service and receives the text.
3. The system extracts nutrition values (calories, carbs, protein, fat, …).
4. The system saves a diet record for the current date.
5. The system updates today's total calories and per-category counts.

### 3.1 Alternate scenario — *Manual entry*

1. The user chooses manual entry instead of scanning.
2. The user types the food name and nutrition values.
3. The scenario continues from step 4.

### 3.E1 Exception — *Unreadable label*

1. At step 2 OCR yields no usable values.
2. The system asks the user to retake the photo or enter the values manually;
   no record is saved automatically.

---

## Function 4 — Consult AI Shopping Assistant

### 4.0 Normal scenario — *Cross-module question answered*

| Field | Content |
|-------|---------|
| **Use case** | Consult AI Shopping Assistant |
| **Primary actor** | User |
| **Supporting actor** | Groq AI Service |
| **Precondition** | The assistant (*助理*) tab is open; the API key is configured. |
| **Postcondition** | The user sees an answer grounded in their data. |
| **Trigger** | The user sends a chat message. |

**Main success scenario**

1. The user types a question (e.g. "How much did I spend on snacks?").
2. The system builds a shopping-context snapshot (inventory + budget + health).
3. The system sends the system prompt, the recent chat history, the context, and
   the question to the LLM.
4. The LLM returns an answer.
5. The system appends the answer to the conversation.

### 4.E1 Exception — *LLM unavailable*

1. At step 3 the LLM call errors or times out.
2. The system shows an error message and keeps the conversation intact so the
   user can retry.

### 4.E2 Exception — *Empty message*

1. The user sends an empty message.
2. The system ignores it and does not call the LLM.

---

## Function 5 — Find Nearby Stores

### 5.0 Normal scenario — *List nearby stores*

| Field | Content |
|-------|---------|
| **Use case** | Find Nearby Stores |
| **Primary actor** | User |
| **Supporting actor** | Google Places Service |
| **Precondition** | Location permission is granted; the maps key is configured. |
| **Postcondition** | The user sees a ranked list of nearby stores. |
| **Trigger** | The user opens the nearby-stores view. |

**Main success scenario**

1. The system reads the current GPS location.
2. The user picks a store category (or uses the default).
3. The system queries the maps service for nearby places of that category.
4. The system displays the results (name, distance, rating).
5. The user taps a store to open directions in the maps app.

### 5.1 Alternate scenario — *AR overlay*

1. From the list the user opens the AR view.
2. The system shows a camera overlay pointing toward the stores.

### 5.E1 Exception — *Location permission denied*

1. At step 1 location permission is missing.
2. The system explains why it is needed and offers to open settings; no search
   runs.

### 5.E2 Exception — *No stores found*

1. At step 3 the service returns no results.
2. The system shows an empty state suggesting a wider radius or another
   category.

---

## Function 6 — In-Store Visual Navigation

### 6.0 Normal scenario — *Photo turn produces a MOVE instruction*

| Field | Content |
|-------|---------|
| **Use case** | In-Store Visual Navigation |
| **Primary actor** | User |
| **Supporting actors** | UniGoal Backend, Ollama VLM Service |
| **Precondition** | The backend is running; the app can reach it. |
| **Postcondition** | A session exists; the user receives turn-by-turn guidance until arrival. |
| **Trigger** | The user starts navigation with a goal. |

**Main success scenario**

1. The user enters a goal (e.g. "find the milk").
2. The app calls the backend to start a session; the backend decomposes the goal
   into goal objects and returns a session id.
3. The user captures a photo; the app uploads it.
4. The backend detects goal objects and reads signs with OCR.
5. The backend adds a map node (and an edge from the previous node).
6. The backend asks the VLM for the next action, passing detections, OCR, and
   the path summary.
7. The VLM returns `MOVE` with guidance; the arrival gate leaves it unchanged.
8. The backend annotates the photo and returns `MOVE`, the guidance, and the
   annotated-photo URL.
9. The app shows the guidance; the user walks and repeats from step 3 until
   `ARRIVED`.

### 6.1 Alternate scenario — *Answer a clarification question*

1. At step 7 the action is `ASK` with a question.
2. The user submits an answer; the app calls the backend answer endpoint.
3. The backend re-runs the decision with the question + answer and returns a new
   action.

### 6.2 Alternate scenario — *Confirmed arrival*

1. The VLM returns `ARRIVED` and the arrival gate finds a goal object detected
   above threshold (or a matching sign).
2. The backend marks the session arrived and sets the goal node; the app shows
   the success state.

### 6.E1 Exception — *Empty goal*

1. The user starts with a blank goal.
2. The backend rejects it (`400 bad_request`); no session is created.

### 6.E2 Exception — *Backend or VLM unreachable / unparseable*

1. A backend call fails, times out, or the VLM returns unparseable output (even
   after one retry).
2. The backend returns a safe fallback `MOVE` ("walk forward and upload another
   photo"); the app surfaces a connection error if the call itself failed.

### 6.E3 Exception — *Arrival claimed without evidence (arrival gate)*

1. The VLM returns `ARRIVED` but no goal object/sign evidence exists.
2. The backend downgrades the action to `ASK` and asks the user to confirm they
   can see the target.

### 6.E4 Exception — *Out-of-order requests*

1. The user uploads a photo while a question is pending, or after arrival.
2. The backend rejects it (`409 answer_pending` or `409 already_arrived`).

---

## Function 7 — Build Environment Map (offline)

### 7.0 Normal scenario — *Build a map from a photo folder*

| Field | Content |
|-------|---------|
| **Use case** | Build Environment Map |
| **Primary actor** | Mapping Operator |
| **Precondition** | A folder of survey photos exists; detector weights are installed. |
| **Postcondition** | A detections JSON and a topological map image are written. |
| **Trigger** | The operator runs the batch mapper with an input folder and a goal. |

**Main success scenario**

1. The operator runs the batch mapper with the folder and goal.
2. The system builds detection classes from the goal plus common indoor objects.
3. The system loads the detector and detects objects in every photo
   (deduplicating case-variant filenames).
4. The system writes all detections to the output JSON.
5. The system clusters photos into zones by Jaccard similarity.
6. The system builds graph edges from objects shared across photos.
7. The system renders the topological map to PNG (standard + hi-res).
8. The system reports zones, objects, and output paths.

### 7.E1 Exception — *Empty input folder*

1. The folder has no JPG/PNG files.
2. The system reports "no photos found" and exits without writing a map.

### 7.E2 Exception — *Detector weights missing*

1. The detector weights/config cannot be located.
2. Model loading fails; the system reports the missing path and aborts.

### 7.E3 Exception — *Map generation skipped*

1. The operator omits the `--map` flag.
2. The system writes the detections JSON but skips clustering and rendering.

---

## Function 8 — Query Product Location on Map

### 8.0 Normal scenario — *Find a product and get directions*

| Field | Content |
|-------|---------|
| **Use case** | Query Product Location |
| **Primary actor** | User / Operator |
| **Precondition** | A pre-built map and knowledge base exist. |
| **Postcondition** | The location and ordered walking directions are returned. |
| **Trigger** | A location query is submitted. |

**Main success scenario**

1. The query (English or Chinese, e.g. "fridge" / "冰箱") is submitted.
2. The system fuzzy-matches it against room names, categories, landmarks, and
   professor offices, ranking candidates by confidence.
3. The system takes the best match and reads its target node.
4. The system computes the shortest path from the start node to the target.
5. The system turns each edge into a direction string and appends an arrival
   line ("You've arrived at … — look for …").
6. The system returns the matched products, location (EN/ZH), directions, zone,
   and confidence.

### 8.E1 Exception — *No match found*

1. Step 2 yields no candidate above the matching threshold.
2. The system returns nothing; the caller tells the user nothing matched.

### 8.E2 Exception — *Match has no mapped node*

1. The best match has no node id.
2. The system returns nothing; directions cannot be computed.

### 8.E3 Exception — *No path to target*

1. Step 4 finds no route between start and target.
2. The system catches the no-path condition and returns nothing.
