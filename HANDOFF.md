# 交接（2026-08-17）

> 給下一個 session 的啟動 prompt，複製即用。本檔與 Claude memory 內容一致；repo 為唯一真相來源。

```
你是「財經文章→影片產線」專案的接手 conductor。先讀再動：

1. Repo（唯一真相來源）：https://github.com/william-agent/finance-article-to-video
   - README.md＝認知基線；EXPERIMENTS.md＝實驗記錄（EXP-001 已完成，含 G3 結論與風險地圖更新）；
     plans/EXP-001.md＝已執行藍圖＋audit 教訓；plans/EXP-002.md＝v2 計畫（未執行）
2. 本機工件（/srv/agent-workspace/）：
   - poc-001/＝EXP-001 全部工件：source/（文章存證+hash）、facts/（facts.json＋verify.py 數字帳本）、
     storyboard/、audio/（edge-tts 五段）、video/（HyperFrames 專案，成片在 renders/）、evidence/（G1 報告、
     QA 核對表、授權清單、逐拍幀圖）
   - hf-bench-inline/bench/＝G1 benchmark 專案
   - 成片已交付 R2 bucket openab-files：poc-001-fund-98636577.mp4

目前狀態：
- EXP-001 完成＝PoC 品質實證：54s 直式片、確定性渲染（兩次 SHA256 一致）、1620 幀約 2.4 分鐘、
  自動 QA 全過、G3 使用者驗收「比預期好，但僅 PoC」。renderer 已不是主要風險。
- EXP-002 v2 已對齊、未執行：驗證 ComfyUI 作為 Generative Asset Factory 的固定合約——
  固定 ShotSpec → 兩個旗艦 adapter（Kling 3.0＋Veo 3.1 Standard，經 ComfyUI 官方 API Nodes，
  統一 credits、本機無 GPU 亦可跑編排）→ provenance ledger → Editor Agent 輸出歸責 EDL →
  HyperFrames deterministic finishing。指標＝cost per acceptable shot（非標價每秒）。
- 等使用者：ComfyUI 帳號＋credits 購買＋預算核可（估 US$40 上限；credits 單價須先在購買頁核對）。
- 零付費可先做：本機裝 ComfyUI（CPU 編排模式）、shotspec.schema.json、EDL schema、
  兩條 adapter workflow 骨架，dry-run 到只差 credits。

必守規則（使用者明確立下，曾糾正過違反）：
1. 宣稱分三級：「能輸出一次」「能按規格輸出」「能穩定量產」是三個各需獨立證據的宣稱；
   未量測的一律寫「未驗證」，禁止用檔案存在性宣稱「環境就緒」「隨時可跑」。
2. 財經數字帳本制：畫面/旁白每個數字必須存在於 facts.json 且可回溯原文出處句；衍生值程式重算。
3. 付費、credential、商用授權 → 先停下向使用者確認，不繞權限。edge-tts 商用 BLOCKED（僅 PoC）。
4. 每個 Agent/模型/工具/成本/授權/責任都要記錄；剪輯決策必須是明確歸責的角色（EXP-001 教訓：
   決策散落在對話中無記錄，被使用者點名）。
5. 實驗執行前定義問題/邊界/PASS-FAIL；結果記入 EXPERIMENTS.md（驗證✅/推翻❌/新發現⚠️），
   認知變更同步回 README，git 留軌跡。

環境備忘：WSL2、12 核、7.4G RAM、無 GPU；/tmp 是 3.8G tmpfs 勿放大檔（幀輸出走 /srv）；
ffmpeg 8.0.1（libx264/aac）、Node 24、Chrome 151（puppeteer cache）、hyperframes 0.7.109、
edge-tts venv 在 ~/.venvs/tts、本地 whisper 在 /srv/agent-workspace/stt/transcribe。
Codex fleet 有權限審批停擺問題（conductor 不可代按批准，被 classifier 擋）；
Editor Agent 建議用 Claude 系 subagent。
```

## 交接時的未結事項

| 事項 | 狀態 | 持有人 |
|---|---|---|
| ComfyUI 帳號＋credits＋預算核可 | 等待 | 使用者 |
| EXP-002 零付費前置（ComfyUI 安裝、ShotSpec/EDL schema、adapter 骨架） | 可隨時開工 | 下一個 session |
| EXP-002b 語音盲測（MiniMax/ElevenLabs/Azure，帳號待開） | 排隊中 | 使用者＋下一個 session |
| 內部 CMoney 數據源（取代爬蟲） | 使用者去問，未回報 | 使用者 |
| 權限批次修正清單（memory: pending-permission-blocks，共 5 項） | 使用者說晚點統一處理 | 使用者 |
| Codex 憑證警報 issue #21 | pane 已消失；scrollback 證據在 poc-001/evidence/issue21-*.txt（結論：該次是誤報，實為審批對話框） | 使用者 |
