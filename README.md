# Activity Diagrams — Vision Recognition & Diet Analysis Modules

This document provides detailed activity diagrams for the key use cases in both modules. These diagrams show the flow of activities, decision points, and system interactions.

---

## Table of Contents

1. [Vision Recognition Module](#vision-recognition-module)
   - [AD1: Start Navigation Session](#ad1-start-navigation-session)
   - [AD2: Capture & Upload Photo](#ad2-capture--upload-photo)
   - [AD3: Build Environment Map](#ad3-build-environment-map)
   - [AD4: Query Product Location](#ad4-query-product-location)

2. [Diet Analysis Module](#diet-analysis-module)
   - [AD5: Log Diet & Nutrition](#ad5-log-diet--nutrition)
   - [AD6: Scan Nutrition Label](#ad6-scan-nutrition-label)

3. [Legend](#legend)

---

## Vision Recognition Module

### AD1: Start Navigation Session

**Purpose**: User states a product goal; system decomposes it into navigation landmarks

```
Mermaid Activity Diagram Specification:

Start
↓
User enters product goal (e.g., "Find milk")
↓
System creates navigation session
↓
System records goal in current context
↓
System + Current Map Context → Ollama VLM
↓
[Decision: Goal found in knowledge base?]
├─ YES → VLM decomposes goal into landmarks (e.g., "Enter store → Find aisle → Locate shelf")
│   ↓
│   System displays decomposed plan
│   ↓
│   System prompts user to start capturing photos
│   ↓
│   Navigation Session Created ✓
│   ↓
│   End
│
└─ NO → System informs user
    ↓
    System suggests alternatives or manual search
    ↓
    [Decision: User accepts alternative?]
    ├─ YES → Restart with new goal
    │
    └─ NO → Navigation Session NOT created
        ↓
        End
```

**Swimlanes**: User, System, Ollama VLM Service

---

### AD2: Capture & Upload Photo

**Purpose**: User captures photo; system processes it and returns navigation instruction (MOVE/ASK/ARRIVED)

```
Mermaid Activity Diagram Specification:

Start
↓
[Active Navigation Session?]
├─ NO → Prompt user to start session
│   ↓
│   End
│
└─ YES → User taps camera button
    ↓
    User captures photo of surroundings
    ↓
    System uploads photo to backend
    ↓
    ┌─────────────────────────────┐
    │ Parallel Processing:        │
    │ 1. Send to GroundingDINO    │
    │ 2. Extract OCR text         │
    └─────────────────────────────┘
    ↓
    GroundingDINO detects objects (doors, aisles, shelves, signs, plants)
    ↓
    System extracts visible text using OCR
    ↓
    [Decision: Objects detected & text extracted?]
    ├─ NO (Blurry/unrecognizable) → System asks user to retake photo
    │   ↓
    │   [Decision: User retakes?]
    │   ├─ YES → Back to camera capture
    │   └─ NO → End without node creation
    │
    └─ YES → Prepare inference payload
        ↓
        Payload: Original image + detections + OCR + goal + history
        ↓
        System sends to Ollama VLM
        ↓
        VLM processes and reasons about scene
        ↓
        [Decision: VLM output valid?]
        ├─ NO (Unparseable/Ambiguous) → System asks user to describe scene
        │   ↓
        │   User provides description
        │   ↓
        │   System sends description + image to VLM again
        │   ↓
        │   VLM re-reasons
        │   ↓
        │   Proceed to response handling
        │
        └─ YES → Proceed to response handling
            ↓
            [Decision: VLM Response Type]
            ├─ MOVE {direction} → Display navigation direction
            │   ↓
            │   Create topological node with: image ID, detections, VLM response, navigation state
            │   ↓
            │   Update map: add edge between previous node and new node
            │   ↓
            │   End
            │
            ├─ ASK {question} → Display clarification question to user
            │   ↓
            │   Create node (intermediate state)
            │   ↓
            │   [Awaiting user response]
            │   ↓
            │   End
            │
            └─ ARRIVED {landmark} → Verify user has reached target
                ↓
                Display success message with landmark
                ↓
                Create final topological node
                ↓
                Update map
                ↓
                End
```

**Swimlanes**: User, System, GroundingDINO Service, Ollama VLM Service, Database

---

### AD3: Build Environment Map

**Purpose**: Offline batch process to build a topological map from survey photos

```
Mermaid Activity Diagram Specification:

Start
↓
Mapping Operator provides folder of survey photos
↓
System initializes batch mapper
↓
System reads list of photos from folder
↓
┌──────────────────────────────┐
│ For Each Photo:              │
├──────────────────────────────┤
│
│ ↓ Send photo to GroundingDINO
│ ↓ GroundingDINO detects objects & landmarks
│ ↓ Store detections in memory (object ID, location, confidence)
│
│ [Decision: All photos processed?]
│ └─ NO → Next photo (loop)
│    YES → Proceed to clustering
│
└──────────────────────────────┘
↓
[Phase: Clustering]
↓
System groups photos into zones based on shared objects
├─ Photos with "Frozen Foods" sign → Zone 1 (Frozen Aisle)
├─ Photos with "Produce" sign → Zone 2 (Produce Section)
├─ Photos with "Checkout" sign → Zone 3 (Checkout Area)
└─ [Repeated for all zones]
↓
[Decision: Clustering consistent?]
├─ NO (Gaps or noise) → Flag photos; suggest retakes
│   ↓
│   System pauses batch run
│   ↓
│   Operator reviews and adds more photos
│   ↓
│   Resume from clustering
│
└─ YES → Proceed to edge building
    ↓
    [Phase: Build Graph Edges]
    ↓
    ┌──────────────────────────────┐
    │ For Each Zone Pair:          │
    ├──────────────────────────────┤
    │
    │ [Decision: Zones share objects?]
    │ ├─ YES → Create edge (Zone A ↔ Zone B)
    │ └─ NO → No connection
    │
    └──────────────────────────────┘
    ↓
    System saves all detections, clusters, zones to JSON file
    ↓
    System renders topological map image:
    ├─ Nodes = Zones
    ├─ Edges = Adjacent zones
    └─ Labels = Zone names
    ↓
    Map image saved to filesystem
    ↓
    Operator reviews rendered map
    ↓
    [Decision: Map acceptable?]
    ├─ NO → Operator can manually annotate/adjust
    │   ↓
    │   System updates map
    │   ↓
    │   Save finalized version
    │
    └─ YES → Map approved
        ↓
        End
```

**Swimlanes**: Mapping Operator, System, GroundingDINO Service, File Storage, Database

---

### AD4: Query Product Location

**Purpose**: User searches for product; system returns directions on pre-built map

```
Mermaid Activity Diagram Specification:

Start
↓
User enters product query (e.g., "milk", "SKU-12345")
↓
[Decision: Store map loaded?]
├─ NO → System loads offline map from database
│   ↓
│   Proceed to fuzzy matching
│
└─ YES → Proceed to fuzzy matching
    ↓
    System fuzzy-matches query against knowledge base
    ↓
    [Decision: Product found?]
    ├─ NO → System suggests similar products or shows product list
    │   ↓
    │   [Decision: User refines search?]
    │   ├─ YES → New query (restart from fuzzy matching)
    │   └─ NO → End without directions
    │
    └─ YES → Identify zone(s) where product is located
        ↓
        System computes shortest path:
        ├─ Start point: Current location (or store entrance)
        ├─ End point: Target zone
        └─ Use graph edges to find shortest route
        ↓
        [Decision: Path found?]
        ├─ NO → Product unreachable / path error
        │   ↓
        │   System notifies user
        │   ↓
        │   End
        │
        └─ YES → Build step-by-step directions
            ├─ Step 1: Enter store → Aisle entrance
            ├─ Step 2: Walk to frozen section
            ├─ Step 3: Find milk shelf
            └─ [More steps as needed]
            ↓
            System displays directions to user
            ↓
            [Decision: User wants visual navigation?]
            ├─ YES → Start visual navigation session (AD1 + AD2)
            │   ↓
            │   End
            │
            └─ NO → User follows text directions
                ↓
                End
```

**Swimlanes**: User, System, Database, Knowledge Base

---

## Diet Analysis Module

### AD5: Log Diet & Nutrition

**Purpose**: Main use case for recording food intake (can use scanning OR manual entry)

```
Mermaid Activity Diagram Specification:

Start
↓
User opens Diet Analysis tab
↓
System displays diet logging interface
↓
[Decision: How to enter?]
├─ SCAN → Proceed to AD6 (Scan Nutrition Label)
│
└─ MANUAL → Proceed to manual entry
    ↓
    System displays form:
    ├─ Food name input field
    └─ Nutrition values (calories, carbs, protein, fat)
    ↓
    User enters food name (e.g., "Apple")
    ↓
    User enters nutrition values:
    ├─ Calories: 95 kcal
    ├─ Carbs: 25g
    ├─ Protein: 0.5g
    └─ Fat: 0.3g
    ↓
    User submits form
    ↓
    [Decision: Input valid?]
    ├─ NO (Empty fields / negative values) → System shows error
    │   ↓
    │   [Decision: User corrects?]
    │   ├─ YES → Back to form entry
    │   └─ NO → End without saving
    │
    └─ YES → Proceed to save
        ↓
        [Phase: Save Diet Record]
        ↓
        System creates record:
        ├─ Food name: "Apple"
        ├─ Nutrition values
        ├─ Date/Time: Current timestamp
        └─ User ID
        ↓
        System stores record in local database
        ↓
        [Phase: Summarise Daily Calories]
        ↓
        System queries all records for current date
        ↓
        System calculates totals:
        ├─ Total calories: 2150 kcal
        ├─ Total carbs: 285g
        ├─ Total protein: 95g
        └─ Total fat: 72g
        ↓
        [Decision: Daily goal exists?]
        ├─ YES → Calculate remaining and % consumed
        │   ├─ Daily goal: 2000 kcal
        │   ├─ Consumed: 2150 kcal (107.5%)
        │   └─ Remaining: -150 kcal (EXCEEDED)
        │   ↓
        │   [Decision: Goal exceeded?]
        │   ├─ YES → Send notification to user
        │   │   ↓
        │   │   Proceed to display update
        │   │
        │   └─ NO → Proceed to display update
        │
        └─ NO → Display totals without goals
            ↓
            Proceed to display update
        ↓
        System updates diet summary display:
        ├─ Today's totals (calories, macros)
        ├─ Goal progress (if applicable)
        └─ Recent food entries
        ↓
        User sees updated diet summary
        ↓
        End
```

**Swimlanes**: User, System, Database

---

### AD6: Scan Nutrition Label

**Purpose**: User photographs nutrition label; OCR extracts values and saves record

```
Mermaid Activity Diagram Specification:

Start
↓
User opens Diet Analysis tab
↓
User selects "Scan Nutrition Label"
↓
System opens camera interface
↓
User aligns camera with nutrition label
↓
User captures photo
↓
System sends image to PaddleOCR service
↓
PaddleOCR processes image and recognizes text
↓
[Decision: Text successfully extracted?]
├─ NO (Label unreadable / unclear) → System asks user to retake
│   ↓
│   System displays: "Photo is unclear. Please retake with better lighting/angle."
│   ↓
│   [Decision: User retakes?]
│   ├─ YES → Back to camera capture
│   └─ NO → End without saving
│
└─ YES → System parses OCR output
    ↓
    System extracts nutrition values:
    ├─ Calories (per serving): 150 kcal
    ├─ Carbohydrates: 20g
    ├─ Protein: 8g
    ├─ Fat: 3g
    └─ [Other nutrients as available]
    ↓
    [Decision: All key values found?]
    ├─ NO (Missing calories or serving) → System asks user for clarification
    │   ↓
    │   System displays: "Serving size unclear. Please enter manually."
    │   ↓
    │   User enters missing values manually
    │   ↓
    │   Proceed to record creation
    │
    └─ YES → Proceed to record creation
        ↓
        System creates diet record:
        ├─ Food name (inferred from label or user input)
        ├─ Extracted nutrition values
        ├─ Date/Time: Current timestamp
        └─ Source: "Scanned label"
        ↓
        System stores record in database
        ↓
        [Phase: Summarise Daily Calories]
        ↓
        (Same as AD5 - recalculate totals and update display)
        ↓
        System updates diet summary
        ↓
        User sees confirmation: "Food logged successfully!"
        ↓
        [Decision: User wants to scan another?]
        ├─ YES → Back to camera interface
        └─ NO → End
```

**Swimlanes**: User, System, PaddleOCR Service, Database

---

## Legend

### Activity Diagram Symbols

| Symbol | Meaning |
|--------|---------|
| ● (Filled circle) | Start activity |
| ○ (Empty circle) | End activity |
| Rectangle | Activity/Action (process step) |
| Diamond | Decision point (YES/NO branch) |
| Arrow → | Flow of control |
| ∥ | Parallel processing (fork/join bar) |
| Rectangle with lines | Swimlane (actor/component) |
| ┌─┐ | Subactivity/Phase group |

### Key Concepts

- **Decision Points**: Branches flow based on conditions (YES/NO)
- **Parallel Processing**: Multiple activities can occur simultaneously (marked with ∥)
- **Swimlanes**: Show which actor/component performs each activity
- **Loops**: Activities can repeat based on conditions
- **Subactivities**: Complex activities can be broken into phases

---

## How to Use These Diagrams

1. **In Visual Paradigm**:
   - Create a new Activity Diagram
   - Add swimlanes for each actor (User, System, Services)
   - Add activities and decision points following the flow above
   - Connect with arrows

2. **In Lucidchart**:
   - Use the activity diagram shapes
   - Organize by swimlanes
   - Add decision branches with diamond shapes

3. **In Draw.io / Miro**:
   - Create swimlane containers
   - Add activity shapes (rectangles) and decision shapes (diamonds)
   - Connect with arrows

4. **In Figma** (with plugins):
   - Use diagram components
   - Follow the flow structure above

---

## Notes

- These activity diagrams represent the **happy paths** and **common exception flows**
- Each diagram can be expanded with more detail as needed
- Decision outcomes are marked with brackets: `[Decision: ...]`
- Parallel activities are shown in boxes with vertical lines: `├─`, `└─`
- For implementation, focus on swimlanes to understand responsibility distribution

---

**Document Version**: 1.0  
**Last Updated**: 2026-06-12  
**For**: OOSD Assignment - Vision Recognition & Diet Analysis Module
