# 鋼鐵人整合進《軍團防線》藍軍英雄指揮官實施計畫 (Implementation Plan)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 將 `IronMan.html` 的 3D 鋼鐵人 Mark IV 關節模型與英雄戰鬥系統深度整合進 `index.html`，作為防禦底線的藍軍領袖英雄指揮官，支援靈活移動開火、胸口集束砲 [E] 與微型導彈 [Q] 技能。

**Architecture:** 
在 `index.html` 中構建模組化 3D 鋼鐵人英雄模型層次（軀幹、面罩、反應爐、四肢、掌心脈衝發射器與推進噴嘴），採用純金屬 Standard PBR 材質與主題自適應自發光核心；替換原先底部的簡易圓柱砲台，綁定玩家左右位移與開火後座力動畫；並為藍軍加入專屬英雄技能系統（Unibeam 貫穿雷射光束、追蹤微型導彈）及桌機/行動端雙 HUD 按鈕控制。

**Tech Stack:** Three.js (0.160.0+ / 0.182.0), WebGL, HTML5 Canvas, Web Audio API, Vanilla JavaScript, CSS3.

## Global Constraints
- 嚴格遵守 `three-best-practices`：零迴圈內 GC 記憶體分配、單一 AnimationLoop、及時 dispose 幾何體與材質。
- 免外部圖片依賴（純 3D 金屬 PBR 裝甲 + Canvas 自發光材質），確保本機 `file://` 與離線環境 100% 正常運行零報錯。
- 嚴格維持原本藍軍增殖機制、倍增門運算、紅軍波次與 60 FPS 流暢度。

---

### Task 1: 建立鋼鐵人 3D 英雄模型層次與 PBR 金屬材質系統

**Files:**
- Modify: `D:\Programming\Game21\index.html` (CSS 樣式區、ThreeGameScene 區、實體構建區)

**Interfaces:**
- Produces: `createIronManHero(scene)` 傳回包含 `root`, `pelvisGroup`, `chestGroup`, `headGroup`, `leftArm`, `rightArm`, `leftLeg`, `rightLeg`, `reactorCore`, `flashSprites`, `unibeamGroup` 的物件結構。
- Consumes: Three.js 場景 `this.scene` 與底線座標。

- [ ] **Step 1: 在 CSS 中新增英雄技能 HUD 按鈕樣式**
在 `index.html` 的 `<style>` 區段新增 `#hero-skills-layer`, `.skill-btn`, `.skill-cd-mask` 樣式，支援桌機與行動端懸浮觸控。

- [ ] **Step 2: 在 JS 中構建高質感純金屬 PBR 材質庫 (`createIronManMaterials`)**
包含金屬深紅 `armorRed`、合金黃金 `armorGold`、碳纖黑鈦 `jointDark` 與動態發光 `arcGlow`。

- [ ] **Step 3: 移植並模組化 `createIronManHero()` 關節模型構建函數**
精確建立：
- 頭部：經典紅金戰盔、流線金色面罩 (Faceplate)、青藍狹縫目鏡 (Glowing Slit Eyes)。
- 胸腔：方舟反應爐 (Arc Reactor Core)、金色胸肌飾板、背部雙噴嘴與減速風翼。
- 雙臂：肩甲、上臂、手肘、前臂護腕、手掌與掌心脈衝節點 (Repulsor Node + Muzzle Flash Sprite)。
- 雙腿：金色大腿裝甲、膝部關節、小腿護脛、推進戰靴。
- 英雄面向前方 +Z 敵軍戰場，尺度等比例微調適配跑道（高度約 2.0 單位）。

- [ ] **Step 4: 將鋼鐵人英雄掛載至底線戰鬥座標**
在 `ThreeGameScene.createPlayerCannon()` 中，將原本圓柱替換為鋼鐵人專屬戰術基座與 `this.ironMan` 英雄本體，並在 `cannonGroup` 中同步管理。

- [ ] **Step 5: 驗證模型加載**
開啟頁面或執行驗證，確認畫面底線正確出現精細立體的 3D 鋼鐵人 Mark IV 英雄模型，無報錯。

- [ ] **Step 6: Commit Task 1**
```bash
git add index.html
git commit -m "feat: add 3D IronMan hero model and PBR materials to blue alliance"
```

---

### Task 2: 實作鋼鐵人開火運動學、後座力與多聯裝姿態切換

**Files:**
- Modify: `D:\Programming\Game21\index.html` (GameController 與動畫迴圈)

**Interfaces:**
- Produces: `updateIronManKinematics(dt, time, isFiring, streamCount)`
- Consumes: `this.cannonX`, `gameStorage.getStreamColumnCount()`

