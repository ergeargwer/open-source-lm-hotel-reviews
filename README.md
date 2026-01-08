# 本地與雲端評論分析工具 (Local & Cloud Hotel Review Analysis)

本專案提供一套基於 Jupyter Notebook 的評論分析工具，整合了本地模型 (Ollama) 與雲端模型 (OpenAI, Deepseek, Grok 等) 的 API，針對飯店評論資料進行多維度的自動化分析。系統同時包含人工標註介面與 SQLite 資料庫整合，方便進行模型效能比較與資料管理。

## � 相關論文 (Related Paper)

**論文標題 (Paper Title)**: A Study of Content Analysis of Hotel Reviews Assisted by Open-Source Language Models: A Case Study of Accommodation Reviews in the Taipei Area

**論文摘要 (Abstract)**:  
本研究旨在探討開源語言模型（Open-Source LLMs）於輔助內容分析（LLM-Assisted Content Analysis, LACA）框架之應用潛力，評估其於台北地區旅宿評論多維度文本分析之可行性與成本效益。研究採用混合方法設計，系統性比較 23 款開源語言模型（參數規模涵蓋 0.5B 至 32B）與 4 款主流雲端大語言模型，針對包含情感傾向、反諷識別及舒適度等九大維度的 4,952 筆中文旅宿評論進行測試。

研究結果顯示：（1）模型效能呈現顯著分層現象，參數量 14B 以上之模型（如 mistral-small3.1_24b）在準確性與穩定性上表現最佳，任務完成率超過 95%；（2）頂尖開源模型之總體準確率達 0.75，已具備與雲端模型（如 GPT-4o-mini）同等之分析水準；（3）在成本效益方面，本地部署模型分析一萬筆評論之能源成本約為新台幣 109 元，顯著優於雲端 API 服務所需之 137 元，然而雲端運算速度快約 4 倍。

綜合而言，實驗結果證實大型開源模型（≥ 14B）結合 LACA 框架，能有效處理中文評論分析任務，為旅宿業者提供一套兼具高準確度與低成本的顧客洞察工具。

## �📁 專案檔案結構

- **`本地與雲端評論分析v18.ipynb`**: 核心程式碼，包含所有分析邏輯與操作介面。
- **`codebook_v5.xlsx`**: 編碼簿 (Codebook)，定義分析的維度、定義與評分標準。
- **`hotel_data_....xlsx`**: 原始評論資料檔 (需包含評論內容與相關 metadata)。
- **`analysis_log.txt`**: 程式執行日誌。
- **`processing_progress.pkl`**: 批次處理的進度備份檔 (Pickle 格式)。
- **`multidimensional_analysis_results.json`**: 分析結果輸出檔。
- **`模型一致性分析工具/`**: 子資料夾，包含模型一致性分析工具。
  - `模型一致性分析工具20251213.ipynb`: 分析各模型與人工標註一致性的工具。
  - `multidimensional_analysis_results_....csv`: 一致性分析的輸入資料。
  - `README.md`: 一致性分析工具的詳細說明。

## 🚀 功能特色

1.  **多模型支援**：
    -   **本地模型**：透過 Ollama 執行 (如 Llama 3, Phi-3 等)。
    -   **雲端模型**：支援 OpenAI (GPT), Deepseek, Grok (xAI) 等相容 API。
    -   **適配器模式**：透過 `ModelAdapter` 輕鬆擴充新的模型供應商。

2.  **穩健的分析流程**：
    -   **自動重試機制**：當 API 請求失敗或回傳格式錯誤時自動重試。
       - 支援指數退避策略：每次重試間隔時間遞增，避免頻繁請求造成額外負擔。
       - 最大重試次數預設為 3 次，可根據需求調整。
       - 適用於網路異常、API 速率限制 (429 錯誤) 等情境。
    -   **JSON 格式驗證與比對**：確保模型輸出符合預定義的 JSON 結構。
       - 自動檢查輸出是否為有效 JSON，並驗證必要欄位是否存在。
       - 若格式錯誤，觸發重試機制並使用簡化提示詞重新請求。
       - 支援動態格式比對：根據 Codebook 定義的維度自動生成驗證規則。
       - 輸入資料格式檢查：驗證評論資料和代碼簿檔案的欄位完整性，自動映射欄位名稱，統計資料品質指標。
    -   **斷點續傳**：批次分析時會定期儲存進度 (`processing_progress.pkl`)，避免中斷後需重頭開始。

