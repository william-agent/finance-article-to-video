# 實驗記錄

> 每次實驗一節，對照 [README](README.md) 的認知基線：驗證了什麼、推翻了什麼、暴露了什麼新斷點。

## 模板

```
## EXP-NNN：<名稱>（YYYY-MM-DD）
**假設**：
**做法**：
**結果**：
**對照基線**：驗證 ✅ / 推翻 ❌ / 新發現 ⚠️
**下一步**：
```

---

## EXP-001：基金文章 → 54 秒 PoC reference sample（2026-08-15）

**假設**：見 [plans/EXP-001.md](plans/EXP-001.md)（G0→G1→PoC 三關）。
**做法**：G0 環境實測 → G1 最重場景 benchmark（6s/180幀）→ 54s 全片 cold+warm 渲染 → 自動 QA（ffprobe/抽幀目檢/blackdetect/freezedetect/whisper coverage/數字帳本比對）。
本機檔案：來源存證與 facts `poc-001/source|facts`、成片 `poc-001/video/renders/`、證據 `poc-001/evidence/`（G1 報告、QA 核對表、授權清單、逐拍幀圖）。

**結果**（機器：WSL2 12核/7.4G RAM/無GPU）：
- G0 PASS：libx264+aac 實列、Chrome 151 實渲 CJK 無豆腐、edge-tts 端點實連（5.6s 音檔）
- G1 PASS：180幀 warm 16.5s ≈ 10.9 幀/秒，峰值 RAM 788MB，0 錯誤
- 54s 全片：**cold pipeline 139.4s（牆鐘 2:21.6）／warm 151.3s（牆鐘 2:33.3，與 QA 工作並行有干擾，非乾淨樣本）**，峰值 RAM 833MB，無 swap/OOM
- 輸出：h264 1080x1920 30fps CFR 1620幀 54.000s，AAC 48kHz 立體聲，7.26MB
- **兩次渲染 SHA256 完全一致**（556d39e4…）——確定性渲染實證
- 自動 QA：規格 7/7 ✅；畫面數字 ⟷ facts 帳本逐項 ✅；blackdetect 0 筆；freezedetect 2 筆（判讀為動畫間靜態停留，設計特性）；whisper coverage 五段全在、數字全對

**對照基線**：
- 驗證 ✅：確定性渲染（SHA256 相同）、CJK 字型、渲染速度可行（~2.6x 實時）、RAM 遠低於上限、lint 有真實價值（audio 缺 id 會整段靜音，被 lint 攔下）
- 推翻 ❌：「1620 幀渲染速度未知是最大風險」——實測完全不是瓶頸
- 新發現 ⚠️：(1) 計數動畫過程會顯示非帳本中間值（合規視角待人工判定）(2) 靜態停留 2 段（設計視角待判定）(3) TTS 時長超差在 S3 就被攔截重排（時長實測機制有效）(4) Codex worker 因權限審批無法遠端代批而停擺，conductor inline 完成（fleet 權限模型是產線化的真障礙）(5) 字幕為估切非逐字對齊
- 未驗證（維持 [NOT DONE]）：G3 真人手機驗收、20-run、soak、商用授權（TTS BLOCKED，見 licenses.md）

**下一步**：使用者手機完整觀看驗收（交付通道另測）→ 通過後 20-run。

---

### 模板

```
## EXP-NNN：<名稱>（YYYY-MM-DD）
**假設**：
**做法**：
**結果**：
**對照基線**：驗證 ✅ / 推翻 ❌ / 新發現 ⚠️
**下一步**：
```
