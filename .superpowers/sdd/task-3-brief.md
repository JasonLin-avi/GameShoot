# Task 3 Brief: 實作英雄主動專屬技能：胸口集束巨砲 [E] 與微型導彈 [Q]

## Files
- Modify: `D:\Programming\Game21\index.html`

## Global Constraints
- 嚴格遵守 `three-best-practices`：零迴圈內 GC 記憶體分配、單一 AnimationLoop、及時 dispose 幾何體與材質。
- 免外部圖片依賴（純 3D 金屬 PBR 裝甲 + Canvas 自發光材質），確保本機 `file://` 與離線環境 100% 正常運行零報錯。
- 嚴格維持原本藍軍增殖機制、倍增門運算、紅軍波次與 60 FPS 流暢度。

## Detailed Requirements

1. **技能資料結構與冷卻計時器 (GameController)**
在 `GameController` 建構子中初始化：
```javascript
this.heroSkills = {
    unibeam: {
        cooldown: 12.0,
        timer: 0,
        duration: 1.2,
        activeTimer: 0,
        active: false
    },
    missiles: {
        cooldown: 8.0,
        timer: 0
    }
};
```
- 每幀在 `GameController.loop()` 中根據 `dt` 倒數：
  - `this.heroSkills.unibeam.timer = Math.max(0, this.heroSkills.unibeam.timer - dt)`
  - `this.heroSkills.missiles.timer = Math.max(0, this.heroSkills.missiles.timer - dt)`
- 更新 HUD 冷卻狀態（已在 HTML/CSS 中備妥 `#btn-skill-unibeam`, `#cd-mask-unibeam`, `#cd-text-unibeam`, `#btn-skill-missiles`, `#cd-mask-missiles`, `#cd-text-missiles`）：
  - 冷卻中：按鈕加上 `.disabled` 樣式，遮罩依 `timer / cooldown` 顯示高度百分比，顯示秒數如 `3.5s`。
  - 冷卻完成：移除遮罩，按鈕高亮，文字清空。

2. **胸口集束砲 (Unibeam Blast)**
- **3D 幾何體與材質**：
  - 長圓柱體光柱 (`CylinderGeometry(0.5, 0.5, 75, 16)`，旋轉 `rotateX(Math.PI/2)`)，位置跟隨鋼鐵人胸口，沿 +Z 貫穿整個戰場。
  - 材質採用高亮青藍能量光 (`MeshBasicMaterial({ color: 0x00f0ff, transparent: true, opacity: 0 })`)。
  - 光柱外圍附帶 4~6 個旋轉同軸光環 (`TorusGeometry(0.8, 0.05, 8, 20)`)。
- **觸發 `triggerUnibeam()`**：
  - 若 `this.state !== 'PLAYING'` 或 `timer > 0` 則忽略。
  - 設定 `active = true`, `activeTimer = 1.2`, `timer = 12.0`。
  - 鋼鐵人姿態：胸部挺起 (`chestGroup.rotation.x = 0.22`)，方舟反應爐自發光增強。
  - 觸發畫面強震 `this.shakeAmount = 0.6`。
  - 播放音效 `soundFX.playBossWarning()` 或高能雷射充能音效。
- **貫穿傷害解算 (Collision Loop)**：
  - 當 `active` 為 true 時：
    - 光柱沿 +Z 方向平移跟隨 `this.cannonX`。
    - 路徑上所有紅軍步兵 (`Math.abs(r.x - this.cannonX) < 1.3 && r.z > 0`) 瞬間蒸發消滅，觸發粒子爆破並增加擊殺與金幣。
    - 石像鬼受到重創。
    - 若 Boss 處於射線走道內，每秒承受持續重創（每幀扣減 `50 * dt` 血量並擊退），觸發劇烈火花。

3. **肩部微型追蹤導彈 (Micro-Missiles)**
- **導彈物件池 (`missilePool`)**：
  - 預先配置 12 枚微型導彈（`CylinderGeometry(0.05, 0.06, 0.4, 8)`，金屬金/紅 Standard/Basic 材質），預設 `visible = false`。
  - 欄位包含 `active, x, y, z, vx, vy, vz, targetEnemy, life`。
- **觸發 `fireMicroMissiles()`**：
  - 若 `this.state !== 'PLAYING'` 或 `timer > 0` 則忽略。
  - 設定 `timer = 8.0`。
  - 從左右雙肩各發射 3 枚（共 6 枚）導彈：
    `spawnPos = (this.cannonX ± 0.38, y = 1.6, z = 0.3)`。
    初始向量向上與向前：`vx = (random - 0.5) * 3.0, vy = 6.0 + random * 3.0, vz = 10.0 + random * 4.0`。
  - 導彈自動搜尋前方最近的活動敵軍（優先鎖定 Boss，其次石像鬼，其次紅軍步兵）。
- **推進與命中解算**：
  - 每幀導彈向量以 `lerp` 平滑轉向目標位置 (`dt * 7.5`)。
  - 當接近目標或碰撞時 (`distanceSq < 0.8`)：
    - 觸發 `entityMgr.spawnExplosion(m.x, m.y, m.z, 6)`。
    - 目標受到 30 點爆破傷害（若為小兵則直接消滅，Boss 大幅失血）。
    - 播放爆破音效 `soundFX.playHit()`。
    - 導彈重設為非活動狀態。

4. **鍵盤熱鍵與行動端 HUD 綁定**
- 監聽 `window.addEventListener('keydown')`：
  - `KeyE` / `e` / `E`：觸發 `triggerUnibeam()`。
  - `KeyQ` / `q` / `Q`：觸發 `fireMicroMissiles()`。
- 監聽按鈕點擊：
  - `#btn-skill-unibeam` 點擊/pointerdown 觸發 `triggerUnibeam()`。
  - `#btn-skill-missiles` 點擊/pointerdown 觸發 `fireMicroMissiles()`。
- 更新操作提示文字：在 `#drag-guide` 或控制引導中標示 `[E] 胸口集束砲 | [Q] 追蹤導彈`。

5. **驗證與測試**
- 運行 node 語法驗證。
- 確認施放胸口砲與微型導彈時無死循環或 NaN，冷卻計時精確。
- 提交代碼：`git commit -m "feat: implement IronMan Unibeam and Micro-Missiles hero skills"`
