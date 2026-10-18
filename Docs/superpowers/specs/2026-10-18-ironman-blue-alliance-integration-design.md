# 鋼鐵人整合進《軍團防線》藍軍英雄指揮官設計規範

- **日期**：2026-10-18
- **項目**：Game21 - `index.html` (軍團防線：增殖衝擊) & `IronMan.html`
- **狀態**：設計規劃階段 (Brainstorming & Design Spec)

---

## 1. 概述與設計目標 (Overview & Objectives)

將 `IronMan.html` 中高規格的 **鋼鐵人 Mark IV (藍軍戰場先鋒)** 3D 關節模型、戰甲材質與英雄戰鬥系統，深度整合進 `index.html`《軍團防線：增殖衝擊》中，擔任**藍軍領袖英雄指揮官**。

### 核心變更點
1. **替換單調砲台**：將原本底線 Z=0 處簡易的圓柱形砲台，升級為具備高精細結構的 3D 鋼鐵人 Mark IV 英雄模型。
2. **純金屬高質感 PBR 材質**：採用免圖片載入的高效能 PBR 金屬材質（經典紅金裝甲 + 自發光方舟反應爐與目鏡），保證 60 FPS 極致流暢度，杜絕貼圖加載失敗與 CORS 限制。
3. **靈動開火姿態與後座力**：隨玩家觸控滑動或 A/D 鍵平滑橫移，左右手掌心脈衝砲隨多聯裝升級靈活切換單/雙手齊射姿態，發射藍軍增殖部隊。
4. **藍軍英雄專屬技能**：
   - **[E] 鍵 / 技能按鈕**：胸口集束砲 (Unibeam)，貫穿全跑道造成高額穿透傷害。
   - **[Q] 鍵 / 技能按鈕**：肩部微型追蹤導彈 (Micro-Missiles)，自動鎖定來襲紅軍與 Boss。
5. **雙端支援 (Desktop + Mobile)**：鍵盤快捷鍵 + 觸控 HUD 技能按鈕，手機與桌機均可暢快體驗。

---

## 2. 系統架構與組件設計 (Architecture & Components)

### 2.1 鋼鐵人 3D 英雄模型層次 (`IronManHero`)
依據 `IronMan.html` 之關節結構，於 `index.html` 構建模組化 Group：
- `root`：整體英雄座標節點（綁定 `this.cannonX`, Y=0, Z=0，面朝戰場前方 +Z）。
  - `pelvisGroup`：骨盆與戰術胯部裝甲。
    - `spineGroup` & `chestGroup`：胸甲總成、金色胸肌護板、背部推進噴嘴與散熱擾流翼。
      - `reactorGroup`：方舟反應爐核心 (Arc Reactor Core)，具備主題配色之高亮度自發光。
      - `neckGroup` & `headGroup`：紅金戰盔、流線金色面罩 (Faceplate)、發光狹縫目鏡 (Glowing Slit Eyes) 與耳部圓盤。
      - `leftArm` & `rightArm`：肩甲 (Pauldron)、上臂、手肘、前臂護腕 (Gauntlet)、手掌 (Palm) 與掌心脈衝雷射節點 (Repulsor Node + Muzzle Flash Sprite)。
    - `leftLeg` & `rightLeg`：大腿股四頭肌金屬甲、膝部活塞、小腿護脛、推進戰靴 (Boots) 與底噴氣孔。

### 2.2 材質與色彩系統 (Materials)
遵循 Three.js 最佳實踐 (`three-best-practices`)，全面複用共用材質：
- `armorRed`：`THREE.MeshStandardMaterial`，金屬暗紅色 (`0xb91c1c`, roughness: 0.28, metalness: 0.85)。
- `armorGold`：`THREE.MeshStandardMaterial`，鈦金合金黃金色 (`0xf59e0b`, roughness: 0.24, metalness: 0.90)。
- `jointDark`：`THREE.MeshStandardMaterial`，關節碳纖黑鈦色 (`0x1f2937`, roughness: 0.55, metalness: 0.70)。
- `arcGlow`：`THREE.MeshBasicMaterial`，青藍色超能光 (`0x00f0ff`)，可隨遊戲配色主題（電光競技/清爽晴空/賽博糖果）自適應切換。

