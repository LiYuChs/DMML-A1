# PROMPT LOG（整理版）

本檔案將對話紀錄整理為統一格式，便於快速回顧。

## 01. 缺失比例圖與直方圖

### Q
針對各欄特徵繪製缺失比例圖表、直方圖。

### A（重點）
1. 先檢查 notebook cell 結構與現有變數命名。
2. 更新繪圖 cell，輸出兩張圖：
   - Missing Ratio by Feature（各欄缺失比例長條圖）
   - Histogram of Numeric Features（數值欄位直方圖，自動多子圖排版）
3. 執行成功，確認缺失率較高欄位主要為 marital_status、AdmitProcedure、ethnicity。

### 延伸建議
1. 只顯示缺失率 > 0 的欄位。
2. 長尾欄位改用對數尺度。
3. 類別欄位補上 countplot。

---

## 02. 條件篩選後查看 age unique

### Q
在 df[df['marital_status'].isnull()] 條件下，想看 age 是否 unique。

### A（可直接使用）
```python
s = df.loc[df['marital_status'].isnull(), 'age']

# 1) 是否全部唯一
print(s.is_unique)

# 2) 不同 age 數量
print('unique age count:', s.nunique())

# 3) 所有不重複 age 值
print(np.sort(s.dropna().unique()))
```

---

## 03. Jupyter 顯示完整 DataFrame

### Q
如何讓 jupyter cell 顯示完整 dataframe？

### A（重點）
```python
import pandas as pd
pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)
pd.set_option('display.max_colwidth', None)
pd.set_option('display.width', None)
```

輸出可用：
```python
df
# 或
from IPython.display import display
display(df)
```

暫時生效可用：
```python
with pd.option_context('display.max_rows', None, 'display.max_columns', None):
    display(df)
```

恢復預設：
```python
pd.reset_option('display.max_rows')
pd.reset_option('display.max_columns')
pd.reset_option('display.max_colwidth')
pd.reset_option('display.width')
```

---

## 04. 用 groupby().agg() 做資料集重點表

### Q
使用 df.groupby(...).agg(...) 整理一張關於資料集的重點表。

### A（重點）
建立 4 張統計表：
1. 按性別（gender）
2. 按入院類型（admit_type）
3. 按種族（ethnicity）
4. 按結果（ExpiredHospital）

聚合欄位包含：患者數量、平均年齡、平均住院天數、診斷數、檢查/事件數、死亡人數、死亡率。

---

## 05. TargetEncoder 用法

### Q
TargetEncoder 的用法。

### A（重點）
1. 核心概念：以類別對目標的平均效果編碼，適合高基數類別。
2. 可搭配平滑避免小樣本過擬合。
3. 實務規則：
   - 只能在訓練集 fit，測試集只 transform（避免 leakage）。
   - 高基數類別用 TargetEncoder，低基數類別用 OneHotEncoder。
   - 交叉驗證時要放進完整 Pipeline。

### Pipeline 範例
```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import TargetEncoder, OneHotEncoder, StandardScaler

target_cats = ['AdmitDiagnosis', 'AdmitProcedure', 'ethnicity', 'religion', 'admit_location']
one_hot_cats = ['gender', 'admit_type', 'marital_status', 'insurance']
num_cols = ['age', 'LOSdays', 'TotalNumInteract']

preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), num_cols),
        ('cat_target', TargetEncoder(target_type='binary', smooth='auto'), target_cats),
        ('cat_ohe', OneHotEncoder(drop='first', handle_unknown='ignore'), one_hot_cats),
    ],
    remainder='drop'
)

X_train_enc = preprocessor.fit_transform(X_train, y_train)
X_test_enc = preprocessor.transform(X_test)
```

---

## 06. OneHotEncoder(drop='first') 是什麼

### Q
OneHotEncoder(drop='first') 的意思？

### A（重點）
1. 每個類別欄位在 one-hot 後，會少保留一欄（基準類別）。
2. 目的：避免 dummy variable trap（完全共線性）。
3. 對線性模型（如 Logistic Regression）較穩定且更好解釋。
4. 樹模型通常不一定需要 drop='first'。

---

## 07. 找出 unknown categories 的欄位

### Q
針對警告：
UserWarning: Found unknown categories in columns [2] during transform...
幫我 print 出遺漏的 col。

### A（結果）
已在前處理區塊加入 train/test 類別差集檢查，並對照 warning 的欄位索引。

實際輸出：
1. Column index [2] -> marital_status: ['UNKNOWN (DEFAULT)', '__MISSING__']

### 對應處理
1. 前處理前統一清洗 marital_status（UNKNOWN (DEFAULT) -> UNKNOWN，缺失補 UNKNOWN）。
2. 保留 handle_unknown='ignore' 讓新類別編碼為全 0。

---

## 08. 將 train_df 的遺失值處理同步到 test_df

### Q
把對 train_df 做的遺失值處理也套用到 test_df，寫在新 cell。

### A（結果）
已新增新 cell，並執行成功。

處理內容：
1. 先把隱性缺失值轉為 NaN
   - AdmitProcedure == 'na'
   - religion: NOT SPECIFIED / UNOBTAINABLE
   - ethnicity: UNKNOWN/NOT SPECIFIED / UNABLE TO OBTAIN / PATIENT DECLINED TO ANSWER
   - marital_status == 'UNKNOWN (DEFAULT)'
2. 套用與 train_df 一致補值規則
   - age == 0 且 marital_status isnull -> NEWBORN
   - marital_status 其餘缺失 -> UNKNOWN
   - AdmitProcedure -> NO_PROCEDURE
   - religion -> UNKNOWN
   - ethnicity -> UNKNOWN
3. drop AdmitDiagnosis 缺失列

執行結果：
1. test_df 缺失值欄位為空（Series([], dtype: int64)）
2. test_df shape: (11791, 28)

---

## 09. Excel 開啟 CSV 中文亂碼

### Q
Excel 打開時中文亂碼，如何解決？

### A（重點）
匯出 CSV 時使用 UTF-8 BOM：

```python
eda_key_stats.to_csv('eda_key_stats.csv', index=False, encoding='utf-8-sig')
```

原因：Excel 若無 BOM，常用系統預設編碼讀取，導致中文亂碼。

---

## 10. 文件校對（錯字、格式、標點）

### Q
檢視整份文件是否有錯字或格式錯誤（換行、括號、標號等）。

### A（建議摘要）
1. 全半形括號風格統一。
2. 中文與英文/數字間空格統一。
3. 修正拼字（如 maritial_status -> marital_status）。
4. 中文語境標點一致（例如頓號使用）。
5. 修正 Markdown 圖片與連結語法（圖片括號內僅放路徑/URL）。
6. 超連結文字改為具語義標題。

---

## 11. 後續請求紀錄

### Q
幫我整理這整份 DEBUG_LOG.md

### A（狀態）
已另行整理並輸出為結構化版本（見 DEBUG_LOG.md）。

---

## 12. 後續請求紀錄

### Q
幫我整理這整份 PROMPT_LOG.md

### A（狀態）
已另行整理並輸出為結構化版本（見 PROMPT_LOG.md）。