3.  **人工標註與驗證**：
    -   內建人工標註介面，可對評論進行人工評分。
    -   支援模型結果與人工標註的比較統計。

4.  **資料庫整合**：
    -   支援將分析結果寫入 SQLite 資料庫，方便後續查詢與管理。

## 🛠️ 環境需求與安裝

### 1. Python 套件
請確保安裝以下 Python 套件 (可直接執行 Notebook 第一個 Cell 安裝)：

```bash
pip install pandas numpy requests tqdm pynvml matplotlib python-dotenv openpyxl ollama
```

### 2. 環境變數 (.env)
若使用雲端 API，請在專案根目錄建立 `.env` 檔案並設定 API Key：

```env
OPENAI_API_KEY=sk-...
DEEPSEEK_API_KEY=...
GROK_API_KEY=...
```

### 3. 本地模型 (Ollama)
若需使用本地模型，請確保已安裝 [Ollama](https://ollama.com/) 並啟動服務：
- API URL 預設為: `http://localhost:11434/api/generate`

## 📖 程式結構解析 (Notebook 章節說明)

本程式 (`本地與雲端評論分析v18.ipynb`) 的邏輯結構如下：

### **1. 初始化與設定**
- **Cell 1**: 安裝必要套件。
- **Cell 2**: 匯入 Library，設定 `APIConfig` (API 端點與金鑰讀取)，定義檔案路徑 (Codebook, 資料檔, 輸出檔)。

### **2. 核心邏輯 (Model Adapters)**
- **Cell 3**: 定義模型適配器類別。
    - `ModelAdapter` (Base Class)
    - `OllamaAdapter`: 處理本地 Ollama API 呼叫。
    - `OpenAIAdapter`: 處理 OpenAI 格式 (包含 Deepseek, Grok) 的 API 呼叫。

### **3. 資料處理**
- **Cell 4**:
    - `load_codebook()`: 讀取 Excel 編碼簿。
    - `load_reviews()`: 讀取評論資料。
    - `build_system_prompt()`: 根據 Codebook 動態產生給 LLM 的 System Prompt。

### **4. 分析功能**
- **Cell 5**: `configure_model()` - 選擇要使用的模型供應商與具體模型名稱。
- **Cell 6**: `analyze_single_review()` - 執行單筆分析，包含 Prompt 組合、API 請求、JSON 解析與錯誤重試。
- **Cell 7**: `batch_analyze_reviews()` - 批次處理所有評論，包含進度條顯示與斷點儲存。

### **5. 測試與驗證**
- **Cell 8**: `test_api_connection()` - 測試 API 連線狀態。
- **Cell 12**: 互動式模型測試介面。

### **6. 人工標註**
- **Cell 9**: 提供介面供使用者閱讀評論並手動輸入各維度的分析結果，用於建立黃金標準 (Gold Standard)。

### **7. 主程式與資料庫**
- **Cell 10**: 主選單 (Main Menu)，整合上述所有功能，提供互動式操作流程。
- **Cell 13-15**: SQLite 資料庫操作 (寫入結果、查詢記錄、清除資料)。

## 📝 使用說明

1.  **準備資料**：確認 `codebook_v5.xlsx` 與評論資料檔 (Excel) 位於正確路徑。
    - *注意：請檢查 Cell 2 中的 `reviews_path` 變數是否與您的實際資料檔名相符。*
2.  **啟動 Notebook**：依序執行 Cell 1 與 Cell 2 初始化環境。
3.  **執行主選單**：執行 **Cell 10**，透過選單選擇功能：
    - 輸入 `1`：執行批次分析 (Batch Analysis)。
    - 輸入 `2`：進行單筆測試。
    - 輸入 `5`：進入人工標註模式。
    - 輸入 `db_write`：將現有 JSON 結果寫入資料庫。