### 2.3 英雄開火與運動學解算 (`HeroKinematics`)
- **待機與衝刺位移**：
  - 鋼鐵人水平位置隨 `this.cannonX` 柔和追蹤（`lerp` 平滑插值）。
  - 跑動時帶有微幅懸浮推進波紋（腳底/背部隱約噴氣藍光）。
- **射擊姿態與後座力**：
  - 發射藍軍時，左臂（或雙臂）平舉瞄準前方的倍增門，手掌上揚 90 度亮起掌心 Repulsor 節點。
  - 每次齊射觸發微後座力震動 (`recoilTimer`)，軀幹微傾後迅速回彈。
  - 多聯裝彈道（Lv.1~3 雙聯裝、Lv.4+ 四聯裝）時切換為雙臂齊射姿態。

### 2.4 技能系統 (Hero Ultimate Abilities)
1. **胸口集束砲 (Unibeam)**：
   - 快捷鍵：`[E]` / 畫面右下 UI 按鈕。
   - 冷卻時間：12 秒。
   - 效果：鋼鐵人挺胸蓄力，胸口方舟反應爐釋放貫穿整條跑道的光柱，消滅路徑上的紅軍步兵，並重創石像鬼與 Boss。
2. **肩部微型追蹤導彈 (Micro-Missiles)**：
   - 快捷鍵：`[Q]` / 畫面右下 UI 按鈕。
   - 冷卻時間：8 秒。
   - 效果：肩甲彈艙升起，朝空中發射 6 枚金屬微型導彈，劃出弧線自動追蹤轟炸最近的敵軍。

---

## 3. UI 與操作介面設計 (HUD & Controls)

1. **戰鬥 HUD 技能列**：
   - 在戰鬥畫面右側或右下方增加 2 個科幻風格圓形技能按鈕：
     - `⚡ [E] 集束砲`（附帶冷卻 CD 扇形遮罩/倒數）。
     - `🚀 [Q] 微型導彈`（附帶冷卻 CD 倒數）。
2. **頂部 HUD 標題與引導更新**：
   - 提示資訊加入 `[E] 集束砲`、`[Q] 導彈` 鍵位提示。
   - 大廳升級頁同步顯示鋼鐵人英雄屬性資訊。

---

## 4. 效能與最佳化規劃 (Performance & three-best-practices)

1. **零垃圾回收負擔 (Zero Allocations in Loop)**：
   - 動畫計算複用全域靜態 `_tempVec3`, `_tempMatrix`, `_dummy`。
2. **單一主動畫迴圈**：
   - 鋼鐵人動畫解算整合至現有 `renderer.setAnimationLoop` 中，以 `dt` (Delta Time) 平滑更新。
3. **記憶體釋放**：
   - 幾何體與材質在銷毀/重新載入時徹底 `dispose()`。
4. **維持藍軍 Swarm 萬人衝撞高格率**：
   - 藍軍部隊保持極高效的 `InstancedMesh(CapsuleGeometry)`，鋼鐵人僅作為單一英雄本體（Draw Call 增量小於 15），絲毫不影響千人戰場幀率。

---

## 5. 驗證與驗收標準 (Verification Criteria)

1. **視覺驗收**：
   - 進入戰鬥後，底線出現完整精美的鋼鐵人 Mark IV 英雄模型。
   - 鋼鐵人面向前方戰場，左右移動響應絲滑。
   - 開火時掌心光芒閃爍、手部後座力自然。
2. **技能驗收**：
   - 按 `E` 發射胸口砲，光柱貫穿跑道消滅敵軍並進入冷卻。
   - 按 `Q` 發射導彈，導彈弧線追蹤敵軍並引爆。
   - 手機觸控點擊技能圖示正常觸發。
3. **整合相容性驗收**：
   - 原有遊戲邏輯（倍增門乘除加減、小兵碰撞、Boss 戰、主題風格切換、升級養成）完全正常運作。
   - 控制台零報錯，FPS 穩定在 60 幀。