- [ ] **Step 1: 實作水平跟隨與懸浮動態**
鋼鐵人 `root.position.x` 實時平滑跟隨玩家的 `cannonX`；在待機與前進時，腳底推進器帶有微幅懸浮浮動（Bobbing）。

- [ ] **Step 2: 實作平舉瞄準與開火後座力 (Recoil)**
- 藍軍部隊發射時，鋼鐵人手掌上揚 90 度瞄準倍增門，掌心閃爍 Repulsor 光芒精靈。
- 加入 `recoilTimer`，開火時手臂微震後座，軀幹自然微仰並迅速回彈。

- [ ] **Step 3: 實作多聯裝姿態自適應**
- 當 `streamCount <= 2` 時，採用單手穩健平舉瞄準姿態。
- 當升級至 `streamCount >= 3`（3 顆或 4 顆並排齊射）時，自動切換為**雙手齊射姿態 (Dual-Repulsor Aiming)**，雙臂平舉兩側掌心同時亮起開火。

- [ ] **Step 4: 驗證開火與移動姿態**
進入戰鬥，滑動滑鼠/觸控，確認鋼鐵人左右靈活位移，連續開火藍軍時後座力流暢。

- [ ] **Step 5: Commit Task 2**
```bash
git add index.html
git commit -m "feat: implement IronMan kinematics, recoil, and dual-arm aiming stance"
```

---

### Task 3: 實作英雄主動專屬技能：胸口集束巨砲 [E] 與微型導彈 [Q]

**Files:**
- Modify: `D:\Programming\Game21\index.html` (技能系統與 HUD 介面)

**Interfaces:**
- Produces: `triggerUnibeam()`, `fireMicroMissiles()`, `updateHeroSkills(dt)`
- Consumes: `redEnemies`, `boss`, `soundFX`

- [ ] **Step 1: 新增英雄技能資料結構與冷卻計時器**
- `unibeam`: CD 12 秒，持續時間 1.2 秒。
- `missiles`: CD 8 秒，發射 6 枚追蹤導彈。

- [ ] **Step 2: 實作胸口集束砲 (Unibeam Blast)**
- 構建同軸能量光柱幾何體與多重旋轉光環。
- 觸發時鋼鐵人挺胸蓄力，發射貫穿全跑道能量光柱，消滅路徑上所有紅軍步兵並對 Boss 造成大量持續傷害。
- 增加全螢幕能量震顫與專屬音效反饋。

- [ ] **Step 3: 實作肩部微型導彈 (Micro-Missiles)**
- 構建 6 枚金屬微型導彈物件池。
- 發射時從肩甲彈射向上弧線飛行，並利用向量引導自動追蹤最近的紅軍與石像鬼，命中產生爆炸火花與濺射傷害。

- [ ] **Step 4: 綁定鍵盤熱鍵與行動端 HUD 技能按鈕**
- 監聽 `KeyE` (胸口砲) 與 `KeyQ` (微型導彈)。
- 在右下角 HUD 渲染兩個科幻圓形技能按鈕，顯示冷卻倒數遮罩與動態高亮。

- [ ] **Step 5: 驗證技能施放與冷卻**
按下 E 與 Q 鍵或點擊按鈕，確認集束砲貫穿清場、導彈追蹤爆炸，冷卻倒數正常。

- [ ] **Step 6: Commit Task 3**
```bash
git add index.html
git commit -m "feat: implement IronMan Unibeam and Micro-Missiles hero skills"
```

---

### Task 4: 主題動態適配、音效整合與端到端綜合驗收

**Files:**
- Modify: `D:\Programming\Game21\index.html`

**Interfaces:**
- Consumes: `THEME_PRESETS`, `SoundFX`

- [ ] **Step 1: 反應爐與目鏡光芒適配主題配色**
切換「電光競技 / 清爽晴空 / 賽博糖果」主題時，鋼鐵人胸口反應爐與目鏡的光芒自發光顏色同步切換為相應主題青藍/翠綠/薄荷色。

- [ ] **Step 2: 音效合成器擴充**
在 `SoundFX` 中加入專屬雷射脈衝音效、導彈呼嘯爆破音效與集束巨砲高能充能轟鳴。

- [ ] **Step 3: 綜合效能與功能驗收 (Verification Checklist)**
- 檢查控制台 Console 確保 0 報錯、0 警告。
- 檢查三維渲染 `Calls` 與 `FPS` 穩定於 60 幀。
- 測試全 10 關與無盡生存模式流程順暢。

- [ ] **Step 4: Commit Task 4**
```bash
git add index.html
git commit -m "feat: complete theme adaptation, audio fx, and end-to-end integration"
```
