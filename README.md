# Eyes-Catching

### 使用簡單演算法進行眼睛偵測（不使用任何學習模型）

---

## 專案概述

本專案展示一種**輕量且可解釋性高**的眼睛偵測方法，完全基於傳統影像處理技術，**不依賴任何機器學習或深度學習模型**。

整體流程採用兩階段方法：

> **先定位眼睛 ROI → 再於 ROI 中偵測眼珠位置**

（本專案於 Google Colab 環境實作）

---

## 設計理念

本方法主要運用以下技術：

* 灰階轉換（Grayscale）
* 二值化（Thresholding）
* 幾何結構分析（Row-wise Geometry Analysis）
* 群心計算（Centroid）

其特點為：

* 執行速度快
* 可解釋性高
* 不需訓練資料
* 適合低資源環境

---

## 處理流程

---

### (1) 擷取人體輪廓（Row-based）

本方法**不使用 contour 或 bounding box**，而是透過逐列掃描方式分析人體輪廓。

**步驟：**

* 將影像轉為灰階
* 使用固定 threshold（如 215）進行二值化（前景=人體）
* 對每一列（row）：
  * 找出該列的最左與最右前景點
  * 計算寬度與中心位置

**計算結果：**

* 臉部水平中心（x_face）：所有列中心的中位數
* 頭頂（y_top）：第一個出現前景的列
* 脖子（y_neck）：在指定範圍內「最窄寬度」的位置



**目的：**
利用人體幾何特徵（頭寬→脖子變窄）來建立穩定參考點。

<img width="1135" height="719" alt="image" src="https://github.com/user-attachments/assets/56064a76-84ad-4846-935f-7557cf4ad090" />
---

### (2) 根據幾何比例定位眼睛 ROI

透過頭頂與脖子之間的距離，推估臉部比例，進而定位眼睛區域。

**步驟：**

* 計算頭部高度：
  ```
  head_h = y_neck - y_top
  ```

* 定義比例單位：
  ```
  dot = head_h / 10
  ```

* 估計眼睛中心位置：
  ```
  y_eye ≈ y_top + 0.55 * head_h
  ```

* 以臉部中心為基準建立 ROI：
  ```
  ROI 寬度 ≈ ±3 * dot
  ROI 高度 ≈ ±1.2 * dot
  ```

**目的：**
將搜尋範圍限制在「雙眼區域」，降低干擾（背景、身體、頭髮）。
<img width="1136" height="719" alt="image" src="https://github.com/user-attachments/assets/cdc3668f-37ea-4b30-be2d-6f5bf6c279f0" />


---

### (3) 在 ROI 中偵測眼珠中心（核心步驟）

在 ROI 中找出左右兩個「最暗區域群」的中心，視為眼珠位置。

---

#### Step 1：ROI 前處理

* 轉灰階
* 使用 Gaussian Blur 去除雜訊
* 二值化（反轉，讓暗區變白）
* 做Close使區域更完整
<img width="263" height="103" alt="image" src="https://github.com/user-attachments/assets/4daf1783-47cb-4f9d-8f37-94c26374a8fb" />
<img width="262" height="103" alt="image" src="https://github.com/user-attachments/assets/4dad55b2-b209-459a-9833-5e977d831cbb" />
<img width="263" height="103" alt="image" src="https://github.com/user-attachments/assets/cbd7f067-396d-437f-9d20-0b2060e368ef" />

---
#### Step 2：左右分群計算群心（Centroid）

* 使用：
  ```
  cv2.connectedComponentsWithStats()
  ```

* 每個白色區塊代表一個「候選暗區」

取得資訊：

* 面積（area）
* 外接框（bounding box）
* 群心（centroid）


*  每一群使用：

```
(cx, cy) = centroid
```

作為該眼睛的位置。
<img width="263" height="103" alt="image" src="https://github.com/user-attachments/assets/bba3543a-7357-41e7-8c8b-e786b6d281e3" />


---

## 實驗結果

最終結果會在原始影像上標示：

* ROI 區域（藍框）
* 左右眼瞳孔位置（紅點）

<img width="1934" height="1069" alt="image" src="https://github.com/user-attachments/assets/d07309cd-ed56-4d64-b64c-cdc4a7a09bc2" />

---

## 優點

* 不需要訓練資料
* 計算速度快（適合即時系統）
* 流程透明、可解釋性高
* 易於調整與除錯

---

## 限制

* 對光線變化敏感（threshold 固定）
* 二值化效果會影響結果
* 群心可能因雜訊偏移
* 無法處理：
  * 側臉
  * 遮擋（瀏海 / 手）
  * 強烈陰影

---

## 結論

本方法展示了一種**完全不依賴學習模型**的眼睛定位方式，透過幾何分析與影像處理技術即可達成基本的眼睛偵測。

雖然在複雜環境下仍有限制，但其**輕量性與可解釋性**使其在嵌入式系統或教學應用中具有實用價值。
