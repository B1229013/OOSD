# Chapter 7 — Activity Diagrams

For each use case diagram (and its descriptions), this chapter gives one
complete activity diagram. Read top-to-bottom: a start node, rectangular
actions, diamond decisions with guard conditions, and end nodes. The exception
scenarios from Chapter 6 appear as branches off the decisions.

---

## Function 1 — Manage Shopping List

```mermaid
flowchart TD
    Start(["Start"]) --> A["User enters item name, qty, price"]
    A --> D1{"Name non-empty?"}
    D1 -- "no" --> E1["Reject, keep dialog open"] --> A
    D1 -- "yes" --> B["Create item with id and timestamp"]
    B --> C["Auto-categorise item"]
    C --> F["Show item in list"]
    F --> D2{"User action?"}
    D2 -- "check off" --> G["Mark purchased, record time"]
    D2 -- "edit/delete" --> H["Update or remove item"]
    D2 -- "none" --> EndIdle(["End"])
    G --> I["Recompute budget spending"]
    H --> I
    I --> EndDone(["End"])
```

---

## Function 2 — Scan Receipt & Track Budget

```mermaid
flowchart TD
    Start(["Start"]) --> A["Capture or pick receipt photo"]
    A --> B["Send image to OCR service"]
    B --> D1{"Usable text returned?"}
    D1 -- "no" --> E1["Report 'could not read receipt'"] --> EndE(["End"])
    D1 -- "yes" --> C["Send text to LLM with parse prompt"]
    C --> D2{"Valid line items parsed?"}
    D2 -- "no" --> E2["Report parsing failure"] --> EndE
    D2 -- "yes" --> F["Add line items"]
    F --> G["Recompute total spent, remaining, by category"]
    G --> H["Show updated budget summary"]
    H --> EndOk(["End"])
```

---

## Function 3 — Log Diet & Nutrition

```mermaid
flowchart TD
    Start(["Start"]) --> D0{"Scan or manual?"}
    D0 -- "scan" --> A["Capture nutrition-label photo"]
    A --> B["Send to OCR service"]
    B --> D1{"Values readable?"}
    D1 -- "no" --> E1["Ask to retake or enter manually"] --> EndE(["End"])
    D1 -- "yes" --> C["Extract nutrition values"]
    D0 -- "manual" --> M["User types food + values"]
    C --> S["Save diet record for today"]
    M --> S
    S --> T["Update daily calories and categories"]
    T --> EndOk(["End"])
```

---

## Function 4 — Consult AI Shopping Assistant

```mermaid
flowchart TD
    Start(["Start"]) --> A["User sends a message"]
    A --> D1{"Message non-empty?"}
    D1 -- "no" --> E1["Ignore"] --> EndE(["End"])
    D1 -- "yes" --> B["Build shopping-context snapshot"]
    B --> C["Send prompt + history + context to LLM"]
    C --> D2{"LLM responded?"}
    D2 -- "no" --> E2["Show error, keep conversation"] --> EndE
    D2 -- "yes" --> F["Append answer to conversation"]
    F --> EndOk(["End"])
```

---

## Function 5 — Find Nearby Stores

```mermaid
flowchart TD
    Start(["Start"]) --> D1{"Location permission granted?"}
    D1 -- "no" --> E1["Explain need, offer settings"] --> EndE(["End"])
    D1 -- "yes" --> A["Read current GPS location"]
    A --> B["User picks store category"]
    B --> C["Query maps service nearby"]
    C --> D2{"Any results?"}
    D2 -- "no" --> E2["Show empty state"] --> EndE
    D2 -- "yes" --> F["Display stores (name, distance, rating)"]
    F --> D3{"User choice?"}
    D3 -- "open directions" --> G["Launch maps app"]
    D3 -- "AR view" --> H["Show AR overlay"]
    D3 -- "none" --> EndIdle(["End"])
    G --> EndOk(["End"])
    H --> EndOk
```

---

## Function 6 — In-Store Visual Navigation

```mermaid
flowchart TD
    Start(["Start"]) --> A["User enters goal"]
    A --> D0{"Goal non-empty?"}
    D0 -- "no" --> E0["Backend 400 bad_request"] --> EndE(["End"])
    D0 -- "yes" --> B["Start session, decompose goal"]
    B --> C["Capture and upload photo"]
    C --> G1{"Request allowed?"}
    G1 -- "no" --> E1["409 answer_pending / already_arrived"] --> EndE
    G1 -- "yes" --> D["Detect objects + read signs (OCR)"]
    D --> E["Add map node and edge"]
    E --> F["Ask VLM for next action"]
    F --> G2{"Response parseable?"}
    G2 -- "no" --> Fb["Use fallback MOVE"]
    G2 -- "yes" --> H["Take VLM action"]
    Fb --> I["Arrival gate"]
    H --> I
    I --> J{"Action?"}
    J -- "MOVE" --> K["Show guidance"] --> C
    J -- "ASK" --> Q["Show question"]
    Q --> R["User answers"]
    R --> F
    J -- "ARRIVED" --> L{"Goal object or sign evidence?"}
    L -- "no" --> N["Downgrade to ASK (confirm)"] --> Q
    L -- "yes" --> M["Mark arrived, set goal node"]
    M --> EndOk(["End — arrived"])
```

---

## Function 7 — Build Environment Map (offline)

```mermaid
flowchart TD
    Start(["Start"]) --> A["Read goal and input folder"]
    A --> B["Build detection classes"]
    B --> D1{"Folder has photos?"}
    D1 -- "no" --> E1["Report 'no photos found', exit"] --> EndE(["End"])
    D1 -- "yes" --> C["Load detector"]
    C --> D2{"Weights found?"}
    D2 -- "no" --> E2["Report missing path, abort"] --> EndE
    D2 -- "yes" --> E["Detect objects in every photo"]
    E --> F["Write detections JSON"]
    F --> D3{"--map flag set?"}
    D3 -- "no" --> H["Skip map generation"] --> EndDone(["End"])
    D3 -- "yes" --> I["Cluster photos into zones (Jaccard)"]
    I --> J["Build edges from shared objects"]
    J --> K["Render map PNG (standard + hi-res)"]
    K --> L["Report zones, objects, paths"]
    L --> EndDone
```

---

## Function 8 — Query Product Location on Map

```mermaid
flowchart TD
    Start(["Start"]) --> A["Receive query and start node"]
    A --> B["Fuzzy-match against knowledge base"]
    B --> D1{"Any match found?"}
    D1 -- "no" --> E1["Return nothing"] --> EndE(["End"])
    D1 -- "yes" --> C["Take best match"]
    C --> D2{"Match has a mapped node?"}
    D2 -- "no" --> E2["Return nothing"] --> EndE
    D2 -- "yes" --> E["Compute shortest path"]
    E --> D3{"Path exists?"}
    D3 -- "no" --> E3["Return nothing"] --> EndE
    D3 -- "yes" --> F["Convert edges to direction steps"]
    F --> G["Append arrival line"]
    G --> H["Return location, directions, confidence"]
    H --> EndOk(["End"])
```
