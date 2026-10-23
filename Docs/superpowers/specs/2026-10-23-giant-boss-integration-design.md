# 綠巨人整合進《軍團防線》巨型頭目 (Giant Boss) 設計規範

- **日期**：2026-10-23
- **項目**：Game21 - `index.html` (軍團防線：增殖衝擊) & `giant.html`
- **狀態**：已批准 (Approved Design Spec)

---

## 1. 概述與設計目標 (Overview & Objectives)

將 `giant.html` 中高規格、純程序化有機曲面建模的 **綠巨人 (Giant Boss)** 3D 骨骼模型與生物力學行進動作系統，整合進 `index.html`《軍團防線：增殖衝擊》中，替代原本簡約的圓柱體紅軍頭目，擔任關卡中的**巨型空降頭目 (Red Boss)**。

### 核心設計亮點
1. **升級原版簡約圓柱 Boss**：替換原先由圓柱與角錐組成的幾何 Boss，升級為具備完整肌肉細節、紫褲、黑髮與發怒神情的 3.8 單位高精細綠巨人 3D 角色。
2. **純程序化 PBR 高效材質**：直接複用 `giant.html` 的純程序化材質方案（草綠肌肉主色、暗部肌肉陰影、受光高光、破損深紫戰褲、黑髮白牙），杜絕外部紋理加載失敗與 CORS 限制。
3. **沉重行進步態與踏步震波 (Walk Cycle & Stomp Waves)**：
   - 雙足與雙臂依步態週期交替擺動。
   - 雙腳踩地時，自物件池中生成擴散光環並觸發畫面微震，極具戰場壓迫感。
4. **殘血狂暴怒吼 (Enrage & Roar Mechanic)**：
   - 當巨人血量首次低於 30% 時，觸發狂暴咆哮動作（雙臂沖天張開咆哮），隨後行進速度提升 30%，迫使玩家集中火力消滅。
5. **完整相容原有遊戲與英雄技能**：
   - 完美相容 10 個關卡波次與無盡模式。
   - 完美承接藍軍小兵衝撞、鋼鐵人 [E] 胸口集束砲 (Unibeam) 貫穿傷害，以及 [Q] 肩部微型導彈的自動追蹤打擊。

---

## 2. 系統架構與組件設計 (Architecture & Components)

### 2.1 巨人 3D 階層式模型架構 (`GiantBossSystem`)
於 `index.html` 構建模組化 Group 結構：
- `root`：整體頭目座標節點（綁定 `this.boss.x`, `this.boss.y`, `this.boss.z`，面朝玩家方向 -Z，高度縮放比例約為 1.0~1.15）。
  - `pelvisGroup`：骨盆與深紫破損短褲腰帶。
    - `spineGroup` & `chestGroup`：軀幹腰椎、外凸胸大肌、高聳斜方肌、八塊腹肌與前鋸肌。
      - `neckGroup` & `headGroup`：粗壯頸肌、下顎咬牙齒、狂暴怒意雙目與層次黑髮。
      - `leftArm` & `rightArm`：肩部三角肌、肱二頭肌、粗壯前臂與巨大拳頭。
    - `leftLeg` & `rightLeg`：大腿股四頭肌、膝關節、小腿腓腸肌與寬厚腳掌。
  - `bossHpSprite`：掛載於頭頂 (Y = 4.2) 的即時動態血條。

### 2.2 材質與色彩系統 (Materials)
遵循 `three-best-practices` 的 `material-reuse` 準則，單例複用下列標準材質：
- `skinMain`：`THREE.MeshStandardMaterial`，草綠膚色 (`0x6ea33b`, roughness: 0.56, metalness: 0.04)。
- `skinDark`：`THREE.MeshStandardMaterial`，肌肉凹槽陰影 (`0x4e7926`, roughness: 0.65, metalness: 0.04)。
- `skinLight`：`THREE.MeshStandardMaterial`，肌肉受光高光 (`0x7ab044`, roughness: 0.50, metalness: 0.04)。
- `pantsPurple`：`THREE.MeshStandardMaterial`，破損紫褲 (`0x4e2a50`, roughness: 0.72, metalness: 0.06)。
- `pantsDark`：`THREE.MeshStandardMaterial`，腰帶皺褶 (`0x371b39`, roughness: 0.78, metalness: 0.06)。
- `hairBlack`：`THREE.MeshStandardMaterial`，黑髮濃眉 (`0x1c201d`, roughness: 0.78)。
- `teethWhite`：`THREE.MeshStandardMaterial`，牙齒與眼白 (`0xf8fafc`, roughness: 0.25)。
- `eyePupil`：`THREE.MeshBasicMaterial`，深邃瞳孔 (`0x0f172a`)。

### 2.3 運動學解算與狀態機 (Kinematics & State Machine)
- **一般行進狀態 (`WALKING`)**：
  - `boss.vz` 預設為 -2.2。
  - 步態相位角 `walkPhase += dt * walkSpeed * 4.2`。
  - 大腿、膝關節與腳掌依正弦曲線交替踩踏；雙臂與對側腿部反向前後擺盪。
  - 踏地判定（相角臨界點）觸發地面震波環 `spawnStompRing` 與鏡頭微震。
- **狂暴咆哮狀態 (`ROAR`)**：
  - 血量初次小於 `maxHp * 0.3` 觸發，持續 2 秒。
  - 雙臂大幅向兩側與上方揚起屈肘，挺胸仰頭。
  - 咆哮結束後，速度提升至 `boss.vz = -2.86`（加速 30%）。
- **受擊與擊退反饋**：
  - 受到藍軍衝撞、Unibeam 或微型導彈攻擊時，產生受擊火花與微擊退 (+Z 位移)。

---

## 3. 踏步震波物件池 (Stomp Shockwave Object Pool)

- 預先配置 8 個擴散環 (`RingGeometry(0.2, 0.45, 28)`)，水平放置於戰場地面（Y = 0.02）。
- 材質採用 `MeshBasicMaterial`（透明度動畫由 0.7 衰減至 0，尺寸由 1.0 擴散至 2.4）。
- 循環複用，不產生任何執行階段物件分配。

---

## 4. 戰鬥系統整合與驗證標準 (Verification Criteria)

1. **視覺驗收**：
   - 關卡生成 Boss 時，戰場上方出現雄壯霸氣的綠巨人 3D 模型，頂部血條位置精準。
   - 雙腿沉重向前踩踏、雙臂前後擺動，地面隨步調產生光環衝擊波。
2. **戰鬥機制驗收**：
   - 藍軍小兵與 Boss 碰撞時正常扣除 Boss 血量、產生擊中特效與微擊退。
   - 鋼鐵人 [E] 集束砲貫穿打擊巨人，造成持續巨量傷害與後推。
   - 鋼鐵人 [Q] 微型導彈優先鎖定巨人並引爆。
   - 血量低於 30% 時，成功觸發狂暴咆哮姿態並提速壓境。
   - 擊殺巨人後，觸發巨型爆炸衝擊波、掉落 50 金幣，模型正確隱藏與重置。
3. **效能驗收**：
   - 戰鬥全過程維持 60 FPS，無記憶體洩漏與垃圾回收卡頓。
