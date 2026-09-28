# Reports 導覽：報告清單與狀態、建議閱讀順序與關係圖、關鍵事實速查表、縮寫編號索引

> 最後整理：2026-09-28（新增 08-28 起的五份主定理相關報告；主定理稿 v3.1、KT18 正式稿與定義稿 PDF 收於 `Paper drafts/`；舊報告文首補上 09-28 狀態註記）。報告間存在「後文更正前文」的關係；讀任何一份之前，先看本頁確認它的狀態與被更正處。各報告文首均有更正/狀態註記，與本頁同步。
>
> **只想知道現在在做什麼** → 讀[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（現行定義、構造與完整證明）與 [KT18 報告](./KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md)（最新進展與待拍板問題 Q1–Q5）。整體技術地圖（定義維度 A1–A5、與既有工作的邊界）見 [Unclonable IBE 主軸路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)，但其 D-W1 與甲乙案已被後續報告更新（見其文首狀態註記）。

---

## 1. 現況摘要（截至 2026-09-28）

- **主軸已定案：W1 — Unclonable IBE**。「unclonable IBE」一詞文獻中仍不存在（KT18 報告 2026-09-28 複查）。
- **主定理已有完整證明（[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)，2026-09-27）**：Construction 1 `ct = (ct₁, c₂ := k̄ ⊕ k, otUE.Enc(k, m))`（GKK25 RNC-IB-KEM ＋ OTP 墊層 ＋ otUE）；一條 hybrid 鏈、兩個身分相反的歸約（Claim 1 → RNC-IB-KEM 模擬安全；Claim 2 → otUE cloning 遊戲），t 無損。已拍板：修訂二（OTP 墊層）、修訂四（揭露 sk_{id\*}，老師要求）；修訂一（memoized）撤回；修訂五刪除凍結、表 K 與延後引理。**D-W1 從一開始就不是問題**，主定理對任意身分空間成立；真正的時間邊界是分裂（Remark 2）。待老師確認查詢階段 II／III 的寫法與 Remark 2。
- **後量子實例**：SXDH／DDH 實例對 QPT 對手不安全（cloning 對手可在分裂前以 Shor 打開古典槽）——**Construction 1 目前唯一合法的實例是 GKK25 Thm 3/14 取 LWE（poly-ID）**。
- **最新進展：KT18 × 不可複製加密（2026-09-28）**：Construction 2 以 KT18 的 RSO 雙重加密當槽，**任意**後量子 IND-ID-CPA IBE ⇒ UIBE（LWE 下指數身分空間、允許挑戰後查詢、證明不用模擬器）——對現行定義而言甲案／乙案困境消失，msk 強化版仍只由 Construction 1 提供；Construction 3（雙槽）達到 KDM-CPA 與 KDM-unclonable。待老師拍板 Q1–Q5。
- **揭露後禁查 id\* 的評估（2026-09-28）**：老師認為揭露後禁查 id\* 怪怪的、或可改用 non-adaptive。[評估](./Post_Reveal_Challenge_Identity_Query_Ban_vs_Non_Adaptive_Evaluation.md)建議改為「分裂前禁查 id\*；分裂後先揭露同一把 sk_{id\*}，再不設限查詢」：Construction 1 零成本（msk 版已蘊含），Construction 2 需以 PRF 去隨機化 KeyGen（即 KT18 報告 Q5）；non-adaptive 不需要。待拍板 Q-a–Q-c。
- **定義章待同步**：[定義草稿](./Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md)（md 與 PDF 版）的 Definition A/B 需改為 v3.1 Definition 2／3 的形式。
- **路線甲（Unclonable IPFE）已降為備援／延伸章**——引用地圖確認零競爭者，但技術風險較高（D2：合法解密者經 r 洩漏 x）。Deep Dive 報告內容仍有效，作為 W1 撞牆時的退路。
- **白板線（重構 UE）原形已關閉**——正解 RNCE 已被 HKNY24（TCC 2024）Appendix E 先行做掉（含 unclonable-IND 保持）。剩餘 V1（compiler 基礎化）可當論文理論章、與 W1 共用工具；V2 高風險、V3 價值未驗證。
- **已否定的構造想法**：Clifford-QOTP 直構（可逆性洩漏攻擊，轉為 no-go observation，本身可交付）。

---

## 2. 報告清單與狀態

檔名即大綱；下表的「內容」欄是同一份大綱的展開，「暱稱」欄是各報告內文互相引用時使用的簡稱。

| 報告（點擊即為完整標題） | 暱稱 | 日期 | 內容與狀態 |
|---|---|---|---|
| [揭露後禁查 id\* 與 non-adaptive 選項：評估與建議修改](./Post_Reveal_Challenge_Identity_Query_Ban_vs_Non_Adaptive_Evaluation.md) | 揭露後禁查 id\* 報告 | 2026-09-28 | **最新**：回應老師「揭露後禁查 id\* 怪怪的，或用 non-adaptive」。分裂後的禁令不擋平凡攻擊；引理 A（揭露前後的查詢等價）、B（msk 版 ⇒ 不設限版 ⇒ 現行版 ⇒ 分裂後無查詢）、C（KeyGen 確定性時禁令是空的）；建議「分裂前禁查 id\*、分裂後先揭露同一把 sk_{id\*} 再不設限查詢」，Construction 1 零成本、Construction 2 需 PRF 去隨機化；non-adaptive 不需要。附可貼 Overleaf 的定義框與 Remark；待拍板 Q-a–Q-c |
| [KT18 × 不可複製加密：RSO 機制直接填槽（任意 IBE ⇒ UIBE）、KDM 機制的雙槽設計、兩個新安全概念與證明骨架](./KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md) | KT18 報告 | 2026-09-28 | Construction 2（RSO 雙重加密當 k 的槽；任意 pq IND-ID-CPA IBE ⇒ UIBE，指數 ID）、RSO 延伸（Theorem 4、Proposition 1）、KDM 的三個陷阱與 Construction 3（Theorem 5/6）、三個構造總對照、待拍板 Q1–Q5。正式稿見 `Paper drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf` |
| [Lemma 1（分裂後查詢延後）不需要：誰在回答查詢、兩個揭露事件、兩階段遊戲的文獻做法與真正的時間邊界](./Deferred_Query_Lemma_Unnecessary_Who_Answers_Queries_Two_Reveals.md) | 延後引理報告 | 2026-09-27 | 現行：刪除 Lemma 1、兩個揭露事件、時間邊界是分裂、文獻對照（GKK25 Thm 15、HMNY21）；**主定理稿 v3.1 由此產生**（§7 為 v3.1 的修改清單） |
| [老師的兩個提問：古典／量子物件盤點；「不得查詢 id\*」規則與 non-adaptive 選項的評估](./Classical_vs_Quantum_Inventory_and_Challenge_Identity_Query_Rule_Evaluation.md) | 古典／量子盤點報告 | 2026-09-13 | 現行（§1 物件表、§2.1 分裂前禁查 id\* 的必要性、§2.3 維持 adaptive、§2.4 D-W1 不存在）。09-27 更正：SXDH／DDH 不是合法實例。§2.2 的「分裂後不設限」提案未被 v3.1 採用（KT18 報告 Q5）；09-28 老師再度提出，評估見揭露後禁查 id\* 報告 |
| [換掉 Hyb₁ 的凍結表 K：分裂後查詢的「延後引理」、四條替代路線的評估與文獻對照](./Replacing_Frozen_Key_Table_in_Hyb1_Deferred_Query_Lemma_and_Alternatives.md) | 凍結表報告 | 2026-09-12 | 結論（刪掉表 K、身分空間任意）成立，但機制已被古典／量子盤點報告 §2.4 取代：延後引理不需要（v3.1 已刪除）。09-27 更正：SXDH／DDH 不是合法實例 |
| [主定理證明設計：一條 Hybrid 鏈、兩個身分相反的歸約（自足版）](./Proof_Design_Main_Theorem_Two_Reductions_and_Hybrid_Chain.md) | 證明設計 | 2026-08-28 | 已由主定理稿 v3.1 取代：兩步骨架不變；修訂一撤回、修訂二改採 OTP 墊層（非 key-transparent 前提）、凍結與 T = poly 的限制刪除、揭露改交 sk_{id\*} |
| [Unclonable IBE 主軸路線圖：定義五維度、構造與主定理、證明骨架與 D-W1、後量子兩案](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md) | W1 路線圖 | 2026-08-01 | **主軸技術總圖**（定義維度 A1–A5、與既有工作的邊界、執行計畫。其 §6 更正了 HMNY21 的發表處與「零量子應用」的說法）。部分已更新：D-W1（§2.3）確認不存在；構造改為 OTP 墊層、揭露改交 sk_{id\*}（主定理稿 v3.1）；§4.3 甲案／乙案的前提被 KT18 報告 Construction 2 改變 |
| [Unclonable IBE 定義章草稿：game vs simulation 的分層、四個模擬器對應、定義 A/B 與 KEM 版槽位介面](./Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md) | 定義草稿 | 2026-08-07 | 路線圖 §1–§2 的細化：釐清 fake-key 對應的是 Sim₄ 而非 Sim₂、E3 是「歸約者在第二步變成模擬器」的邏輯後果、主定理應以 RNC-IB-KEM（Def 8）為介面——仍有效。**定義 A/B 待同步**為 v3.1 Definition 2／3（揭露 sk_{id\*}、查詢階段 II／III、標準預言機）；§2 的 A2 三個解已不需要。PDF 版見 `Paper drafts/Draft_of_Security_Definition_of_Unclonable_IBE.pdf` |
| [Meeting 筆記（2026-08）：RNC-IBE 介面核對的五個發現與三個待拍板問題](./Meeting_2026-08_RNC_IBE_Interface_Check_and_Open_Decisions.md) | 2026-08 Meeting 筆記 | 2026-08-01 | 歷史紀錄（meeting 用：E1–E4 逐條對上、Def 6 交出 msk、D-W1 的三條出路、後量子只有 Thm 3 可用、主定理改走 KEM 介面；待拍板＝混淆電路的範圍／pq 甲乙案／split 後查詢是否進定義）。後續：split 後查詢已進定義（v3.1）；D-W1 不存在；甲乙案的前提被 KT18 報告改變（Construction 2 不用 garbled circuit）；§1.4 仍成立 |
| [Meeting 報告（2026-07）：兩條候選主軸的做法、可行性、對照表與三個論文骨架選項](./Meeting_2026-07_Two_Candidate_Routes_IPFE_vs_IBE.md) | Meeting 討論報告 | 2026-07-13 | 歷史紀錄：兩線並列的討論定稿。**§4 的三個骨架選項已由 W1 路線圖取代**（主軸定為 W1）；§1–§3 的背景與兩線描述仍有效 |
| [量子函數加密方向 Survey：兩個延伸方向、閱讀清單與「升級階梯 compiler」構想](./Quantum_FE_Directions_and_Upgrade_Ladder_Idea.md) | QFE survey | 2026-05 | 部分過時：方向 1 直構想法被 D4 攻擊否定；Project B/C 判斷已由深度分析更新 |
| [不可複製加密的五個研究前沿：公鑰偽金鑰、演進史、與 NCE 的三角關係、不可複製 FE、一對多推廣](./Unclonable_Encryption_Five_Research_Frontiers_Map.md) | 五方向報告 | 2026-05 | 部分過時：方向一/三的新穎性判斷被 HKNY24 App E 推翻；方向五結果端被 CGKNY26 拿走 |
| [偽金鑰性質為何至今沒有公鑰版：結構性障礙、公鑰 UE 的替代範式、可模糊性概念的角色](./Fake_Key_Property_Why_No_Public_Key_Version_Exists.md) | 偽金鑰報告 | 2026-05（07-12 更正） | 核心判斷已更正：「六種範式」應為七種（HKNY24 App E 的 RNCE 路線） |
| [文獻總盤點（2026-07）：MM24 之後的最新結果、空白區與擁擠區、五條候選路線優先序](./Literature_Survey_2026-07_Results_Gaps_and_Route_Ranking.md) | 7 月 survey | 2026-07-10 | 兩處更正：漏掉 HKNY24 App E；路線甲的直構想法被否定。其餘文獻盤點與路線評估仍有效 |
| [Unclonable IPFE 備援路線深潛：目標語義、MM24 介面需求、路徑 P1–P3、難點 D1–D10、補完清單 K1–K12](./Unclonable_IPFE_Backup_Route_Paths_Obstacles_Knowledge_Gaps.md) | 路線甲深度分析 / Deep Dive | 2026-07-10 | **降為備援**：技術內容仍有效（P1–P3、D1–D10、K1–K12），但路線甲已非主軸。用途＝W1 撞牆時的退路、論文延伸章、Clifford no-go 短文的來源 |
| [白板配方解碼：otUE + PQC 的出處與真實介面、四個換槽方向 R1–R4、與路線甲的分界](./Whiteboard_Recipe_Decoded_and_PQC_Slot_Swap_Directions.md) | 白板報告（v3） | 2026-07-12 | R1/R3 新穎性被 v3 更正推翻；R2（V1）存活；§4 兩線對照與 §1 白板解碼仍有效 |
| [AK21 逐頁精讀與槽位替換品盤點：介面 E1–E4、HKNY24 App E 查證、真空格 V1–V3、候選 W1–W6](./AK21_Close_Reading_Slot_Interfaces_and_Route_Candidates.md) | AK21 精讀報告 | 2026-07-12（07-13 追加 §4+） | 現行（**W1 的出處**：E1–E4 介面、HKNY24 查證、V1–V3、W1–W6。注意：§6 的 HMNY21 發表處與 §3.3「同年同會」已被路線圖 §6 更正；§5.3 的「主軸轉回路線甲」建議已被推翻） |

**相關正式稿（`Paper drafts/`，LaTeX 原始檔未收入本儲存庫）**

| 檔案 | 暱稱 | 日期 | 內容與狀態 |
|---|---|---|---|
| [Proof_of_Main_Theory_in_UIBE.pdf](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf) | 主定理稿 v3.1 | 2026-09-27 | **現行主定理文件**：Definition 2／3、Construction 1、Theorem 1／2 與完整證明、修訂一～五的拍板狀態、下一步 |
| [UIBE 兩個歸約圖解.html](../Paper%20drafts/UIBE%20兩個歸約圖解.html) | 歸約圖解 | 2026-09-28 | 主定理稿 v3.1 第 3 節兩個歸約（Claim 1、Claim 2）的圖解 |
| [KT18_x_UE_Constructions_Theorems_Proofs.pdf](../Paper%20drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf) | KT18 正式稿 | 2026-09-28 | KT18 報告 §3、§4 的正式版：Construction 2/2′/3、Definition 2⁺／SO-cloning／K1／K2、Theorem 3–6、Proposition 1 與證明（討論用草稿） |
| [Draft_of_Security_Definition_of_Unclonable_IBE.pdf](../Paper%20drafts/Draft_of_Security_Definition_of_Unclonable_IBE.pdf) | 定義稿 PDF | 2026-08-14 | 定義草稿的 PDF 版（定義 A/B 為揭露 msk 的版本），待同步為 v3.1 Definition 2／3 |

---

## 3. 建議閱讀順序

**只要現況／要動手做事** → **主定理稿 v3.1 ＋ KT18 報告**；需要整體地圖時再讀 W1 路線圖（先看其文首狀態註記）。

**要知道為什麼是 W1**（三份，約 40 分鐘）→ 精讀報告 §3（HKNY24 App E 的發現）→ 精讀報告 §4+（第二輪盤點與 W1 的誕生）→ W1 路線圖。

**要知道主定理的證明怎麼走到 v3.1**（五份）→ 證明設計 → 凍結表報告 → 古典／量子盤點報告 §2.4 → 延後引理報告 → 主定理稿 v3.1。

**完整脈絡（時間順）**：

1. **QFE survey**（起點：老師兩點建議 + 升級階梯構想）
2. **五方向報告**（UE 端五個前沿的地圖）
3. **偽金鑰報告**（方向一的深入調查）
4. **7 月 survey**（文獻總盤點：空白區/擁擠區 + 路線甲乙丙丁戊評估）
5. **Deep Dive**（路線甲技術深潛：P1 首攻、D1/D4 兩個關鍵發現）
6. **白板報告 v3**（白板線 = 重構 UE 的換槽方向 R1–R4；與路線甲的分界）
7. **精讀報告**（AK21 逐頁精讀 → HKNY24 App E 發現 → W1 的提出）
8. **Meeting 報告**（兩線並列的定稿，供老師討論）
9. **W1 路線圖**（主軸定案，RNC-IBE 精讀落地）
10. **定義草稿**（路線圖 §1–§2 的細化：定義風格分層、模擬器對應、定義 A/B）
11. **證明設計**（兩步 hybrid、兩個身分相反的歸約、三項修訂提案）
12. **凍結表報告**（回應老師「表 K 會碰到指數」：延後引理）
13. **古典／量子盤點報告**（老師的兩個提問；發現 D-W1 不存在）
14. **延後引理報告**（刪除 Lemma 1 → 主定理稿 v3.1）
15. **KT18 報告**（KT18 兩個機制與不可複製加密的結合：Construction 2／3）

```
QFE survey ──────────┐
                     ├─→ 7月survey ─→ Deep Dive（路線甲）──（備援）
五方向 ─→ 偽金鑰報告 ─┘         └────→ 白板報告 v3 ─→ 精讀報告 §4+
                                                              │ W1 誕生
                                       Meeting 報告（兩線並列）─┤
                                                              ↓
                                                W1 路線圖（主軸技術總圖）
                                                              ↓
                                                定義草稿（定義章落地）
                                                              ↓
                                                證明設計（兩步 hybrid）
                                                              ↓
                                                凍結表報告（延後引理）
                                                              ↓
                                                古典／量子盤點報告（D-W1 不存在）
                                                              ↓
                                                延後引理報告 ─→ 主定理稿 v3.1
                                                              ↓
                                                KT18 報告（Construction 2／3）
（─→ = 承接；精讀報告回頭更正了 五方向/偽金鑰/7月survey 三份；
  W1 路線圖回頭更正了 精讀報告 的兩處引用與一處建議；
  定義草稿更正了「fake-key 對應 Sim₂」這個容易搞混的對應；
  古典／量子盤點報告發現 D-W1 不存在，取代了 證明設計／凍結表報告 的凍結與延後引理；
  09-27 重讀時更正了 v3、凍結表報告、古典／量子盤點報告 中「SXDH／DDH 是合法實例」的錯誤；
  KT18 報告改變了 W1 路線圖 §4.3 甲案／乙案的前提）
```

---

## 4. 關鍵事實速查（各報告敘述以此為準）

| 事實 | 定稿版本 | 出處報告 |
|---|---|---|
| **現行定義（主定理稿 v3.1 Definition 2／3）**：揭露物是 sk_{id\*}（修訂四，老師要求）；分裂後查詢分為查詢階段 II／III，皆由挑戰者回答且禁查 id\*；預言機為標準約定（memoized 已撤回）；Setup(1^λ, 1^d) | 揭露整把 msk 是 Remark 3 的強化版，同一份證明成立 | 主定理稿 v3.1 §1、§4 |
| **構造第 2 步是 OTP 墊層** `k ← otUE.Setup`、`c₂ := k̄ ⊕ k`（修訂二出路 2），不是 key-transparent 前提 | 主定理對任意 t-unclonable otUE 成立；BL20、AS26 可直接代入 | 主定理稿 v3.1 §4（取代證明設計 §7 的「主定理採用出路 1」） |
| **D-W1 從一開始就不存在**：挑戰後的所有金鑰由 KeyGen(m̃sk, ·) 供應；Claim 2 的 B̃ 一開始執行就拿到 k、算得出 m̃sk。凍結、表 K、延後引理皆不需要 | 主定理對任意身分空間成立 | 古典／量子盤點報告 §2.4；延後引理報告 |
| **兩個揭露事件**：UIBE 遊戲的揭露（B、C 收到 sk_{id\*}）≠ otUE 遊戲的揭露（B̃ 收到 k）；後者發生在 B̃ 開始模擬 B 之前 | 混淆兩者是先前以為需要延後引理的原因 | 延後引理報告 §3；v3.1 Remark 11 |
| **時間邊界是分裂，不是揭露**：A 在挑戰後、分裂前不得查詢；分裂後任何時點都可查 | 對 Construction 1 是必要的（GKK25 Def 8 挑戰後不回答查詢）；Construction 2 可移除（Definition 2⁺） | 延後引理報告 §4；v3.1 Remark 2；KT18 報告 §3.6–§3.7 |
| **SXDH／DDH 實例不合法**：cloning 對手是量子的，可在分裂前以 Shor 打開古典槽 | Construction 1 唯一合法的實例＝GKK25 Thm 3/14 取 LWE（poly-ID） | 2026-08 Meeting 筆記 §1.4；v3.1 Remark 9 |
| 分裂前禁查 id\* 是**必要**的（否則 A 解密後把古典的 m 複製給兩人，平凡獲勝） | 這就是不可複製性的內容，不是人為的 admissibility 條件 | 古典／量子盤點報告 §2.1 |
| **Construction 2**：KT18 §5 的雙重加密對「使用者金鑰」非承諾；任意 pq IND-ID-CPA IBE ＋ t-unclonable otUE ⇒ t-unclonable UIBE（揭露 sk_{id\*}） | LWE adaptive IBE ⇒ 指數身分空間；msk 揭露版既無證明也無攻擊 | KT18 報告 §3 |
| **KDM 不能直接填 k 的槽**（與事後模糊化衝突）；Construction 3 用雙槽：k 走 RSO 槽、z = y ⊕ m 走 KdmIBE 槽 | KDM-CPA（Theorem 5）＋ KDM-unclonable（Theorem 6） | KT18 報告 §4 |
| **RNC-IBE = GKKNRY, PKC 2025**（各報告亦稱 GKK25）。Def 6 adaptive／Def 7 selective；模擬器 Sim₁–Sim₄，**Sim₄ 輸出的是 msk 不是 sk_{id\*}** | 挑戰階段對手收到 (msk, ct\*)——W1 因此可做「msk 洩漏仍不可複製」的強版（現行定義依老師要求只揭露 sk_{id\*}，msk 版為 v3.1 Remark 3） | W1 路線圖 §3 |
| **後量子 RNC-IBE 的唯一現成選項 = 其 Thm 3 取 X = LWE**：adaptive 安全、mpk compact、**身分空間僅多項式大小 T**、密文 poly(T, \|m\|, λ)。主構造 Thm 1 是 SXDH（非 pq）；Thm 5/6 需 iO | 別誤記為 selective；relaxed 版是 adaptive 的。UIBE 的後量子實例不再只靠它：Construction 2 只需 pq IND-ID-CPA IBE | W1 路線圖 §3.2、§4；KT18 報告 §3.6 |
| **HMNY21 = ASIACRYPT 2021**（非 TCC 2021），全名 *Quantum Encryption with Certified Deletion, Revisited: Public Key, Attribute-Based, and Classical Communication*；NC-ABE 出自此文（用 iO），動機是**憑證刪除**不是不可複製性 | 更正精讀報告 §6 的 TCC 2021 與 §3.3 的「同年同會」 | W1 路線圖 §6 (C1) |
| 「RNC-IBE 零量子應用」需精確化：**該篇論文**未做量子應用，但 NC-ABE 這個概念本來就是為量子應用（憑證刪除）而生 | 競速風險應上調 | W1 路線圖 §6 (C2) |
| HKNY24 = arXiv:2311.09487 = *Robust Combiners and Universal Constructions for Quantum Cryptography*（TCC 2024）。§7 UE combiner、§8 明文擴展、**App E：RNCE ⇒ unclonable PKE（Lemma E.2 保 unclonable-IND）** | 三個章節是同一篇論文，勿當三篇 | 精讀報告 §3 |
| AK21 證明真正消耗的槽位介面是 E1–E4（trapdoored 可模糊化即足夠），Def 16（無 trapdoor + 誠實密文）過強 | Bendlin 牆咬的是過強版，證明不需要它 | 精讀報告 §2 |
| AK21 假設已最優（私鑰 OWF／公鑰 PKE），「換槽」不可能是降假設 | — | 白板報告 §2 |
| Clifford-QOTP 直構有可逆性洩漏攻擊（L_C 可逆 ⇒ 取回 ρ 本身） | 已從構造候選轉為 no-go observation | Deep Dive §3.3（D4） |
| MM24 Thm 7 lifting 消耗 universality，對 IPFE 類不適用 as stated | 換底層的真正工作量在重建 lifting | Deep Dive §2（D1） |
| BBC26《The Uncloneable Bit Exists》：**引用 v2**（2026-06，v1 分析有瑕疵）；Haar 隨機酉、**無有效構造** | 不能直接當 lifting 的底座（違反 poly 描述需求） | 7 月 survey §2.2；Deep Dive §3.2（D6） |
| BC26（Bhattacharyya-Culf, decoupling）：arXiv 2025，Nature Physics 正式刊出為 **2026-02**，逆多項式安全 | 引用年份以 2026 為準 | 7 月 survey §2.2 |
| CGKNY26 多複製安全：**CRYPTO 2026**（ePrint 2025/1921） | 「結果」已關閉；「一對多偽金鑰」技術路徑仍空 | 7 月 survey；白板報告 R4 |
| MM24 仍是 ePrint 預印本（2024/1683），未落地會議；引用它的僅約五篇、無構造端跟進 | 路線甲零競爭的依據（需每季複查） | 7 月 survey §2.1 |
| AKY24 = ITCS 2025（2024 年流傳） | 統一寫 ITCS 2025 | — |
| Somewhere equivocal encryption = [HJO+16] Hemenway-Jafargholi-Ostrovsky-**Scafuro**-Wichs, CRYPTO 2016 | 作者含 Scafuro（非 Sahai） | — |
| unclonable-IND（Def 12，強概念）one-time 現況：QROM 有 AKLL22；標準模型仍開放（領域大魔王） | compiler 的 one-time 原料現況 | 精讀報告 §3.1 |

---

## 5. 常用縮寫/編號索引

**主定理稿 v3.1 與後續報告**

- **Construction 1**：v3.1 的構造（GKK25 RNC-IB-KEM ＋ OTP 墊層 c₂ ＋ otUE）；**Construction 2／2′／3**：KT18 報告的新構造（RSO 雙重加密槽／加一層遮罩的 SIM-RSO 變體／KDM 雙槽）
- **Theorem 1／2**：v3.1 主定理（搜尋型／不可區分型）；**Theorem 3–6、Proposition 1**：KT18 報告的定理（Construction 2 不可複製／多目標選擇性開啟／KDM-CPA／KDM-unclonable；Construction 2′ 的 SIM-RSO 機密性）
- **Hyb₀／Hyb₁、Claim 1／2**：主定理證明的 hybrid 鏈與兩個歸約（Claim 1 → RNC-IB-KEM 模擬安全；Claim 2 → otUE cloning 遊戲）
- **修訂一～五**：證明反饋給定義／構造的五項修訂（v3.1 §4）——一 memoized（撤回）、二 OTP 墊層（已拍板）、三 QPT 提升檢查清單、四 揭露 sk_{id\*}（老師要求）、五 刪除凍結／表 K／延後引理
- **Definition 2／3**：v3.1 的搜尋型／不可區分型 unclonable 安全；**Definition 2⁺**：再允許 A 在挑戰後、分裂前查詢的加強版；**Definition K1／K2**：KT18 報告的 KDM-CPA／KDM-unclonable 定義（勿與 Deep Dive 的知識補完清單 K1–K12 混淆）
- **查詢階段 I／II／III**：分裂前／分裂後揭露前／揭露後的金鑰查詢（v3.1 Definition 2）
- **Q1–Q5**：KT18 報告 §6 的待拍板問題
- **延後引理**：分裂後查詢延後到揭露後（凍結表報告稱 Lemma D、v3 稱 Lemma 1）——本身正確但不需要，v3.1 已刪除；v3.1 的 Lemma 1 是另一條（OTP 重參數化）

**W1 路線圖**

- **A1–A5**：unclonable IBE 定義設計的五個維度（§1.2）——不可複製性強度／查詢時機／reveal 給什麼／selective vs adaptive／挑戰身分規則
- **D-W1**：路線圖當時唯一被識別的技術硬點——split 後金鑰查詢 vs RNC-IBE 的 stateful Sim₂（§2.3）。**已確認從一開始就不存在**（古典／量子盤點報告 §2.4、延後引理報告）
- **甲案／乙案**：後量子零件的兩條策略——poly-ID 誠實起步／自造 LWE selective RNC-IBE（§4.3）。對現行定義而言被 KT18 報告的 Construction 2 消解（§3.6），乙案只剩 msk 版指數 ID 的用途
- **C1–C4**：W1 路線圖對既有報告的四項更正（§6）

**歷史編號**

- **P1–P3**：路線甲的三條技術路徑（Deep Dive §3）；**D1–D10**：難點總表；**K1–K12**：知識補完清單（Deep Dive §4–5）
- **E1–E4**：AK21 證明真正消耗的槽位介面（精讀報告 §2）——W1 路線圖 §2.4 逐條核對過 RNC-IBE
- **R1–R4**：白板線的四個換槽方向（白板報告 §2–3；R1/R3 已關閉）
- **V1–V3**：白板線剩餘真空格（精讀報告 §4；V1 可當論文理論章）
- **W1–W6**：第二輪換槽候選（精讀報告 §4+）——**W1 = unclonable IBE 已升為主軸**
- **路線甲～戊**：7 月 survey §四的五條候選路線（路線甲 = Unclonable IPFE，現為備援）
