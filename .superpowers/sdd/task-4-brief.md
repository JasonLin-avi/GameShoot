# Task 4 Brief: 主題動態適配、音效整合與端到端綜合驗收

## Files
- Modify: `D:\Programming\Game21\index.html`

## Global Constraints
- 嚴格遵守 `three-best-practices`：零迴圈內 GC 記憶體分配、單一 AnimationLoop、及時 dispose 幾何體與材質。
- 免外部圖片依賴（純 3D 金屬 PBR 裝甲 + Canvas 自發光材質），確保本機 `file://` 與離線環境 100% 正常運行零報錯。
- 嚴格維持原本藍軍增殖機制、倍增門運算、紅軍波次與 60 FPS 流暢度。

## Detailed Requirements

1. **主題色彩動態適配 (`applyTheme`)**
在 `ThreeGameScene.applyTheme(theme)` 與 `GameController.setTheme()` 中：
- 當玩家切換「電光競技 (arcade) / 清爽晴空 (bright) / 賽博糖果 (candy)」時：
  - 鋼鐵人方舟反應爐與目鏡 `arcGlow.color` 動態更新為主題的藍軍主色：
    - `arcade`: `0x00f0ff` (電光青藍)
    - `bright`: `0x0284c7` (晴空寶石藍)
    - `candy`: `0x00f5d4` (青檸薄荷綠)
  - 胸口集束砲 (Unibeam) 外層能量光柱與同軸光環的顏色同步對齊主題色。
  - 確保底盤能量光環及掌心閃光顏色自然契合。

2. **Web Audio 音效系統深度整合 (`SoundFX`)**
在 `SoundFX` 類別中新增或完善英雄技能專用合成音效（無外部音檔，免下載）：
- `playUnibeam()`：包含瞬間蓄能重低音 (70Hz 鋸齒波向下掃描) 與高能能量貫穿共振音。
- `playMissileLaunch()`：短促清脆的推進器點火呼嘯音 (頻率由 200Hz 迅速滑升至 700Hz)。
- `playMissileHit()`：更具立體打擊感的爆裂轟鳴。
- 在 `triggerUnibeam()` 與 `fireMicroMissiles()` 及其命中時正確呼叫。

3. **微型導彈尋標演算法極致最佳化 (0-GC 優化)**
根據 Task 3 審查建議，重構 `findMissileTarget()`：
- 淘汰內部 `activeGargoyles = []` 與 `.sort()` 陣列分配。
- 改為單次線性迴圈遍歷（Single-pass scan），直接計算並返回距離最近的 Boss、石像鬼或紅軍步兵，達成 100% 純粹的零記憶體配置 (0 GC)。

4. **端到端功能與效能驗證**
- 執行 `node --check index.html` 或相關驗證指令，確保代碼語法無任何瑕疵。
- 檢查以下遊戲流程全覆蓋：
  1. 主選單：主題切換正常更新鋼鐵人反應爐色彩，開始戰鬥按鈕正常。
  2. 戰鬥中：鋼鐵人左右移動滑順，開火單/雙手姿態自適應切換，藍軍小兵正常穿過倍增門並增殖。
  3. 英雄技能：按 [E] 發射集束巨砲消滅路徑敵軍，按 [Q] 發射追蹤導彈精準命中敵軍，冷卻遮罩與倒數正常。
  4. 結算：勝利與失敗彈窗正常，金幣結算無誤，返回大廳不報錯。
- 提交代碼：`git commit -m "feat: complete theme adaptation, audio fx, and end-to-end integration"`
