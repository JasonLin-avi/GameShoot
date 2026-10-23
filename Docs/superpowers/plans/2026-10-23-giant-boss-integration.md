# 綠巨人巨型頭目 (Giant Boss) 整合實作計畫

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 將 `giant.html` 中的 1:1 有機曲面綠巨人 3D 模型、生物力學步態動作與踩踏震波系統，無縫整合進 `index.html`，替換原本簡約圓柱體紅軍頭目，打造具備沉重步伐與殘血狂暴咆哮的史詩級巨型頭目 (Giant Boss)。

**Architecture:** 
- 模組化封裝純程序化 PBR 材質 `createGiantMaterials()` 與階層式 3D 骨骼模型 `createGiantBossModel()`。
- 建構零 GC 步態運動學解算器 `updateGiantKinematics()` 與踏步衝擊波物件池 (`StompRingsPool`)。
- 在 `EntityManager` 中以全新綠巨人替換原有圓柱 Boss，精準銜接血條 Sprite、關卡波次、藍軍衝撞、鋼鐵人集束砲 (Unibeam) 與微型導彈。

**Tech Stack:** Three.js (r182+, WebGLRenderer, PCFSoftShadowMap, PBR MeshStandardMaterial), Vanilla JavaScript (ES Module).

## Global Constraints
- **零 GC 分配 (Zero Allocations in Loop)**：步態運動學矩陣與向量計算必須複用靜態變數，絕不於動畫迴圈內動態配置。
- **純程序化材質**：不使用任何外部貼圖檔案，避免 CORS 或圖片載入異常。
- **維持 60 FPS**：整體 Draw Call 增量嚴格受控於 30 以內，踏步震波採用物件池循環重用。
- **繁體中文與註釋規範**：代碼註釋說明「Why」，對外訊息使用繁體中文。

---

### Task 1: 建立綠巨人純程序化材質與階層式 3D 骨骼模型

**Files to modify:**
- `index.html`

- [ ] **Step 1: 定義巨人專用 PBR 材質生成器 `createGiantMaterials`**
  - 在 `index.html` 的英雄材質系統後方，定義 `createGiantMaterials()`。
  - 建立共享材質：`skinMain` (草綠肌理 `0x6ea33b`)、`skinDark` (陰影 `0x4e7926`)、`skinLight` (高光 `0x7ab044`)、`pantsPurple` (破損紫褲 `0x4e2a50`)、`pantsDark` (腰帶皺褶 `0x371b39`)、`hairBlack` (黑髮 `0x1c201d`)、`teethWhite` (白牙 `0xf8fafc`)、`eyePupil` (黑瞳 `0x0f172a`)、`nailMat` (指甲 `0x8cc152`)。

- [ ] **Step 2: 移植並重構階層式骨骼模型生成器 `createGiantBossModel`**
  - 從 `giant.html` 移植骨骼組裝邏輯，封裝為 `createGiantBossModel(materials)`。
  - 組裝層次：
    - `root` (面朝 -Z 方向，縮放適配戰場 11 寬度跑道，高度約 3.8 單位)。
    - `pelvisGroup` (骨盆核心、臀部肌肉、紫褲腰帶)。
    - `spineGroup` & `chestGroup` (腰椎、胸大肌、斜方肌、八塊腹肌、前鋸肌)。
    - `neckGroup` & `headGroup` (頸肌、方下顎、張口咬牙、怒意雙眼、黑髮)。
    - `leftArm` & `rightArm` (三角肌、二頭肌、三頭肌、前臂、手腕與巨拳)。
    - `leftLeg` & `rightLeg` (股四頭肌、膝蓋、小腿、腳掌與腳趾)。
  - 返回完整關節節點物件字典，供運動學解算器調用。

- [ ] **Step 3: 驗證模型語法與初始化無報錯**
  - 使用 PowerShell 檢查 `index.html` 語法，確保沒有括號或未閉合標籤。
  - 確保無全域命名衝突。

---

### Task 2: 建立踏步震波物件池與運動學解算器 (`StompRingsPool`, `updateGiantKinematics`)

**Files to modify:**
- `index.html`

