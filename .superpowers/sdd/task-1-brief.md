# Task 1 Brief: 建立鋼鐵人 3D 英雄模型層次與 PBR 金屬材質系統

## Files
- Modify: `D:\Programming\Game21\index.html`

## Global Constraints
- 嚴格遵守 `three-best-practices`：零迴圈內 GC 記憶體分配、單一 AnimationLoop、及時 dispose 幾何體與材質。
- 免外部圖片依賴（純 3D 金屬 PBR 裝甲 + Canvas 自發光材質），確保本機 `file://` 與離線環境 100% 正常運行零報錯。
- 嚴格維持原本藍軍增殖機制、倍增門運算、紅軍波次與 60 FPS 流暢度。

## Detailed Requirements

1. **CSS 英雄技能 HUD 按鈕樣式**
在 `index.html` 的 `<style>` 中加入：
- `#hero-skill-bar`：固定在右下方（或靠近操作區），具備半透明毛玻璃背景、圓角與陰影。
- `.hero-skill-btn`：圓形或科幻圓角按鈕，包含技能圖示（⚡ 集束砲 / 🚀 導彈）、熱鍵標籤（E / Q）、冷卻遮罩 `.cd-overlay` 與冷卻倒數數字 `.cd-text`。

2. **高質感純金屬 PBR 材質庫 (`createIronManMaterials`)**
在 `index.html` 的 `<script type="module">` 內定義：
- `armorRed`：`new THREE.MeshStandardMaterial({ color: 0xb91c1c, roughness: 0.28, metalness: 0.85 })`
- `armorGold`：`new THREE.MeshStandardMaterial({ color: 0xf59e0b, roughness: 0.24, metalness: 0.90 })`
- `jointDark`：`new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.55, metalness: 0.70 })`
- `arcGlow`：`new THREE.MeshBasicMaterial({ color: 0x00f0ff })` (青藍發光，供反應爐、目鏡與掌心 Repulsor 使用)

3. **`createIronManHero(materials)` 關節模型構建函數**
參考 `IronMan.html` 中的結構，建構階層式 3D 鋼鐵人（等比例適配跑道，總高度約 2.0 單位）：
- `root` (THREE.Group)：面朝戰場前方 +Z（`root.rotation.y = 0`，因為在 index.html 中跑道是沿 +Z 向前延伸！請確保鋼鐵人正面朝向 +Z 跑道與倍增門，背面朝向相機 -Z）。
- `pelvisGroup`：骨盆裝甲。
- `spineGroup` & `chestGroup`：
  - 胸甲基礎與胸肌飾板（紅金相間）。
  - 方舟反應爐 `reactorGroup`：金屬固定環 + 內部發光核心 `reactorCore` (使用 `arcGlow`)。
  - 背部推進噴嘴 (雙噴嘴) 與飛行擾流片。
- `neckGroup` & `headGroup`：
  - 頭盔紅金球體，標誌性金色面罩 (Faceplate)，下巴突出部。
  - 雙眼青藍色發光細縫目鏡 (Glowing Slit Eyes)。
  - 耳部金色圓盤飾蓋。
- `leftArm` & `rightArm`：
  - 肩甲 (Pauldron)、上臂、手肘、前臂護腕 (Gauntlet)。
  - 手掌 (Palm) 與掌心圓形發光脈衝節點 (Repulsor Node, 使用 `arcGlow`)。
  - 掌心槍口閃光精靈 (Muzzle Flash Sprite, AdditiveBlending，平時 visible = false)。
- `leftLeg` & `rightLeg`：
  - 大腿金色護甲與外側護板。
  - 膝蓋金色護蓋與小腿護脛。
  - 腳踝與推進戰靴，鞋底噴氣孔。
- 傳回完整的物件參照字典：
  `{ root, pelvisGroup, chestGroup, headGroup, leftArm, rightArm, leftLeg, rightLeg, reactorCore, flashSprites: [leftArm.flashSprite, rightArm.flashSprite] }`

4. **掛載於 `ThreeGameScene.createPlayerCannon()`**
- 在 `ThreeGameScene.createPlayerCannon()` 中：
  - 保留並優化底座（科技戰術圓盤），將鋼鐵人 `this.ironMan = createIronManHero(this.ironManMaterials)` 掛載於 `this.cannonGroup`。
  - 移除原先單調的圓柱砲管（或將其整合至基座隱蔽），讓鋼鐵人成為藍軍戰術先鋒與核心視覺焦點。
  - 調整整體縮放 scale 與 y 軸位置（讓鋼鐵人雙腳踩在地面 y=0 或戰術基座頂部 y=0.15）。

5. **HTML UI 元素**
在 `#ui-layer` 內部加入技能 HUD DOM 節點：
```html
<div id="hero-skill-bar">
    <button class="hero-skill-btn" id="btn-skill-unibeam" title="胸口集束巨砲 [E]">
        <div class="skill-icon">⚡</div>
        <div class="skill-key">E</div>
        <div class="skill-cd-mask" id="cd-mask-unibeam"></div>
        <div class="skill-cd-text" id="cd-text-unibeam"></div>
    </button>
    <button class="hero-skill-btn" id="btn-skill-missiles" title="肩部微型導彈 [Q]">
        <div class="skill-icon">🚀</div>
        <div class="skill-key">Q</div>
        <div class="skill-cd-mask" id="cd-mask-missiles"></div>
        <div class="skill-cd-text" id="cd-text-missiles"></div>
    </button>
</div>
```

6. **驗證與測試**
- 檢查代碼無語法錯誤，HTML 與 JS 正確解析。
- 運行測試或透過瀏覽器確認控制台 0 報錯，底線出現完整立體鋼鐵人。
- 提交代碼：`git commit -m "feat: add 3D IronMan hero model and PBR materials to blue alliance"`
