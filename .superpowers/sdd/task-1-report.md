# Task 1 Report: 建立鋼鐵人 3D 英雄模型層次與 PBR 金屬材質系統

- **執行日期**：2026-10-18
- **對應檔案**：`D:\Programming\Game21\index.html`
- **提交雜湊**：`34de375`
- **提交訊息**：`feat: add 3D IronMan hero model and PBR materials to blue alliance`

---

## 1. 成果摘要 (Summary of Accomplishments)

1. **高質感純金屬 PBR 材質庫 (`createIronManMaterials`)**：
   - `armorRed`：`MeshStandardMaterial` (紅色金屬裝甲, `color: 0xb91c1c, roughness: 0.28, metalness: 0.85`)。
   - `armorGold`：`MeshStandardMaterial` (經典金面罩與護甲, `color: 0xf59e0b, roughness: 0.24, metalness: 0.90`)。
   - `jointDark`：`MeshStandardMaterial` (鈦金屬深灰關節, `color: 0x1f2937, roughness: 0.55, metalness: 0.70`)。
   - `arcGlow`：`MeshBasicMaterial` (方舟反應爐、發光目鏡與掌心 Repulsor, `color: 0x00f0ff`)。
   - 免外部貼圖依賴，支援本機 `file://` 與離線環境零報錯。

2. **鋼鐵人階層式 3D 關節模型 (`createIronManHero`)**：
   - 樹狀骨骼結構：`root` (朝向前方 +Z 跑道) -> `pelvisGroup` -> `spineGroup` -> `chestGroup` -> `neckGroup` & `headGroup`，左右手肘腕掌 `leftArm`/`rightArm`，大腿小腿靴底 `leftLeg`/`rightLeg`。
   - 背部視角強化：包含背部飛行擾流片 (`leftFlap`/`rightFlap`)、脊椎散熱片與雙推進噴嘴。
   - 掌心 Repulsor 與槍口閃光精靈 (`Sprite` + 動態 Canvas 放射漸層貼圖，`AdditiveBlending`)。
   - 縮放比例：等比例適配 11 單位跑道，整體高度約 2.08 單位，採英雄備戰姿態。
   - 完整返回參考字典：`{ root, pelvisGroup, spineGroup, chestGroup, neckGroup, headGroup, leftArm, rightArm, leftLeg, rightLeg, reactorCore, flashSprites, backFlames, leftFlap, rightFlap }`。

3. **科技底盤優化與英雄掛載 (`ThreeGameScene.createPlayerCannon`)**：
   - 升級為科技戰術部署圓盤 (`baseGeo`) 搭配青藍霓虹光環 (`ringGeo`)。
   - 鋼鐵人模型掛載至 `this.cannonGroup`，雙腳穩立於基座表面 (`y = 0.20`)。
   - `updateCannonBarrels` 移除原先穿透鋼鐵人身體的巨大圓柱砲管，改為底座前緣隱蔽整合的微型能量發射埠，保留多聯裝升級的視覺反饋。

4. **英雄技能 HUD 樣式與 HTML**：
   - 在 `#ui-layer` 加入技能快捷鍵按鈕：胸口集束砲 [E] 與肩部微型導彈 [Q]。
   - 配備毛玻璃科幻圓角、按鈕懸停縮放微光、冷卻遮罩 `.skill-cd-mask` 與倒數數字 `.skill-cd-text`。

---

## 2. 測試與驗證 (Verification & Testing)

- **語法檢查**：使用 Node.js 解析 ES 模組代碼，確認 0 語法錯誤。
- **Three.js 規範**：零迴圈 GC 記憶體分配，`updateCannonBarrels` 保留幾何體 dispose 清理機制。
- **主題相容性**：`applyTheme` 支援切換主題配色而不影響鋼鐵人金屬質感。

---

## 3. 後續任務接口 (Handoff to Task 2)

- 鋼鐵人部位參考可在 `this.threeScene.ironMan` 直接取得。
- `ironMan.leftArm.shoulderPivot` / `ironMan.rightArm.shoulderPivot` / `ironMan.leftArm.handGroup` / `ironMan.flashSprites` 可直接供 Task 2 實現雙掌開火抬手瞄準與後座力回彈動畫。
