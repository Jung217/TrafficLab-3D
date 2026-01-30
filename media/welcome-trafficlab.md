```bash
  ██    ▒▒          ██▓▓    ██   
██░░▓▓██░░██        ▓▓░░  ▒▒░░██ 
  ▓▓░░▒▒░░██        ▓▓░░▒▒░░▓▓   ████████╗██████╗  █████╗ ███████╗███████╗██╗ ██████╗██╗      █████╗ ██████╗ 
    ▓▓░░░░██        ▓▓░░░░▓▓     ╚══██╔══╝██╔══██╗██╔══██╗██╔════╝██╔════╝██║██╔════╝██║     ██╔══██╗██╔══██╗
      ▓▓░░██        ▓▓░░░░▒▒        ██║   ██████╔╝███████║█████╗  █████╗  ██║██║     ██║     ███████║██████╔╝
      ██░░██        ▓▓░░▓▓░░▒▒      ██║   ██╔══██╗██╔══██║██╔══╝  ██╔══╝  ██║██║     ██║     ██╔══██║██╔══██╗
      ██░░▒▒▒▒▒▒▒▒▒▒▒▒░░  ▓▓░░██    ██║   ██║  ██║██║  ██║██║     ██║     ██║╚██████╗███████╗██║  ██║██████╔╝
        ████▒▒░░░░▒▒▓▓▓▓            ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝ ╚═════╝╚══════╝╚═╝  ╚═╝╚═════╝ 
        ████  ░░░░  ▓▓▓▓         
```

---

# 歡迎來到 TrafficLab

---

發布版本: v1.0

