# 資料探勘作業 - MIMIC-3C 前處理與 EDA

## 專案簡介
本專案針對 MIMIC-3C 資料集，完成院內死亡預測（in-hospital mortality prediction）所需的端到端資料前處理與探索式資料分析（EDA）。

核心目標：
- 將原始臨床表格資料轉換為可直接用於 machine learning 的 model-ready 資料集。

主要流程包含：
1. 資料理解與資料品質檢查
2. 隱性缺失值清理與 missing value handling
3. 特徵轉換（log transform、scaling、encoding）
4. 特徵工程與降維（PCA）
5. 分析與建模輸出檔案匯出

---

## 資料集說明
- 來源：MIMIC-III Clinical Database Demo（Kaggle）
- 本專案使用檔案：`data/mimic3c.csv`
- 目標欄位（target label）：`ExpiredHospital`（0 = survived，1 = expired）

---

## 專案目錄結構

```text
A1_1158029_李宇晟/
├─ A1_1158029.ipynb        # 主 notebook（前處理 + EDA + 匯出）
├─ A1_1158029.md           # 作業說明與分析報告
├─ README.md               # 本文件
├─ DEBUG_LOG.md            # 除錯紀錄
├─ PROMPT_LOG.md           # 問答與操作紀錄
├─ requirements.txt        # 相依套件
└─ data/
   └─ mimic3c.csv          # 原始資料
```

---

## 環境設定

### 1) 建立並啟用 Python 環境
可使用你習慣的環境管理工具（conda 或 venv）。

### 2) 安裝相依套件
```bash
pip install -r requirements.txt
```
或是
```bash
pip install ipykernel
pip install matplotlib
pip install seaborn
pip install scikit-learn
```

### 3) 開啟並執行 notebook
請依序執行：
- `A1_1158029.ipynb`

---

## Notebook 流程摘要

### Step 1. 資料切分
- 以 `ExpiredHospital` 進行 stratified train/test split

### Step 2. 缺失值標準化
先將隱性缺失字串轉為標準 missing values：
- `UNKNOWN (DEFAULT)`
- `na`
- `NOT SPECIFIED`
- `UNOBTAINABLE`
- `UNKNOWN/NOT SPECIFIED`
- `UNABLE TO OBTAIN`
- `PATIENT DECLINED TO ANSWER`

### Step 3. 缺失值處理
- 依欄位語意進行 categorical imputation
- 對 `AdmitDiagnosis` 缺失列執行 drop（占比小且臨床資訊重要）

### Step 4. 特徵前處理
- 數值欄位：log transform（右偏分佈特徵）+ standardization
- 類別欄位：
  - 高基數欄位使用 TargetEncoder
  - 低基數欄位使用 OneHotEncoder

### Step 5. 特徵工程
- `resource_intensity`
- `long_LOS_flag`
- `complexity_score`

### Step 6. 對 `Num*` 計數欄位做 PCA
- 降低共線性關係，壓縮高維且噪音較高的計數特徵

### Step 7. 匯出 artifacts
Notebook 完整執行後，會將分析與建模輸出寫入 `output/`。

---

## 預期輸出檔案
完整執行 notebook 後，`output/` 內預期會有：

- `df_feature.csv`：
  最終整併前的特徵資料

- `df_model_ready.csv`：
  可直接建模的最終表格（numeric features + target）

- `eda_key_stats.csv`：
  報告中引用的關鍵統計值

- `missing_value_ratio.jpg`：
  缺失比例視覺化圖表

- `distribution_plots.jpg`：
  連續變數分布圖

---

## 驗證檢查清單
建議在執行完後快速檢查：

1. Notebook 可自上而下無錯誤執行。
2. `output/` 會自動建立。
3. `df_model_ready.csv` 存在，且包含 `target`。
4. `df_model_ready.csv` 不含缺失值。
5. `eda_key_stats.csv` 含有 `A1_1158029.md` 引用的關鍵統計項目。

---

## 常見問題

### Excel 開啟 CSV 中文亂碼
匯出 CSV 時請加入 BOM：
```python
df.to_csv(path, index=False, encoding='utf-8-sig')
```

### OneHotEncoder 出現 unknown category warning
建議：
- 設定 `handle_unknown='ignore'`
- 在 transform 前先完成一致的 train/test 類別清理

---

## 相關文件
- `A1_1158029.md`：作業主文件與分析解讀
- `DEBUG_LOG.md`：錯誤與修正紀錄
- `PROMPT_LOG.md`：互動提示與操作歷程