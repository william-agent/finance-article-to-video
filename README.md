# 財經文章 → 影片：自動化產線筆記

> 這份文件記錄產線動工前的認知基線（截至 2026-08-15 的討論共識）。
> 目的：實驗開跑後，用實際結果對照這裡的每一條假設，把「猜的」逐一換成「驗證過的」。
> 實驗記錄寫在 [`EXPERIMENTS.md`](EXPERIMENTS.md)。

## 任務定義

把財經文章（主要來源：[CMoney 投資網誌](https://cmnews.com.tw/twstock)）自動轉成影片。
不是「學會做影片」，而是**建一條幾乎不需要人的內容產線**：文章進、影片出，由 AI agent 操作全程。

---

## 一、工具地圖：三個工具，兩個格子

研究過的三個工具其實分屬產線的不同工序，**不是三選一**：

| 工具 | 本質 | 在產線的格子 |
|---|---|---|
| [ComfyUI](https://github.com/comfyanonymous/ComfyUI) | 節點式 AI 圖像生成（Stable Diffusion 等擴散模型） | **素材生成**：縮圖、風格化背景、頻道視覺資產 |
| [HyperFrames](https://github.com/heygen-com/hyperframes) | HTML → 確定性渲染 MP4（headless Chrome + FFmpeg），為 agent 設計 | **組裝輸出**：模板 × 數據 → 影片 |
| [Remotion](https://www.remotion.dev) | React → 影片，同樣的渲染原理 | **組裝輸出**（與 HyperFrames 同格子、互為替代） |

關鍵區別：**擴散模型每次生成都不一樣，確定性渲染每次都一樣**。
量產模板要的是後者，所以產線主幹是 HyperFrames/Remotion 這類工具；ComfyUI 的正確位置是單點素材，其中槓桿最大的是「YouTube 縮圖」（點擊率的最大單一變因），而不是想像中的「影片畫面主要來源」。

HyperFrames vs Remotion 的選型傾向：HyperFrames（HTML、無建置步驟、Apache 2.0、原生 agent skills、20 個現成工作流如 `/faceless-explainer`）。理由：寫程式碼的是 agent 不是人，HTML 交接最順。Remotion 較成熟但授權是 source-available、以 React 為前提。**此傾向未經實測，屬待驗證假設。**

---

## 二、產線分層與瓶頸地圖

```
財經文章
   │
   ▼
[1] 解析與數據抽取        ◄ 風險瓶頸
   │   數字/表格 → 結構化    （數字錯=事故）
   ▼
[2] 分鏡腳本（LLM）       ◄ 品質瓶頸 ★★★
   │   鉤子、敘事、節奏      （決定留存率）
   ▼
[3] 素材層
   ├── 圖表（真實數據）    ◄ 準確性要求
   ├── TTS 旁白           ◄ 品質瓶頸 ★★
   └── 配圖/縮圖(ComfyUI) ◄ 產能瓶頸(GPU)
   ▼
[4] 組裝渲染(HyperFrames) ◄ 產能瓶頸
   │   模板 × 數據 → MP4    （可無限水平擴展）
   ▼
[5] QA 驗證               ◄ 規模化的真瓶頸 ★★★
   │   數字回溯、合規、聽感  （唯一無法全自動的一步）
   ▼
[6] 發布 + 回饋迴路        ◄ 成長瓶頸
        縮圖CTR、留存曲線 → 回頭修模板
```

### 認知盤點（動工前的自我狀態）

- **原本的注意力全部集中在 [3][4] 層**（工具選型：ComfyUI？HyperFrames？Remotion？）。
- 但 [3][4] 是「花錢就能解」的瓶頸——渲染可水平擴展、生圖可排隊。
- 真正稀缺的是 [2] 腳本品質和 [5] QA：**規模化的極限 = QA 人力，不是算力**。
- 品質金字塔（由下而上）：腳本敘事 → 旁白聲音 → 資訊圖表 → 視覺風格。多數人從最上層開始研究（包括當時的我們），但觀眾流失的原因幾乎都在最下層。
- 素材是三分天下：程式畫的圖表（財經影片占比最大）、模板字卡、生成/圖庫圖。原本高估了「AI 生圖」的占比（想像 60%，實際約 15%）。

### 財經內容的特殊約束

1. **數字必須精確**——LLM 會幻覺數字，影片裡每個數字要能回溯到原文或原始數據源，不能是 LLM 轉述。這道驗證是 [1] 和 [5] 的核心。
2. **擴散模型畫不出正確的數字和文字**——圖表必須用真數據程式繪製，這正是確定性渲染工具的主場。
3. **合規邊界**——個股內容注意「分析 vs 建議」的投顧法規邊界；免責聲明做成固定版位字卡。

---

## 三、業界 best practices（2026-08 查證）

- **標準管線**：research → script → voice → render → upload。主流 stack = 程式化渲染（Remotion 類）+ LLM 腳本 + TTS，與本產線設想一致。
- **旁白是留存率第一槓桿**：雙層聲音策略——主打影片用高階 TTS（ElevenLabs 級），量產用便宜 TTS；30 秒樣本 voice cloning 維持頻道聲音識別。
- **人工 QA 不可省**：再成熟的管線，發布前仍需 2–3 分鐘人工檢查，否則錯誤影片傷頻道信譽。
- **以模板思考，不以影片思考**（think in compositions, not videos）：每支影片是模板的一次實例化；前期投資模板庫，回報是複利。
- **保持組合簡單**：渲染時間隨場景複雜度暴漲。
- **輸出端固定接** `ffmpeg -crf 28 -preset slow`：檔案小 80%，畫質無可見損失。
- **Docker + CI 打包渲染**，一人管 5–10 個頻道在 2026 是常態。
- **平台格式從第一天決定模板**：模板必須「比例無關」（16:9 / 9:16），否則之後全部重做。財經內容甜蜜點：60–90 秒直式速讀 + 不定期橫式深度片。

參考來源：
[Remotion pipeline lessons](https://dev.to/ryancwynar/i-built-a-programmatic-video-pipeline-with-remotion-and-you-should-too-jaa) ·
[Remotion + Docker scaling](https://scotthavird.com/blog/remotion-docker-template/) ·
[Faceless channel voiceover guide](https://www.speechmatics.com/company/articles-and-news/how-to-launch-a-successful-faceless-youtube-channel-for-business) ·
[7-step automation blueprint](https://zeroskillai.com/faceless-youtube-ai-automation/) ·
[awesome-faceless](https://github.com/sasharun/awesome-faceless) ·
[AI video pipeline 2026](https://www.trezalabs.com/blog/how-to-build-an-ai-video-generation-pipeline)

---

## 四、內容源評估：cmnews.com.tw（2026-08-15 實測）

- **爬蟲可行性極高**：純 SSR HTML，`curl` 可得全文，無擋 bot，內建 JSON-LD 結構化資料。
- 文章連結格式 `/article/cmoney-<uuid>`，四個子頻道：`twstock_news` / `twstock_column` / `twstock_report` / `twstock_fund_etf_future_material`。
- **更好的路（待確認）**：文章疑似由數據模板生成，上游應有 CMS／結構化數據源。若能從公司內部拿到，數據準確性問題先天解掉一半。爬蟲是 fallback。

### 「編譯」步驟的定義（文章 → 口說稿的落差）

文章語氣已相當口語，落差不在語氣，在四件事：

1. **長度差 5–8 倍**：2,000 字文章 vs 60–90 秒影片只容 250–400 字 → 第一工序是取捨。
2. **敘事反轉**：文章線性鋪陳 → 影片鉤子先行（前 3 秒）。
3. **數字從散文拆成畫面**：數據不該被「唸」，該變成圖表/字卡，旁白只講結論。
4. **書面元素剝離**：免責聲明→固定字卡、站內連結/標籤/網頁式 CTA→移除或改寫。

### 中間格式：storyboard JSON（產線的真正資產）

「編譯」的產物。範例（節錄自實測文章〈中國信託華盈貨幣市場基金〉）：

```json
{
  "hook":  { "sec": 4,  "voice": "一檔695億的基金，全台灣只有255個人買。",
             "visual": "stat-contrast", "data": {"規模":"695億","持有人":255} },
  "beat1": { "sec": 12, "voice": "它是風險等級最低的貨幣市場基金，大戶把它當數位保險箱…",
             "visual": "concept-card" },
  "beat2": { "sec": 15, "voice": "報酬不刺激，但一路向上…",
             "visual": "bar-chart", "data": {"1年":"1.5%","3年":"4.4%","5年":"5.7%"} },
  "beat3": { "sec": 15, "voice": "三種人適合：等交屋的、觀望中的、存緊急預備金的…",
             "visual": "list-reveal" },
  "outro": { "sec": 8,  "voice": "資金的中繼站，不是終點站。",
             "visual": "brand-card + 免責聲明" }
}
```

這個格式一旦定案，下游渲染工具隨時可換——**它比任何工具選型都重要**。

---

## 五、實驗路線圖

1. **端到端 MVP**（本週）：一篇 cmnews 文章 → 60 秒影片（真實數據圖表 + TTS + 字幕）。醜沒關係，目的是暴露斷點，對照第二節的理論地圖。
2. **定義 storyboard JSON schema + 數據驗證規則**（護城河）。
3. **TTS 盲測**：3–4 家同一段旁白，實聽比較（Azure 台灣聲線 / ElevenLabs / 開源本地方案）。
4. **模板庫**：財報速讀 / 盤後回顧 / 產業專題版型（比例無關設計）。
5. **ComfyUI 縮圖線**、發布自動化、回饋迴路。

另一條待評估支線：HyperFrames 母公司 HeyGen 的數位人——「AI 財經主播」固定臉孔的頻道識別度，faceless 做不到。

## 環境現況（機器已就緒的部分)

- yt-dlp（`~/.venvs/yt-dlp`，YouTube 內容研究用）、FFmpeg、Node.js、Python 3.14
- 本地 faster-whisper STT（影片轉錄）
- HyperFrames 原始碼已研究（v0.7.109，開發極活躍，昨日兩個 release）
- cmnews 爬蟲已驗證可行
