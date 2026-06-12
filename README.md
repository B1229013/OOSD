# 活動圖表 — 視覺語意辨識與飲食分析模組

本文件提供兩個模組主要使用案例的詳細活動圖表。這些圖表顯示活動流程、決策點和系統互動。

---

## 目錄

1. [視覺語意辨識模組](#視覺語意辨識模組)
   - [AD1：開始導航會話](#ad1開始導航會話)
   - [AD2：擷取並上傳照片](#ad2擷取並上傳照片)
   - [AD3：建構環境地圖](#ad3建構環境地圖)
   - [AD4：查詢產品位置](#ad4查詢產品位置)

2. [飲食分析模組](#飲食分析模組)
   - [AD5：記錄飲食與營養](#ad5記錄飲食與營養)
   - [AD6：掃描營養標籤](#ad6掃描營養標籤)

3. [圖例](#圖例)

---

## 視覺語意辨識模組

### AD1: 開始導航會話

**Purpose**: 使用者陳述產品目標；系統將其分解為導航地標

```mermaid
graph TD
    Start([開始]) --> UserInput["使用者輸入<br/>產品目標<br/>例如：找牛奶"]
    UserInput --> CreateSession["系統建立<br/>導航會話"]
    CreateSession --> RecordGoal["系統記錄<br/>目標到背景"]
    RecordGoal --> PrepPayload["準備VLM<br/>輸入資料<br/>目標+地圖背景"]
    PrepPayload --> CallVLM["發送到<br/>Ollama VLM<br/>進行分解"]
    
    CallVLM --> DecisionFound{目標是否<br/>在知識庫<br/>中找到?}
    
    DecisionFound -->|是| DecomposeGoal["VLM分解<br/>目標為<br/>地標步驟"]
    DecomposeGoal --> DisplayPlan["系統顯示<br/>分解後的<br/>計劃"]
    DisplayPlan --> PromptPhotos["提示使用者<br/>開始<br/>拍攝照片"]
    PromptPhotos --> SessionCreated["導航會話<br/>成功建立"]
    SessionCreated --> EndSuccess([結束 - 成功])
    
    DecisionFound -->|否| InformUser["系統通知<br/>使用者<br/>目標未找到"]
    InformUser --> SuggestAlternatives["建議<br/>替代方案<br/>或手動搜尋"]
    SuggestAlternatives --> DecisionAccept{使用者<br/>接受<br/>替代方案?}
    
    DecisionAccept -->|是| UserInput
    DecisionAccept -->|否| SessionFail["導航會話<br/>未建立"]
    SessionFail --> EndFail([結束 - 失敗])
    
    style Start fill:#90EE90
    style EndSuccess fill:#90EE90
    style EndFail fill:#FFB6C6
    style DecisionFound fill:#FFE4B5
    style DecisionAccept fill:#FFE4B5
    style SessionCreated fill:#87CEEB
    style SessionFail fill:#FFB6C6
```

**Swimlanes**: User, System, Ollama VLM Service

---

### AD2: 擷取並上傳照片

**Purpose**: 使用者拍攝照片；系統處理並返回導航指令（移動/詢問/已到達）

```mermaid
graph TD
    Start([開始]) --> CheckSession{存在有效的<br/>導航會話?}
    
    CheckSession -->|否| PromptSession["提示使用者<br/>開始會話"]
    PromptSession --> EndNo([結束 - 無會話])
    
    CheckSession -->|是| CameraButton["使用者點擊<br/>相機按鈕"]
    CameraButton --> CapturePhoto["使用者拍攝<br/>周圍環境<br/>的照片"]
    CapturePhoto --> UploadPhoto["系統上傳<br/>照片到<br/>後端伺服器"]
    
    UploadPhoto --> Fork["開始<br/>平行處理"]
    
    Fork --> Process1["發送到<br/>GroundingDINO<br/>進行物件偵測"]
    Fork --> Process2["使用OCR<br/>提取文字"]
    
    Process1 --> DetectObjects["GroundingDINO<br/>偵測：門、走道、<br/>架子、標示、<br/>植物、物件"]
    Process2 --> ExtractText["系統提取<br/>可見的<br/>文字"]
    
    DetectObjects --> Join["合併<br/>處理結果"]
    ExtractText --> Join
    
    Join --> DecisionQuality{物件偵測<br/>和文字提取<br/>成功?}
    
    DecisionQuality -->|否| BlurryPhoto["照片太模糊<br/>或無信息"]
    BlurryPhoto --> RetakePrompt{使用者<br/>重新<br/>拍攝?}
    RetakePrompt -->|是| CameraButton
    RetakePrompt -->|否| EndBlurry([結束 - 模糊照片])
    
    DecisionQuality -->|是| PrepPayload["準備VLM<br/>輸入資料<br/>照片+偵測結果<br/>+OCR+目標+歷史"]
    PrepPayload --> SendVLM["發送到<br/>Ollama VLM<br/>進行場景推理"]
    SendVLM --> VLMReason["VLM處理<br/>並推理<br/>場景"]
    
    VLMReason --> DecisionVLM{VLM輸出<br/>有效且<br/>可解析?}
    
    DecisionVLM -->|否| VLMError["輸出無法<br/>解析或<br/>模糊"]
    VLMError --> AskDescribe["詢問使用者<br/>描述<br/>所看到的"]
    AskDescribe --> UserDescribe["使用者<br/>提供<br/>描述"]
    UserDescribe --> ResendVLM["發送描述<br/>和照片<br/>到VLM"]
    ResendVLM --> VLMReason
    
    DecisionVLM -->|是| ResponseType{VLM<br/>回應<br/>類型?}
    
    ResponseType -->|移動| HandleMove["VLM返回：<br/>移動方向"]
    HandleMove --> DisplayMove["顯示導航<br/>方向給<br/>使用者"]
    DisplayMove --> CreateNode1["建立拓樸<br/>節點<br/>記錄偵測結果"]
    CreateNode1 --> UpdateMap1["更新地圖：<br/>新增節點<br/>之間的邊"]
    UpdateMap1 --> EndMove([結束 - 移動中])
    
    ResponseType -->|詢問| HandleAsk["VLM返回：<br/>詢問澄清"]
    HandleAsk --> DisplayAsk["顯示問題<br/>給使用者"]
    DisplayAsk --> CreateNode2["建立節點<br/>中間狀態"]
    CreateNode2 --> Waiting["等待使用者<br/>回應"]
    Waiting --> EndAsk([結束 - 等待回應])
    
    ResponseType -->|已到達| HandleArrived["VLM返回：<br/>已到達地標"]
    HandleArrived --> VerifyLandmark["驗證使用者<br/>到達<br/>目標"]
    VerifyLandmark --> DisplaySuccess["顯示成功<br/>訊息與<br/>地標名稱"]
    DisplaySuccess --> CreateNode3["建立最終<br/>節點"]
    CreateNode3 --> UpdateMap2["更新地圖"]
    UpdateMap2 --> EndArrive([結束 - 已到達])
    
    style Start fill:#90EE90
    style EndNo fill:#FFB6C6
    style EndBlurry fill:#FFB6C6
    style EndMove fill:#87CEEB
    style EndAsk fill:#FFD700
    style EndArrive fill:#90EE90
    style Fork fill:#DDA0DD
    style Join fill:#DDA0DD
    style DecisionQuality fill:#FFE4B5
    style DecisionVLM fill:#FFE4B5
    style ResponseType fill:#FFE4B5
    style RetakePrompt fill:#FFE4B5
```

**Swimlanes**: User, System, GroundingDINO Service, Ollama VLM Service, Database

---

### AD3: 建構環境地圖

**Purpose**: 離線批次處理，從調查照片建構拓樸地圖

```mermaid
graph TD
    Start([開始]) --> OperatorInput["地圖操作員<br/>提供調查<br/>照片資料夾"]
    OperatorInput --> InitBatch["系統初始化<br/>批次映射器"]
    InitBatch --> ReadPhotos["系統讀取<br/>照片列表<br/>從資料夾"]
    ReadPhotos --> LoopStart["開始迴圈：<br/>每張照片"]
    
    LoopStart --> SendDINO["發送照片<br/>到<br/>GroundingDINO"]
    SendDINO --> DetectObjects["GroundingDINO<br/>偵測物件<br/>和地標"]
    DetectObjects --> StoreDetections["在記憶體<br/>中儲存<br/>偵測結果"]
    
    StoreDetections --> LoopCheck{所有照片<br/>都已<br/>處理?}
    LoopCheck -->|否| LoopStart
    LoopCheck -->|是| ClusterPhase["階段：聚類"]
    
    ClusterPhase --> GroupZones["系統根據<br/>共享物件<br/>將照片分組為區域"]
    GroupZones --> Zone1["區域1：冷凍走道<br/>共享『冷凍食品』標示"]
    GroupZones --> Zone2["區域2：農產品<br/>共享『農產品』標示"]
    GroupZones --> Zone3["區域3：結帳<br/>共享『結帳』標示"]
    
    Zone1 --> ClusterCheck{聚類<br/>一致?<br/>無間隙/雜訊?}
    Zone2 --> ClusterCheck
    Zone3 --> ClusterCheck
    
    ClusterCheck -->|否| FlagPhotos["標記有<br/>問題的<br/>照片"]
    FlagPhotos --> SuggestRetake["建議操作員<br/>重新<br/>拍攝"]
    SuggestRetake --> PauseRun["暫停<br/>批次<br/>運行"]
    PauseRun --> OperatorReview["操作員<br/>檢視並<br/>新增照片"]
    OperatorReview --> ReadPhotos
    
    ClusterCheck -->|是| EdgePhase["階段：建立圖邊"]
    
    EdgePhase --> EdgeLoop["迴圈：<br/>每對區域"]
    EdgeLoop --> CheckShared{區域<br/>共享<br/>物件?}
    
    CheckShared -->|是| CreateEdge["建立邊<br/>區域A ↔ 區域B<br/>表示相鄰"]
    CreateEdge --> EdgeContinue{還有<br/>更多<br/>區域對?}
    
    CheckShared -->|否| NoEdge["無連接"]
    NoEdge --> EdgeContinue
    
    EdgeContinue -->|是| EdgeLoop
    EdgeContinue -->|否| SaveJSON["保存到<br/>JSON檔案<br/>偵測、聚類、邊"]
    
    SaveJSON --> RenderMap["渲染拓樸<br/>地圖影像<br/>節點=區域"]
    RenderMap --> SaveImage["保存地圖<br/>影像到<br/>檔案系統"]
    SaveImage --> OperatorReview2["操作員<br/>檢視<br/>渲染地圖"]
    
    OperatorReview2 --> MapAccept{地圖<br/>可接受?}
    
    MapAccept -->|否| ManualAnnotate["操作員<br/>手動標注<br/>區域名稱"]
    ManualAnnotate --> UpdateMap["系統<br/>更新<br/>地圖"]
    UpdateMap --> SaveFinal["保存最終<br/>版本"]
    SaveFinal --> EndSuccess([結束 - 地圖完成])
    
    MapAccept -->|是| SaveFinal
    
    style Start fill:#90EE90
    style EndSuccess fill:#90EE90
    style LoopStart fill:#FFD700
    style ClusterPhase fill:#87CEEB
    style EdgePhase fill:#87CEEB
    style EdgeLoop fill:#FFD700
    style LoopCheck fill:#FFE4B5
    style ClusterCheck fill:#FFE4B5
    style CheckShared fill:#FFE4B5
    style MapAccept fill:#FFE4B5
    style EdgeContinue fill:#FFE4B5
```

**Swimlanes**: Mapping Operator, System, GroundingDINO Service, File Storage, Database

---

### AD4: 查詢產品位置

**Purpose**: 使用者搜尋產品；系統在預建地圖上返回方向

```mermaid
graph TD
    Start([開始]) --> UserQuery["使用者輸入<br/>產品查詢<br/>例如：牛奶或SKU-12345"]
    UserQuery --> CheckMap{商店地圖<br/>是否<br/>已加載?}
    
    CheckMap -->|否| LoadMap["系統從<br/>資料庫<br/>加載離線地圖"]
    LoadMap --> FuzzyMatch["系統模糊匹配<br/>查詢與<br/>知識庫"]
    
    CheckMap -->|是| FuzzyMatch
    
    FuzzyMatch --> DecisionFound{產品<br/>是否<br/>找到?}
    
    DecisionFound -->|否| Suggest["系統建議<br/>相似產品或<br/>顯示產品列表"]
    Suggest --> DecisionRefine{使用者<br/>細化<br/>搜尋?}
    
    DecisionRefine -->|是| UserQuery
    DecisionRefine -->|否| EndNoProduct([結束 - 無方向])
    
    DecisionFound -->|是| IdentifyZone["確定產品<br/>所在<br/>的區域"]
    IdentifyZone --> ComputePath["計算最短路徑<br/>開始：目前位置<br/>結束：目標區域"]
    
    ComputePath --> DecisionPath{路徑<br/>是否<br/>找到?}
    
    DecisionPath -->|否| PathError["產品無法<br/>到達或<br/>路徑錯誤"]
    PathError --> NotifyUser["系統通知<br/>使用者<br/>問題"]
    NotifyUser --> EndNoPath([結束 - 路徑未找到])
    
    DecisionPath -->|是| BuildDirections["建立<br/>逐步<br/>方向"]
    BuildDirections --> Step1["步驟1：進入商店<br/>到走道入口"]
    BuildDirections --> Step2["步驟2：走到<br/>冷凍區域"]
    BuildDirections --> Step3["步驟3：找到<br/>牛奶架"]
    
    Step1 --> DisplayDirections["系統顯示<br/>所有方向<br/>給使用者"]
    Step2 --> DisplayDirections
    Step3 --> DisplayDirections
    
    DisplayDirections --> DecisionVisual{使用者<br/>想要<br/>視覺導航?}
    
    DecisionVisual -->|是| StartVisual["開始視覺<br/>導航會話<br/>AD1 + AD2"]
    StartVisual --> EndVisual([結束 - 視覺導航開始])
    
    DecisionVisual -->|否| TextNav["使用者跟隨<br/>文字<br/>方向"]
    TextNav --> EndText([結束 - 文字方向])
    
    style Start fill:#90EE90
    style EndNoProduct fill:#FFB6C6
    style EndNoPath fill:#FFB6C6
    style EndVisual fill:#90EE90
    style EndText fill:#87CEEB
    style CheckMap fill:#FFE4B5
    style DecisionFound fill:#FFE4B5
    style DecisionRefine fill:#FFE4B5
    style DecisionPath fill:#FFE4B5
    style DecisionVisual fill:#FFE4B5
    style ComputePath fill:#DDA0DD
    style BuildDirections fill:#DDA0DD
```

**Swimlanes**: User, System, Database, Knowledge Base

---

## 飲食分析模組

### AD5: 記錄飲食與營養

**Purpose**: 記錄食物攝取的主要使用案例（可使用掃描或手動輸入）

```mermaid
graph TD
    Start([開始]) --> OpenTab["使用者打開<br/>飲食分析<br/>頁籤"]
    OpenTab --> DisplayInterface["系統顯示<br/>飲食記錄<br/>介面"]
    DisplayInterface --> DecisionInput{如何<br/>輸入<br/>資料?}
    
    DecisionInput -->|掃描| ScanPath["進行到AD6<br/>掃描營養<br/>標籤"]
    ScanPath --> SkipManual[ ]
    
    DecisionInput -->|手動| ManualForm["系統顯示<br/>手動輸入<br/>表單"]
    ManualForm --> EnterFood["使用者輸入<br/>食物名稱<br/>例如：蘋果"]
    EnterFood --> EnterNutrition["使用者輸入<br/>營養值<br/>卡路里、碳水、<br/>蛋白質、脂肪"]
    EnterNutrition --> SubmitForm["使用者<br/>提交<br/>表單"]
    
    SubmitForm --> DecisionValid{輸入<br/>有效?<br/>非空名稱、<br/>正數值?}
    
    DecisionValid -->|否| ShowError["系統顯示<br/>錯誤<br/>訊息"]
    ShowError --> DecisionRetry{使用者<br/>修正<br/>輸入?}
    
    DecisionRetry -->|是| EnterFood
    DecisionRetry -->|否| EndNoSave([結束 - 未保存])
    
    DecisionValid -->|是| SavePhase["階段：保存記錄"]
    SavePhase --> CreateRecord["系統建立記錄<br/>食物名稱+營養值<br/>+時間戳記<br/>+使用者ID"]
    CreateRecord --> StoreDB["將記錄存儲<br/>到本地<br/>資料庫"]
    
    SkipManual --> SummarizePhase["階段：<br/>總結每日<br/>卡路里"]
    StoreDB --> SummarizePhase
    
    SummarizePhase --> QueryRecords["系統查詢<br/>目前日期<br/>的所有記錄"]
    QueryRecords --> CalcTotals["計算每日總計<br/>卡路里：2150卡<br/>碳水：285克<br/>蛋白質：95克<br/>脂肪：72克"]
    
    CalcTotals --> DecisionGoal{每日<br/>目標<br/>是否<br/>存在?}
    
    DecisionGoal -->|否| NoGoal["顯示總計<br/>不含<br/>目標"]
    NoGoal --> UpdateDisplay
    
    DecisionGoal -->|是| CalcGoal["計算目標<br/>進度<br/>目標：2000卡<br/>已消耗：2150卡<br/>107.5% 超出"]
    CalcGoal --> DecisionExceeded{目標<br/>是否<br/>超出?}
    
    DecisionExceeded -->|是| SendNotif["發送通知<br/>給使用者"]
    SendNotif --> UpdateDisplay["系統更新<br/>飲食摘要顯示<br/>今日總計、巨量<br/>營養素、進度、<br/>最近的項目"]
    
    DecisionExceeded -->|否| UpdateDisplay
    
    UpdateDisplay --> ShowSummary["使用者看到<br/>更新的<br/>飲食摘要"]
    ShowSummary --> EndSuccess([結束 - 已保存])
    
    style Start fill:#90EE90
    style EndSuccess fill:#90EE90
    style EndNoSave fill:#FFB6C6
    style DecisionInput fill:#FFE4B5
    style DecisionValid fill:#FFE4B5
    style DecisionRetry fill:#FFE4B5
    style DecisionGoal fill:#FFE4B5
    style DecisionExceeded fill:#FFE4B5
    style SavePhase fill:#87CEEB
    style SummarizePhase fill:#87CEEB
```

**Swimlanes**: User, System, Database

---

### AD6: 掃描營養標籤

**Purpose**: 使用者拍攝營養標籤；OCR提取數值並保存記錄

```mermaid
graph TD
    Start([開始]) --> OpenTab["使用者打開<br/>飲食分析<br/>頁籤"]
    OpenTab --> SelectScan["使用者選擇<br/>『掃描營養<br/>標籤』"]
    SelectScan --> OpenCamera["系統打開<br/>相機<br/>介面"]
    OpenCamera --> AlignCamera["使用者將相機<br/>對齊<br/>營養標籤"]
    AlignCamera --> CapturePhoto["使用者拍攝<br/>照片"]
    CapturePhoto --> SendOCR["系統發送<br/>影像到<br/>PaddleOCR服務"]
    
    SendOCR --> OCRProcess["PaddleOCR<br/>處理影像<br/>並識別文字"]
    OCRProcess --> DecisionText{文字是否<br/>成功<br/>提取?}
    
    DecisionText -->|否| Unclear["標籤無法<br/>讀取或<br/>不清楚"]
    Unclear --> RetakePrompt["系統要求<br/>使用者<br/>重新拍攝"]
    RetakePrompt --> DecisionRetake{使用者<br/>重新<br/>拍攝?}
    
    DecisionRetake -->|是| OpenCamera
    DecisionRetake -->|否| EndNoExtract([結束 - 未提取])
    
    DecisionText -->|是| ParseOCR["系統解析<br/>OCR<br/>輸出"]
    ParseOCR --> ExtractValues["系統提取<br/>營養值<br/>卡路里：150卡<br/>碳水：20克<br/>蛋白質：8克<br/>脂肪：3克"]
    
    ExtractValues --> DecisionComplete{是否找到<br/>所有<br/>關鍵<br/>值?}
    
    DecisionComplete -->|否| MissingValues["缺少卡路里<br/>或份量<br/>大小"]
    MissingValues --> AskManual["系統要求<br/>使用者<br/>手動輸入"]
    AskManual --> ManualInput["使用者輸入<br/>缺少的<br/>數值"]
    ManualInput --> CreateRecord
    
    DecisionComplete -->|是| CreateRecord["系統建立<br/>飲食記錄<br/>食物名稱+值<br/>+時間戳記<br/>+來源：掃描"]
    CreateRecord --> StoreDB["將記錄<br/>存儲到<br/>資料庫"]
    
    StoreDB --> SummarizePhase["階段：<br/>總結每日<br/>卡路里"]
    SummarizePhase --> QueryRecords["查詢今天<br/>的所有<br/>記錄"]
    QueryRecords --> CalcTotals["重新計算<br/>總計並<br/>更新顯示"]
    
    CalcTotals --> Confirmation["使用者看到<br/>確認訊息<br/>食物記錄成功"]
    Confirmation --> DecisionAgain{掃描<br/>另一項?}
    
    DecisionAgain -->|是| OpenCamera
    DecisionAgain -->|否| EndSuccess([結束 - 掃描完成])
    
    style Start fill:#90EE90
    style EndSuccess fill:#90EE90
    style EndNoExtract fill:#FFB6C6
    style DecisionText fill:#FFE4B5
    style DecisionRetake fill:#FFE4B5
    style DecisionComplete fill:#FFE4B5
    style DecisionAgain fill:#FFE4B5
    style SummarizePhase fill:#87CEEB
    style OCRProcess fill:#DDA0DD
    style SendOCR fill:#DDA0DD
```

**Swimlanes**: User, System, PaddleOCR Service, Database

---

## 圖例

### 活動圖表符號

| 符號 | 含義 |
|--------|---------|
| ● (實心圓) | 開始活動 |
| ○ (空心圓) | 結束活動 |
| 矩形 | 活動/動作（流程步驟） |
| 菱形 | 決策點（是/否分支） |
| 箭頭 → | 控制流 |
| ∥ | 平行處理（開始/結束點） |
| 有邊框的矩形 | 泳道（參與者/元件） |
| ┌─┐ | 子活動/階段組 |

### 關鍵概念

- **決策點**：根據條件分支流程（是/否）
- **平行處理**：多項活動可同時進行（用∥標記）
- **泳道**：顯示哪位參與者/元件執行各活動
- **迴圈**：活動可根據條件重複
- **子活動**：複雜活動可分解為多個階段

---

## 如何使用這些圖表

1. **在Visual Paradigm中**：
   - 建立新的活動圖表
   - 為每位參與者新增泳道（使用者、系統、服務）
   - 按照上述流程新增活動和決策點
   - 用箭頭連接

2. **在Lucidchart中**：
   - 使用活動圖表形狀
   - 按泳道組織
   - 用菱形新增決策分支

3. **在Draw.io / Miro中**：
   - 建立泳道容器
   - 新增活動形狀（矩形）和決策形狀（菱形）
   - 用箭頭連接

4. **在Figma中**（帶外掛）：
   - 使用圖表元件
   - 按照上述流程結構

---

## 注意事項

- 這些活動圖表代表**正常路徑**和**常見例外流程**
- 每個圖表可根據需要擴展以增加更多細節
- 決策結果用括號標記：`[決策：...]`
- 平行活動在框中用豎線顯示：`├─`、`└─`
- 實現時，重點關注泳道以理解責任分配

---

**文件版本**：1.0  
**最後更新**：2026-06-12  
**用途**：OOSD作業 - 視覺語意辨識與飲食分析模組