由 Yuk 開發
- [Yuk's Blog](https://yuk068.github.io/)
- [Github (Casual)](https://github.com/yuk068)
- [Github (Work)](https://github.com/duy-phamduc68)

---

補充資源:
- Youtube 指南 [建置中]
- 完整學術報告 [建置中]
- TrafficLab 完整部落格文章 [建置中]

---

## 1. 介紹

---

TrafficLab 是一套端到端的交通分析套件，涵蓋：

- 校正 (Calibration): 建立任意 CCTV 與其衛星地圖之間的雙向投影，並支援自訂 SVG。
- 推論 (Inference): 輕鬆更換物件偵測模型與追蹤器，並提供多種動力學參數與全面的控制選項。
- 視覺化 (Visualization): "數位雙生"體驗，並排同步顯示帶有 3D 邊界框的 CCTV 視圖，以及帶有地板框、速度和方向的衛星視圖。

TrafficLab 將易用性放在首位。只要有 mp4 格式的 CCTV 影片，並知道該地點在 Google 地圖上的位置，任何人都可以製作出展示先進電腦視覺技術的數位雙生展示。這特別適合那些可能無法取得相機校正參數和同步高品質衛星影像的學生、個人研究者和愛好者。

點擊程式左上角的任意分頁即可開始使用。

---

## 2. TrafficLab 的功能

---

TrafficLab 的功能分佈在 3 個主要分頁中，您可以在這些分頁之間切換而不會遺失其他分頁的工作。以下是各分頁的簡介。

---

### 2.1. 校正分頁

---

校正分頁產生 G Projection JSON 檔案（請參考報告），這有助於建立 CCTV 與 SAT（衛星）領域之間的雙向投影。它提供了一個全面、向下相容的階段式校正流程，包含以下階段：

- 第一階段：去畸變 (Phase 1: Undistort)
  - 選點階段 (Pick Stage): 快速驗證並初始化指定地點代碼的 G Projection 建構/重建。
  - 鏡頭階段 (Lens Stage): 設定內參矩陣 K。
  - 去畸變階段 (Undistort Stage): 調整符合 Brown-Conrady 畸變模型的畸變係數 [5 個係數]。
  - 驗證 1 (Validation 1): 您可以確認畸變和內參，結束此階段。
- 第二階段：單應性 (Phase 2: Homography)
  - 單應性錨點階段 (Homography Anchors Stage): 基於手動對點的單應性計算，使用 RANSAC 解算器。
  - 單應性視場階段 (Homography FOV Stage): 檢查疊加在衛星地圖上的變形 CCTV 影像，這也可作為直觀視覺化的視場多邊形。
  - 驗證 2 (Validation 2): 在 CCTV 中點擊接地點，看看它是否出現在衛星地圖上。
- 第三階段：視差 (Phase 3: Parallax)
  - 視差主體階段 (Parallax Subjects Stage): 建立 2 個主體的頭部和接地點，輸入其高度，計算相機位置。
  - 距離參考階段 (Distance Reference Stage): 輸入距離 [可從 Google 地圖/地球取得] 以建立像素與公尺的比例。
  - 驗證 3 (Validation 3): 點擊頭部點，輸入高度，查看 CCTV 中的接地點和衛星地圖上的實際位置。
- 選填 (Optional):
  - SVG 階段 (SVG Stage): 計算 SVG 與 SAT 之間的仿射矩陣。
  - 感興趣區域 (ROI): 選擇 ROI 的過濾策略。
- 最終驗證 (Final Validation): 測試 2D 邊界框如何轉換為 CCTV 中的 3D 框和 SAT 中的地板框。
- 儲存階段 (Save Stage): 實際儲存該地點代碼的 G Projection。

---

注意：地點代碼

---

您必須準備必要的資料夾和檔案以執行校正。這也有一個 Location Tab (地點分頁) 可協助您建立基本的空地點資料夾，準備進行校正。以下是範例結構：

```
TrafficLab-3D/
└── location/
    └── {location_code}/
        ├── footage/
        │   └── *.mp4
        │
        ├── illustrator/                 (選填，AI 素材)
        │   ├── layout_{location_code}.ai
        │   ├── roi_{location_code}.ai
        │   └── *.ai
        │
        ├── G_projection_{location_code}.json
        ├── cctv_{location_code}.png     (關鍵!)
        ├── sat_{location_code}.png      (關鍵!)
        ├── layout_{location_code}.svg   (選填)
        └── roi_{location_code}.png      (選填)
```

---

如果您沒注意到，您可以使用 Adobe Illustrator 建立自訂 SVG 和 ROI，請參考部落格文章/Youtube 影片以獲取關於製作這些資源的更詳細指南。

---

### 2.2. 推論分頁

---

推論分頁是您追蹤所有輸出 JSON 檔案產生的中心，這些檔案實際上由視覺化引擎使用，消除了在繁重渲染之上執行高需求計算的需要。關於參數，您將透過專案根目錄中的 inference_config.yaml 和 prior_dimensions.json 來控制它們。JSON 將儲存為壓縮的 .json.gz 格式以提高儲存效率。可控制的參數包括：

- 物件偵測模型 (Object detection model)
- 物件追蹤器 (Object tracker)
- 速度和方向平滑動力學 (Speed and orientation smoothing kinematics)

---

### 2.3. 視覺化分頁

---

這是輸出 JSON 檔案的視覺化引擎，透過工具列和鍵盤捷徑提供全面的控制，還具有靈活的 CCTV 和 SAT 面板並排視圖。

---

### Yuk 的備註

---

感謝您查看此程式。在 1.0 版本中，程式碼基本上只是將我學術報告中的研究筆記拼湊起來，但我真的希望它至少能派上用場，甚至只是為了好玩。距離我上次編寫像這樣帶有 GUI 的獨立應用程式已經很長一段時間了，我想上次應該是一年多前，那時我還在用 Java 做所有事情。那時我已經對 AI 產生了一些興趣，寫了一個五子棋 (Gomoku) 機器人，它能打敗生疏的玩家，但對上其他機器人大概是五五波。嗯，回憶就到此為止，簡單來說，考慮到時間和資源的限制，我對這個程式所達到的成果感到相當滿意。

---

此外，在製作 TrafficLab 3D 的過程中，我也發現了自己對具身化 (embodied) 和基於物理 (physically grounded) AI 這個小眾領域的樂趣，並計劃在這個領域製作更多專案，而不是像在這個程式之前那樣含糊地"做 AI"。總之，後會有期。

---

- Yuk - Dec. 25th, 2025

---
---

# Welcome to TrafficLab

---

Release: v1.0

Developed by Yuk
- [Yuk's Blog](https://yuk068.github.io/)
- [Github (Casual)](https://github.com/yuk068)
- [Github (Work)](https://github.com/duy-phamduc68)

---

Complementary resources:
- Youtube Guide at [Coming Soon]
- Check out full academic report at [Coming Soon]
- Read full blog post on TrafficLab at [Coming Soon]

---

## 1. Introduction

---

TrafficLab is an end-to-end traffic analysis suite that covers:

- Calibration: Establishing a two way projection between any CCTV and its satellite map, with support for custom SVG.
- Inference: Easily swap object detection models and object tracker along with numerous kinetics and comprehensive control of arguments.
- Visualization: A "digital twin" experience with side-by-side, synchronized view of CCTV with 3D bounding boxes and satellite view with floor box, speed, and orientation.

TrafficLab puts accessibility at the forefront, with just access to mp4 CCTV footage and knowing where that location is on Google Maps, anyone can create a fancy digital twin demo that demonstrates advanced computer vision, especially for students, individual investigators, and enthusiasts who might not have access to camera calibration and synchronized high quality satellite imagery.

Get started by navigating to any tabs on the top left corner of the program.

---

## 2. TrafficLab's Functionality

---

TrafficLab functionalities are spread across 3 main tabs, you can navigate to any of these tabs without losing work on another, below are brief description of each of the tabs.

---

### 2.1. Calibration Tab

---

Calibration Tab produces G Projection JSON files (refer to the report) which helps establish a two-way projection between the CCTV and SAT (satellite) domain, it presents a comprehensive, backwards compatible stage-based calibration process comprising of the following stages:

- Phase 1: Undistort
  - Pick Stage: Quickly validate and initialize construction/reconstruction of G Projection for a given location code.
  - Lens Stage: Configure intrinsics matrix K.
  - Undistort Stage: Adjust distortion coefficients obeying the Brown-Conrady distortion model (5 coefficients).
  - Validation 1: You can confirm the distortion and intrinsics, concluding the Phase.
- Phase 2: Homography
  - Homography Anchors Stage: Manual pair point based homography computation with RANSAC solver.
  - Homography FOV Stage: Check the warped CCTV overlaid on the SAT map, which also doubles up as a FOV polygon for intuitive visualization.
  - Validation 2: Click a ground contact point in CCTV and see it shows up on SAT map.
- Phase 3: Parallax
  - Parallax Subjects Stage: Establish Head and Ground Contact point of 2 Subjects, input their height, calculate the camera's position.
  - Distance Reference Stage: Enter the distance (obtainable from Google Maps/Earth) to establish pixel per meter ratio.
  - Validation 3: Click head point, enter height, see ground contact point in CCTV and actual position on SAT map.
- Optional:
  - SVG Stage: Compute affine matrix between SVG and SAT.
  - ROI: Choose a discard strategy for ROI.
- Final Validation: Test how 2D bounding box converts to 3D box in CCTV and floor box in SAT.
- Save Stage: Actually saves a G Projection for the location code.

---

Note: Location Code

---

You will have to prepare the necessary folders and files to perform calibration, there is also a Location Tab to help you with creating the barebone location folder, ready for calibration. Below is an example structure:

```
TrafficLab-3D/
└── location/
    └── {location_code}/
        ├── footage/
        │   └── *.mp4
        │
        ├── illustrator/                 (optional, Adobe Illustrator assets)
        │   ├── layout_{location_code}.ai
        │   ├── roi_{location_code}.ai
        │   └── *.ai
        │
        ├── G_projection_{location_code}.json
        ├── cctv_{location_code}.png     (critical!)
        ├── sat_{location_code}.png      (critical!)
        ├── layout_{location_code}.svg   (optional)
        └── roi_{location_code}.png      (optional)
```

---

If you haven't noticed, you can create custom SVG and ROI using Adobe Illustrator, refer to the blog post/Youtube video for a more detailed guide on crafting said resources.

---

### 2.2. Inference Tab

---

Inference Tab is a Hub for you to keep track of all your production of the output JSON files, these files are what are actually used by the visualization engine, eliminating the need to perform demanding computation on top of heavy rendering. For arguments, you will control them through inference_config.yaml and prior_dimensions.json in the project's root. JSON will be saved as the compressed .json.gz format for storage efficiency. Controllable arguments includes:

- Object detection model.
- Object tracker.
- Speed and orientation smoothing kinematics.

---

### 2.3. Visualization Tab

---

The visualization engine for the output JSON files, features comprehensive controls via a tool bar and keyboard shortcuts, flexible side by side view of CCTV and SAT panel.

---

### Yuk's Remark

---

Thank you for checking out this program, at this 1.0 version, the code is pretty much just glued on research notes from my academic report, however, I really hope it can at least have it uses, maybe even for fun. It has been a really long time since I get to code a standalone application like this with GUI and all, I think last time had to be over a year ago where I still did everything with Java, back then I already had some interest in AI with the Gomoku (5-in-a-row) game with a bot that i coded up myself, it beat rusty players but was a 50/50 against other bots. Welp, enough reminiscing, simply put, I'm quite happy with what I was able to achieve with this program given the time and resource constraint.

---

Moreover, in the process of making TrafficLab 3D, I've also discovered my own enjoyment for this little niche-embodied and physically grounded AI, and plan to make more projects in this field rather than just vaguely "doing AI" like i've been doing before this program. Anyway, see ya around.

---

- Yuk - Dec. 25th, 2025