# DEBUG LOG (整理版)

本文件整理 Notebook 開發過程中遇到的錯誤與修正方式，方便快速回查。

## 1) ImportError: train_test_split 匯入路徑錯誤

### 錯誤訊息
```python
ImportError: cannot import name 'train_test_split' from 'sklearn.preprocessing'
```

### 發生原因
train_test_split 不在 sklearn.preprocessing，而是在 sklearn.model_selection。

### 修正方式
```python
from sklearn.model_selection import train_test_split
```

---

## 2) IndexingError: 布林索引對齊失敗

### 錯誤觸發程式
```python
train_df[(train_df.isnull().sum() > 0)]
```

### 錯誤訊息
```python
IndexingError: Unalignable boolean Series provided as indexer
```

### 發生原因
train_df.isnull().sum() > 0 產生的是欄位層級的布林 Series，
但上述寫法是拿來篩選列，索引軸不一致而報錯。

### 修正方式
```python
missing_count = dataframe.isnull().sum()
missing_features = missing_count[missing_count > 0]
```

---

## 3) AttributeError: seaborn 的 ax 傳入 ndarray

### 錯誤觸發程式
```python
sns.histplot(data=train_df, x='NumCPTevents', bins=30, kde=True, ax=axes[0], color='skyblue')
```

### 錯誤訊息
```python
AttributeError: 'numpy.ndarray' object has no attribute 'xaxis'
```

### 發生原因
plt.subplots(2, 3) 產生的 axes 是二維陣列。
axes[0] 是一列子圖，不是單一 Axes 物件，seaborn 需要單一 Axes。

### 修正方式
```python
sns.set_theme(style='whitegrid')

fig, axes = plt.subplots(2, 3, figsize=(18, 5))

sns.histplot(data=train_df, x='NumCPTevents', bins=30, kde=True, ax=axes[0][0], color='skyblue')
axes[0][0].set_title('Distribution of NumCPTevents')

sns.histplot(data=train_df, x='NumInput', bins=50, kde=True, ax=axes[0][1], color='salmon')
axes[0][1].set_title('Distribution of NumInput')

sns.histplot(data=train_df, x='NumLabs', bins=50, kde=True, ax=axes[0][2], color='lightgreen')
axes[0][2].set_title('Distribution of NumLabs')

plt.tight_layout()
plt.show()
```

---

## 4) UserWarning: OneHotEncoder 出現未知類別

### 警告訊息
```python
UserWarning: Found unknown categories in columns [2] during transform. These unknown categories will be encoded as all zeros
```

### 說明
測試資料出現訓練資料未看過的類別。
若 OneHotEncoder 設定 handle_unknown='ignore'，未知類別被編碼為全 0 屬於預期行為。

### 建議處理
1. 保持 OneHotEncoder(handle_unknown='ignore')。
2. 在 EDA 階段先檢查 train/test 類別差異並記錄來源。
3. 若未知類別比例偏高，可將低頻類別先合併為 OTHER 再編碼。