- [ ] **Step 1: 實作踏步震波環物件池 (Stomp Rings Pool)**
  - 在 `EntityManager` 或獨立模組中預先配置 8 個 `THREE.RingGeometry(0.2, 0.45, 28)` 網格。
  - 物件池資料結構：`{ mesh, active: false, life: 0, maxLife: 0.5, x: 0, z: 0 }`。
  - 實作 `spawnStompRing(x, z)` 與 `updateStompRings(dt)`，實現半徑由 1.0 擴大至 2.4、透明度由 0.7 漸變至 0。

- [ ] **Step 2: 實作零 GC 步態與狂暴運動學解算器 `updateGiantKinematics`**
  - 步態相角累加：`walkPhase += dt * walkSpeed * 4.2`。
  - 雙腿解算：大腿前後擺動、膝關節屈伸、腳掌俯仰踩踏。
  - 踏地判定：當相角通過 $\pi$ 或 $2\pi$ 臨界點時觸發 `spawnStompRing` 與畫面微震 `shakeAmount = 0.15`。
  - 雙臂解算：與對側腿部反向前後擺盪，雙拳微握。
  - 狂暴怒吼 (Roar Mode) 動態：雙臂向兩側上方張開屈肘、胸口向前挺起、頭部後仰，並於咆哮時產生微幅高頻顫動。

- [ ] **Step 3: 驗證解算邏輯與數值穩定性**
  - 確保在各 FPS 與 Delta Time 波動下無旋轉突變或抖動，完全零物件記憶體分配。

---

### Task 3: 升級 `EntityManager` 巨型頭目生命週期與替換圓柱 Boss

**Files to modify:**
- `index.html`

- [ ] **Step 1: 改造 `EntityManager.initBoss()`**
  - 移除原有的簡易圓柱體 (`CylinderGeometry`) 與角錐尖刺 (`ConeGeometry`)。
  - 呼叫 `createGiantMaterials()` 與 `createGiantBossModel()` 生成綠巨人。
  - 將巨人模型加入 `bossGroup`，並微調血條 Sprite 位置（高度調整為 Y = 4.2，縮放適當放大）。
  - 初始化巨人運動學狀態：`walkPhase: 0, walkSpeed: 1.0, isRoaring: false, roarTimer: 0, enraged: false, lastLeftStomp: false, lastRightStomp: false`。

- [ ] **Step 2: 改造 `EntityManager.updateBoss(dt)`**
  - 當 `boss.active` 為 true 時：
    - 檢查狂暴觸發條件：`!boss.enraged && boss.hp <= boss.maxHp * 0.3`。
    - 觸發時進入 2.0 秒咆哮狀態 (`isRoaring = true`)，並播放 Boss 警告音效或怒吼效果。
    - 咆哮結束後，常駐速度由 -2.2 提升至 -2.86。
    - 更新巨人世界座標：`boss.z += boss.vz * dt`。
    - 調用 `updateGiantKinematics()` 驅動肢體動作與震波環。

- [ ] **Step 3: 改造 `EntityManager.resetAll()` 與記憶體釋放**
  - 在重置時重置巨人狀態，將模型移回 `(0, -999, 0)`。
  - 確保震波環物件池同步重置為隱藏。

---

### Task 4: 戰鬥碰撞、英雄技能聯動與全面驗收測試

**Files to modify:**
- `index.html`

- [ ] **Step 1: 對接受擊反饋與碰撞半徑**
  - 確保藍軍小兵、鋼鐵人 Unibeam 集束砲、肩部微型追蹤導彈的碰撞半徑與綠巨人體型（寬 2.5、深 2.0）完美匹配。
  - 受擊時觸發微擊退 (+Z 位移) 與受擊爆花。

- [ ] **Step 2: 陣亡大爆炸與獎勵掉落**
  - 當巨人血量降為 0 時：
    - 觸發大規模爆炸粒子 (`spawnExplosion(boss.x, 2.0, boss.z, 30)`)。
    - 觸發大震屏 `shakeAmount = 0.9` 與擊殺金幣獎勵 (+50 Coins)。
    - 重置巨人狀態並隱藏模型。

- [ ] **Step 3: 瀏覽器即時驗證與截圖**
  - 啟動 local dev server 或使用現有環境進行瀏覽器渲染測試。
  - 檢視 Boss 登場、行走動作、踏步震波、受擊微退、狂暴怒吼與擊殺爆炸。
  - 截圖驗收產出確認報告。
