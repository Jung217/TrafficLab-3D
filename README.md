# TrafficLab 3D

TrafficLab 將「易用性」放在首位。只需擁有 CCTV（監視器）的 MP4 影像，並知道其在 Google 地圖上的位置，任何人都能製作出精美的數位分身（Digital Twin）展示。這對學生、獨立研究者或愛好者特別有幫助，因為他們可能無法取得專業的相機校準數據或同步的高畫質衛星影像。

**執行此程式前，先詳閱此 README。點擊[此處](#getting-started)跳轉至「入門指南」。**

## 檔案結構

```
TrafficLab-3D/
├── location/
│   └── {地點代碼}/
│       ├── footage/
│       │   └── *.mp4 (監視器影片)
│       │
│       ├── illustrator/                 (選填，Adobe Illustrator)
│       │   ├── layout_{地點代碼}.ai
│       │   ├── roi_{地點代碼}.ai
│       │   └── *.ai
│       │
│       ├── G_projection_{地點代碼}.json
│       ├── cctv_{地點代碼}.png         (關鍵檔案！)
│       ├── sat_{地點代碼}.png          (關鍵檔案！)
│       ├── layout_{地點代碼}.svg       (選填)
│       └── roi_{地點代碼}.png          (選填)
│
├── media/                               (README 與介紹分頁所需的資源)
├── gui/                                 (GUI 圖形介面實作)
├── models/                              (物件偵測與追蹤模型)
│   └── *.pt                             (YOLO 權重檔)
├── output/
│   └── model-{模型名稱}_tracker-{追蹤器名稱}/
│       └── {設定名稱}/
│           └── {地點代碼}/
│               └── *.json.gz            (推論結果輸出)
│
├── environment.yml                      (環境設定檔)
├── inference_config.yaml                (推論設定檔)
├── prior_dimensions.json                (先驗尺寸資料)
└── main.py                              (主程式)

```

## 簡介

TrafficLab 是一款端到端的交通分析套件，涵蓋以下功能：

* **校準 (Calibration)：** 建立任何 CCTV 與其衛星地圖之間的雙向投影，並支援自定義 SVG 格式。
* **推論 (Inference)：** 輕鬆切換不同的物件偵測模型與追蹤器，並提供多種運動學參數與完整的參數控制。
* **視覺化 (Visualization)：** 提供「數位分身」體驗，將 CCTV（具備 3D 邊界框）與衛星視圖（具備地面框、速度與方向）進行同步並排顯示。

導航至程式左上角的任一分頁即可開始使用。

## 功能說明

TrafficLab 的功能主要分布在三個分頁中，您可以在這些分頁間自由切換而不會遺失工作進度。以下是各分頁的簡要說明：

### 1. 校準分頁 (Calibration Tab)

校準分頁會生成 **G Projection JSON** 檔案（詳見報告），用於建立 CCTV 與衛星圖（SAT）領域間的雙向投影。它提供了一個完整且具備向後相容性的階段式校準流程：

* **第一階段：去畸變 (Undistort)**
* **選取階段 (Pick Stage)：** 快速驗證並初始化特定地點代碼的投影建構。
* **鏡頭階段 (Lens Stage)：** 設定相機內參矩陣 K。
* **去畸變階段 (Undistort Stage)：** 根據 Brown-Conrady 模型調整 5 個畸變係數。
* **驗證 1：** 確認去畸變與內參設定，結束此階段。


* **第二階段：單應性 (Homography)**
* **單應性錨點階段：** 使用 RANSAC 演算法進行手動成對點的單應性計算。
* **視野階段 (FOV Stage)：** 查看覆蓋在衛星圖上的扭曲 CCTV 影像，同時作為視野多邊形進行視覺化。
* **驗證 2：** 在 CCTV 影像中點擊地面接觸點，觀察其是否正確顯示在衛星地圖上。


* **第三階段：視差 (Parallax)**
* **視差主體階段：** 設定兩個主體的頭部與地面接觸點，輸入其身高，藉此計算相機位置。
* **距離參考階段：** 輸入參考距離（可從 Google 地圖取得）以建立像素與公尺的比例。
* **驗證 3：** 點擊頭部點並輸入高度，觀察 CCTV 中的地面點與衛星圖上的實際位置。


* **選用階段：**
* **SVG 階段：** 計算 SVG 與衛星圖之間的仿射矩陣。
* **ROI 階段：** 設定感興趣區域（ROI）的排除策略。


* **最終驗證：** 測試 2D 邊界框如何轉換為 CCTV 中的 3D 框以及衛星圖中的地面框。
* **儲存階段：** 確認並儲存該地點代碼的 G Projection 檔案。

**備註：地點代碼 (Location Code)**
您需要準備必要的資料夾與檔案來進行校準。程式中也有「地點分頁 (Location Tab)」協助您建立基礎的地點資料夾。您可以使用 Adobe Illustrator 製作自定義的 SVG 與 ROI 檔案，詳細教學請參考部落格或 YouTube 影片。

### 2. 推論分頁 (Inference Tab)

推論分頁是管理所有 JSON 輸出檔案製作的中心。這些檔案是視覺化引擎運作的基礎，能避免在重度渲染時進行高負載的推論運算。參數控制透過專案根目錄下的 `inference_config.yaml` 與 `prior_dimensions.json` 進行。JSON 檔案將以 `.json.gz` 壓縮格式儲存以節省空間。可控制的參數包括：

* 物件偵測模型（Object detection model）。
* 物件追蹤器（Object tracker）。
* 速度與方向的運動學平滑處理。

### 3. 視覺化分頁 (Visualization Tab)

這是輸出 JSON 檔案的視覺化引擎，具有工具列與鍵盤快捷鍵的完整控制，並提供 CCTV 與衛星視圖（SAT）面板的彈性並排顯示。

## 入門指南 (Getting Started)

安裝必要的 conda/venv 環境，然後執行 `main.py`：

```bash
conda env create -f environment.yml
python main.py

```

在這個 [Google Drive](https://drive.google.com/drive/folders/14NVnbrUUfII3tRdI8OOEPnLzKbs3SPvn?usp=sharing) 中，您可以找到：

* 放置於 `models/` 資料夾的微調後 `YOLOv8-s` 和 `YOLOv11-s` 模型。
* 兩個已建構好投影的地點資料夾（一個含 SVG，一個不含）。如果您需要更多不同地點的投影資料，請與我聯繫。請將其放入 `location/` 資料夾。
* 一個預處理好的 `.json.gz` 輸出檔案（需要配合雲端硬碟中的 `119NH` 資料夾）。

本專案靈感來自於論文 [Rezaei et al. 2023](https://www.sciencedirect.com/science/article/pii/S0957417423007558)

## Origin Author Information
> 發布版本：v1.0

* 開發者：Yuk
  * [Yuk 的部落格](https://yuk068.github.io/)
  * [Github (個人/休閒)](https://github.com/yuk068)
  * [Github (工作)](https://github.com/duy-phamduc68)
* 補充資源：
  * [YouTube 專案演示](https://www.youtube.com/watch?v=AYUXXnzenvk)
* 在 **TrafficLab 3D v1.1** 中，目標：
  * 重寫程式碼庫，目前的程式碼相當凌亂，深感抱歉。
  * 針對更平滑、符合物理約束的運動學與追蹤進行更多研究。
  * 優化校準分頁，或許加入一些自動校準的選項。

**注意：** 此方法目前僅適用於單一平面的環境。
