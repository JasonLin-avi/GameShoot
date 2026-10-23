# Task 2 Brief: 實作鋼鐵人開火運動學、後座力與多聯裝姿態切換

## Files
- Modify: `D:\Programming\Game21\index.html`

## Global Constraints
- 嚴格遵守 `three-best-practices`：零迴圈內 GC 記憶體分配、單一 AnimationLoop、及時 dispose 幾何體與材質。
- 免外部圖片依賴（純 3D 金屬 PBR 裝甲 + Canvas 自發光材質），確保本機 `file://` 與離線環境 100% 正常運行零報錯。
- 嚴格維持原本藍軍增殖機制、倍增門運算、紅軍波次與 60 FPS 流暢度。

## Detailed Requirements

1. **實作 `updateIronManKinematics(dt, time, isFiring, streamCount)` 函數或方法**
可在 `ThreeGameScene` 中定義，或作為模組函數，每幀於 `GameController.loop()` 渲染前呼叫：
```javascript
this.threeScene.updateIronManKinematics(dt, time, isFiring, streamCount);
```
參數說明：
- `dt`：幀間隔 Delta Time（例如 0.016s）。
- `time`：累積總時間（用於正弦浮動計算）。
- `isFiring`：布林值，表示當前幀是否剛剛觸發開火（發射藍軍小兵）。
- `streamCount`：當前發射彈道數量（來自 `gameStorage.getStreamColumnCount()`，2 / 3 / 4 聯裝）。

2. **水平跟隨與懸浮推進微浮動**
- 鋼鐵人跟隨 `this.cannonGroup.position.x = this.cannonX`。
- 在地面移動或懸浮時，加入微妙的推進正弦呼吸浮動（Bobbing）：
  `ironMan.root.position.y = 0.05 + Math.sin(time * 4.0) * 0.03;`
- 微小轉彎傾角（Banking）：當移動速度較快時，軀幹有微微的 Z 軸旋轉傾斜，靜止時平滑回正。

3. **平舉瞄準與開火後座力 (Recoil)**
- 鋼鐵人面向前方 +Z（跑道方向）。
- 當開火時 (`isFiring === true`)：
  - 重設 `recoilTimer = 0.12;`
  - 觸發槍口閃光精靈閃爍（`flashSprite.visible = true`，持續約 0.05 秒後自動消失）。
- 在後座力計時中：
  - 手臂與胸部軀幹產生向後微小後座：
    `const kick = Math.sin(recoilTimer * 40.0) * 0.10;`
    `arm.shoulderPivot.rotation.x -= kick;`
    `ironMan.chestGroup.rotation.x -= kick * 0.4;`
  - 伴隨 `recoilTimer -= dt` 衰減，平滑歸零。

4. **多聯裝姿態切換 (Single vs Dual-Arm Stance)**
- 當 `streamCount <= 2`（雙聯裝）：
  - 鋼鐵人右手向前平舉瞄準 +Z：
    `rightArm.shoulderPivot.rotation.set(-1.45, 0, 0);`
    `rightArm.elbowPivot.rotation.set(-0.15, 0, 0);`
    `rightArm.handGroup.rotation.set(Math.PI / 2, 0, 0);`
  - 左手呈自然微屈戒備姿態：
    `leftArm.shoulderPivot.rotation.set(0.18, 0, 0.28);`
    `leftArm.elbowPivot.rotation.set(-0.45, 0, -0.15);`
    `leftArm.handGroup.rotation.set(0, 0, 0);`
  - 閃爍右手閃光精靈。
- 當 `streamCount >= 3`（3 聯裝或 4 聯裝高階齊射）：
  - 自動啟用**雙手齊射姿態 (Dual-Repulsor Aiming)**：
  - 左手與右手對稱向前平舉瞄準 +Z，雙手手掌均上揚 90 度：
    `leftArm.shoulderPivot.rotation.set(-1.45, 0, 0.08);`
    `leftArm.handGroup.rotation.set(Math.PI / 2, 0, 0);`
    `rightArm.shoulderPivot.rotation.set(-1.45, 0, -0.08);`
    `rightArm.handGroup.rotation.set(Math.PI / 2, 0, 0);`
  - 雙手閃光精靈同時同步閃爍，視覺反饋強烈。

5. **與 `GameController.loop()` 發射邏輯串接**
- 當 `time - this.lastShootTime >= fireInterval` 時，在 `spawnBlueTroop()` 同時標記 `isFiring = true`。
- 在每幀的 `updateIronManKinematics(dt, time, isFiring, streamCount)` 中傳入此狀態。
- 在大廳或非 PLAYING 狀態時，鋼鐵人保持自然的英雄待機姿勢。

6. **驗證與測試**
- 執行語法與運行檢查，確認運動學解算旋轉度數方向正確，不反向。
- 確保在控制台沒有任何 NaN 或報錯。
- 提交代碼：`git commit -m "feat: implement IronMan kinematics, recoil, and dual-arm aiming stance"`
