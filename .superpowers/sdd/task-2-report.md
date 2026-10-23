# Task 2 Report: 實作鋼鐵人開火運動學、後座力與多聯裝姿態切換

- **執行日期**：2026-10-18
- **對應檔案**：`D:\Programming\Game21\index.html`
- **提交雜湊**：`3536c5c`
- **提交訊息**：`feat: implement IronMan kinematics, recoil, and dual-arm aiming stance`

---

## 1. 成果摘要 (Summary of Accomplishments)

1. **懸浮微浮動與轉彎側傾 (Bobbing & Banking)**：
   - 推進正弦浮動：`ironMan.root.position.y = 0.20 + Math.sin(time * 4.0) * 0.03`，雙腳懸浮於戰術基座上方，呼吸律動自然流暢。
   - 轉彎側傾解算：根據砲台水平速度 `vx = (currentX - lastCannonX) / dt` 動態計算傾角 `targetBank = -vx * 0.035`，平滑插值到 `ironMan.root.rotation.z`。向右移動身體向右傾，向左移動身體向左傾，靜止或處於非 PLAYING 狀態時平滑回正。

2. **平舉瞄準與開火後座力 (Recoil & Flash)**：
   - 當發射藍軍小兵 (`isFiring === true`) 時觸發開火運動：
     - 重置 `recoilTimer = 0.12`，`flashTimer = 0.05`。
     - 掌心 Repulsor 槍口閃光精靈啟用：`flashSprite.visible = true`，伴隨時間進行動態爆炸縮放並在 0.05 秒後自動隱藏。
     - 後座力解算：`kick = Math.max(0, Math.sin(recoilTimer * 40.0)) * 0.10`，手臂向後上微後座、胸腔後仰 `kick * 0.4`，隨時間衰減平滑歸零。

3. **多聯裝姿態切換 (Single vs Dual-Arm Stance)**：
   - **雙聯裝 (`streamCount <= 2`)**：右手向前平舉瞄準 +Z 跑道（`shoulderPivot` 旋轉 -1.45、手掌上揚 `Math.PI / 2` 瞄準前方），左手呈自然微屈防護戒備姿態；僅閃爍右手掌心閃光。
   - **多聯裝 (`streamCount >= 3`)**：切換為**雙手齊射姿態 (Dual-Repulsor Aiming)**，雙臂對稱平舉瞄準前方 +Z、雙手掌心同時上揚 90 度；雙手閃光精靈同步爆閃，提供強烈戰術視覺反饋。
   - **大廳/待機狀態 (`!isPlaying`)**：手臂自動平滑 LERP 回正至英雄備戰站姿，推進火焰自動熄滅。

4. **推進火焰細節動畫 (Thrusters & Flames)**：
   - 戰鬥中雙腳靴底推進噴嘴啟動高頻脈衝火焰動畫 (`flameScale = 0.85 + Math.sin(time * 25.0) * 0.25`)。
   - 快速橫向平移或開火時，背部推進噴嘴火焰自動噴發，增強速度感。

5. **效能優化 (Three.js Best Practices)**：
   - 關節旋轉目標與插值均採用預先分配的數值欄位，在每幀 `updateIronManKinematics` 運行期間達成**零 GC 記憶體分配**。
   - 與 `GameController.loop()` 的單一 AnimationLoop 完全整合。

---

## 2. 測試與驗證 (Verification & Testing)

- **語法驗證**：提取模組 JavaScript 代碼，經 `node --check` 驗證無任何語法錯誤。
- **運動學單元測試**：撰寫並執行測試腳本，驗證：
  1. 待機時微浮動高度介於 0.17~0.23 單位，閃光精靈保持關閉。
  2. 雙聯裝開火時僅右手閃光，手臂旋轉收斂至 -1.45 rad。
  3. 3 聯裝開火時雙手閃光精靈同時點亮，雙手平舉收斂至 -1.45 rad。
  4. 橫向移動速度對應正確的側傾角度符號與大小。
- **整體運行確認**：代碼內無任何 NaN、未定義變數或報錯。

---

## 3. 後續任務接口 (Handoff to Task 3)

- 鋼鐵人核心動畫管線已完全建立，可直接供 Task 3 整合：
  - 英雄主動技能 1：胸口集束砲 (Unibeam) 蓄力後座與光束特效發射。
  - 英雄主動技能 2：肩部微型追蹤導彈 (Micro-Missiles) 發射與開蓋運動學。
