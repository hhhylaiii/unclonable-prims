# Unclonable Primitives — 碩士論文研究儲存庫

本儲存庫為個人**碩士論文研究**用途，主題聚焦於**不可複製密碼學原語（Unclonable Cryptographic Primitives）**。碩論主軸已於 2026-08 定案為 **Unclonable IBE（不可複製身分基加密）**；儲存庫同時保留通往此定案的完整探索紀錄（不可複製加密 UE 與量子函數加密 QFE 兩條線的 survey 與深度分析）。

## 研究近況（2026-09-28 更新）

### 主軸：路線 W1 —— Unclonable IBE（不可複製身分基加密）

造一個文獻中不存在的原語：**不可複製的身分基加密**。密文含量子態，即使兩個分裂的對手事後都拿到挑戰身分的金鑰 sk_{id\*}，也無法同時解密（Construction 1 在交出整把 master secret key 時仍成立）。

**主定理已有完整證明**——[主定理稿 v3.1](./Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（2026-09-27）：

```
Construction 1:  (ct₁, k̄) ← IBKEM.Encap(mpk, id);  k ← otUE.Setup;  c₂ := k̄ ⊕ k;  ρ ← otUE.Enc(k, m)
                 ct := (ct₁, c₂, ρ)                    IBKEM = GKK25 的 RNC-IB-KEM（Def 8）
```

證明是一條 hybrid 鏈、兩個身分相反的歸約：Claim 1 歸約到 RNC-IB-KEM 的模擬安全，Claim 2 把中間實驗包成 otUE cloning 遊戲的對手，t 無損傳遞（Theorem 1 搜尋型、Theorem 2 不可區分型）。已拍板：OTP 墊層 c₂（修訂二）、揭露物交 sk_{id\*}（修訂四，老師要求；揭露整把 msk 為 Remark 3 的強化版）。原先唯一被識別的硬點 D-W1 確認從一開始就不存在，凍結表 K 與延後引理都已刪除，主定理對任意身分空間成立。後量子實例目前只有 GKK25 Thm 3/14 取 LWE（poly-ID）；SXDH／DDH 實例對量子對手不安全，不列入。

**最新進展（2026-09-28）：KT18 機制 × 不可複製加密**——[報告](./Reports/KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md)／[構造、定理與證明（PDF）](./Paper%20drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf)：

- **Construction 2**：以 KT18 的 RSO 雙重加密當 k 的槽——**任意**後量子 IND-ID-CPA IBE ＋ t-unclonable otUE ⇒ t-unclonable UIBE，證明不用任何模擬器；以 LWE adaptive IBE 實例化即得**指數身分空間**的後量子 UIBE。對現行定義而言，路線圖的甲案／乙案困境消失；揭露 msk 的強化版仍只由 Construction 1 提供。
- **Construction 3（雙槽）**：k 走 RSO 槽、被遮罩的訊息走 KT18 的 KDM 槽，同時達到 KDM-CPA 與 **KDM-unclonable**——文獻中沒有的新組合。

當前決策點：KT18 報告 §6 的 Q1–Q5 待老師拍板（主定理是否改為兩個實例化定理、是否採用允許挑戰後查詢的 Definition 2⁺、KDM-unclonable 的定義形狀等）；v3.1 Definition 2 的查詢階段 II／III 與 Remark 2 的時間邊界待老師確認。

整體技術地圖（定義維度 A1–A5、與既有工作的邊界）見 **[Unclonable IBE 主軸路線圖](./Reports/Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)**——其 D-W1 與甲乙案已被後續報告更新，見文首狀態註記。

### 其他線的狀態

- **路線甲：Unclonable IPFE**（**降為備援／延伸章**）——把 MM24 升級階梯的底層換成受限 function class（inner product）。引用地圖確認零競爭者，但技術風險較高（D2：合法解密者經 r 洩漏 x）。保留為 W1 撞牆時的退路與論文延伸章；其副產品 Clifford 直構 no-go 短文仍是隨時可交付的成果。詳見[路線甲深度分析](./Reports/Unclonable_IPFE_Backup_Route_Paths_Obstacles_Knowledge_Gaps.md)。
- **白板線（原形已關閉）**：老師白板的「otUE + PQC 換槽」配方，其正解（RNCE 填槽、不經 FE 的公鑰 UE、unclonable-IND 保持）**已被 HKNY24（TCC 2024）Appendix E 先行做掉**。但第二輪盤點發現 **W1：unclonable IBE（RNC-IBE 填槽）** 這格全空——**現行主軸即由此長出**。詳見 [AK21 精讀報告](./Reports/AK21_Close_Reading_Slot_Interfaces_and_Route_Candidates.md)。
- **V1：UE compiler 的介面刻畫與可模糊化必要性**——仍空，適合作為論文的理論章，與 W1 共用全部定義與工具。
- **不可複製加密的整體地圖**：從 Broadbent-Lord (2020) 的奠基性定義，到 Bhattacharyya-Broadbent-Culf (2026) 無條件不可複製位元的里程碑結果；資訊論端已被高速收割完畢，構造端仍有空白。

### 目前的工作順位

1. **Meeting**：報告 KT18 報告的 Construction 2，請老師回答 Q1–Q5；確認 v3.1 Definition 2 的查詢階段 II／III 與 Remark 2
2. **定義章同步**：[定義章草稿](./Reports/Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md)的 Definition A/B 改為 v3.1 Definition 2／3 的形式（揭露 sk_{id\*}、查詢階段 II／III、標準預言機、Setup(1^λ, 1^d)）；把「使用者金鑰層級 RNC」寫成介面定義，讓 Construction 1、2 都成為實例
3. **併入論文主文**：v3.1 的證明併入主文並補齊 Theorem 2（不可區分型）的細節；拍板後併入 KT18 正式稿（Construction 2/2′/3、Theorem 3–6、Proposition 1）
4. **QPT 提升附錄**（修訂三）：對 GKK25 Lemma 12–14 逐條檢查；確認所選 LWE IBE 的 adaptive 證明為 straight-line
5. **新穎性複查**：unclonable × {IBE, KDM, selective opening, non-committing} 的 ePrint 全文檢索；考慮盡早把「定義＋Construction 2 的定理」掛 ePrint
6. （平行）路線甲的 Clifford no-go 短文

> **報告閱讀指南**：報告數量已多且部分結論被後續報告更正。各報告的檔名與標題即為其大綱；完整的閱讀順序、報告關係圖與關鍵事實速查表見 [Reports/README.md](./Reports/README.md)。

## 內容索引

### 研究報告

完整的閱讀順序、報告間關係與更正狀態，見 [Reports/README.md](./Reports/README.md)。

**主定理與證明（2026-08-28 起，由新到舊）**

- [KT18 × 不可複製加密：RSO 機制直接填槽（任意 IBE ⇒ UIBE）、KDM 機制的雙槽設計、兩個新安全概念與證明骨架](./Reports/KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md)
  — **最新**（2026-09-28）。KT18 精讀（KDM／RSO 兩個機制與共同骨架）；Construction 2：RSO 雙重加密當 k 的槽，任意後量子 IND-ID-CPA IBE ⇒ UIBE（指數身分空間、允許挑戰後查詢）；RSO 延伸（多目標選擇性開啟，Theorem 4、Proposition 1）；KDM 放進 cloning game 的三個陷阱與 Construction 3（雙槽，Theorem 5/6）；三個構造總對照與待拍板問題 Q1–Q5。
- [Lemma 1（分裂後查詢延後）不需要：誰在回答查詢、兩個揭露事件、兩階段遊戲的文獻做法與真正的時間邊界](./Reports/Deferred_Query_Lemma_Unnecessary_Who_Answers_Queries_Two_Reveals.md)
  — 2026-09-27。四個遊戲逐一核對「誰在回答查詢」；UIBE 的揭露（sk_{id\*}）與 otUE 的揭露（k）是兩個不同事件；真正的時間邊界是分裂而非揭露；文獻對照（GKK25 Thm 15、HMNY21）。**主定理稿 v3.1 由此產生**。
- [老師的兩個提問：系統內的古典／量子物件盤點；「不得查詢 id\*」規則與 non-adaptive 選項的評估](./Reports/Classical_vs_Quantum_Inventory_and_Challenge_Identity_Query_Rule_Evaluation.md)
  — 2026-09-13。只有 otUE 密文 ρ 與對手暫存器是量子的；分裂前禁查 id\* 是必要的；adaptive 不必退成 selective；附帶重大更正：D-W1 從一開始就不是問題。（SXDH／DDH 實例的部分已更正；「分裂後不設限」提案未被 v3.1 採用，見文首註記）
- [換掉 Hyb₁ 的凍結表 K：分裂後查詢的「延後引理」、四條替代路線的評估與文獻對照](./Reports/Replacing_Frozen_Key_Table_in_Hyb1_Deferred_Query_Lemma_and_Alternatives.md)
  — 2026-09-12。回應老師「凍結表 K 會碰到指數大的東西」：表 K 的兩個功能、延後引理、替代方案對照、文獻中分裂後預言機的處理。（機制已被古典／量子盤點報告的更簡單事實取代，見文首更正註記）
- [主定理證明設計：一條 Hybrid 鏈、兩個身分相反的歸約（自足版）](./Reports/Proof_Design_Main_Theorem_Two_Reductions_and_Hybrid_Chain.md)
  — 2026-08-28。證明只有兩步（Claim 1 歸約到 IBKEM 模擬安全、Claim 2 歸約到 otUE cloning 遊戲）、中間實驗 Hyb₁ 的精確定義、三項修訂提案。（已由主定理稿 v3.1 取代，見文首狀態註記）

**主軸的路線圖、定義與 meeting 筆記**

- [Unclonable IBE 主軸路線圖：定義設計五維度、KEM-DEM 構造與主定理、三段證明骨架與硬點 D-W1、後量子零件兩案](./Reports/Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)
  — **主軸技術總圖**。目標原語語義、定義設計的五個維度（A1–A5）、構造與主定理形狀、三段證明骨架與硬點 D-W1、RNC-IBE 精讀的關鍵發現（Def 6 交出的是 msk）、後量子零件的兩案並陳、與 KN23／HMNY21 的邊界、執行計畫與退場條件。（D-W1 已確認不存在、構造與揭露物已更新、甲乙案的前提被 KT18 報告改變，見文首狀態註記）
- [Unclonable IBE 定義章草稿：game-based 上層與 simulation-based 槽位如何分層、四個模擬器對應、定義 A/B 與 KEM 版槽位介面](./Reports/Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md)
  — 路線圖 §1–§2 的細化：為何上層用 cloning game、下層槽位必須用 stateful 模擬器；GKK25 四個模擬器逐一對應到 AK21 的哪一步（fake-key 的對應物是 Sim₄ 而非 Sim₂）；用 split 線重新理解 A1–A5、A2 的三個解；Definition A（搜尋型 t-unclonable）與 Definition B（unclonable-IND）草稿；主定理改以 RNC-IB-KEM（Def 8）為介面的理由。（定義 A/B 待同步為 v3.1 的 Definition 2／3，見文首狀態註記）
- [Meeting 筆記（2026-08）：RNC-IBE 介面核對的五個發現與三個待拍板問題](./Reports/Meeting_2026-08_RNC_IBE_Interface_Check_and_Open_Decisions.md)
  — 個人 meeting 的討論素材：五個發現（E1–E4 逐條對上、Def 6 交出的是 msk、D-W1 的三條出路、後量子只有 Thm 3 可用且為何後量子是必要條件、主定理改走 KEM 介面），與三個待拍板問題（不用混淆電路的範圍是否含古典 Yao、pq 甲案／乙案、split 後金鑰查詢要不要寫進定義）。（待拍板問題的後續見文首狀態註記）
- [Meeting 報告（2026-07）：兩條候選主軸的做法、可行性、對照表與三個論文骨架選項](./Reports/Meeting_2026-07_Two_Candidate_Routes_IPFE_vs_IBE.md)
  — 給老師的 high-level 討論文件：Unclonable IPFE 與 Unclonable IBE 兩條路線各自「要做什麼、為什麼可以、大概的方式」、兩線對照表、三個論文骨架選項與待拍板問題清單。（歷史紀錄：主軸已於 2026-08-01 定案為 W1，其 §4 的骨架選項由路線圖取代）

**探索紀錄（通往 W1 的歷史）**

- [不可複製加密的五個研究前沿：公鑰偽金鑰、UE 構造演進史、偽金鑰與 NCE／可否認加密的三角關係、不可複製函數加密、一對多偽金鑰推廣](./Reports/Unclonable_Encryption_Five_Research_Frontiers_Map.md)
  — 涵蓋公鑰偽金鑰、UE 演進史、偽金鑰與 NCE 的三角關係、不可複製函數加密、一對多偽金鑰推廣等五個方向的詳細分析。（方向一/三的新穎性判斷已被 2026-07 精讀報告更正，見文首更正註記）
- [量子函數加密方向 Survey：老師建議的兩個延伸方向、推薦閱讀清單與「升級階梯 compiler」的研究構想](./Reports/Quantum_FE_Directions_and_Upgrade_Ladder_Idea.md)
  — 整理老師建議的兩個延伸方向、推薦閱讀清單、以及「升級階梯 compiler」的研究構想。（部分構想已被路線甲深度分析更新，見文首狀態註記）
- [偽金鑰性質為何至今沒有公鑰版：AK21 Def 16 的結構性障礙、公鑰 UE 的替代範式盤點與古典可模糊性概念的角色](./Reports/Fake_Key_Property_Why_No_Public_Key_Version_Exists.md)
  — 深入調查為何偽金鑰性質至今未被推廣至公鑰設定的結構性障礙，與相關古典可模糊性概念在 UE 證明中的角色。（核心判斷已被 HKNY24 App E 發現更正，見文首更正註記）
- [文獻總盤點（2026-07）：MM24 之後的最新結果、空白區與擁擠區地圖、五條候選路線的優先序](./Reports/Literature_Survey_2026-07_Results_Gaps_and_Route_Ranking.md)
  — 以已精讀的四篇論文為錨點，盤點 2025 下半年至 2026 年 7 月的最新進展（BMMS26 不可能性、不可複製位元、HROM UE、多複製安全上 CRYPTO 2026 等），分析空白區與擁擠區，並評估五條候選研究路線的優先序。
- [Unclonable IPFE 備援路線深潛：目標語義釘死、MM24 三定理的介面需求、三條技術路徑 P1–P3、難點總表 D1–D10 與知識補完清單 K1–K12](./Reports/Unclonable_IPFE_Backup_Route_Paths_Obstacles_Knowledge_Gaps.md)
  — 釘死目標語義、逐條盤點 MM24 三個定理的介面需求（發現 Thm 7 lifting 消耗 universality）、提出三條技術路徑（直接合成 / 重建階梯 / Clifford 直構與其可逆性洩漏攻擊）、難點總表 D1–D10 與知識補完清單 K1–K12、12–16 週執行計畫。
- [AK21 逐頁精讀與槽位替換品盤點：介面 E1–E4、HKNY24 App E 查證、剩餘真空格 V1–V3 與第二輪候選 W1–W6](./Reports/AK21_Close_Reading_Slot_Interfaces_and_Route_Candidates.md)
  — 逐頁精讀 AK21 全文（Def 16 偽金鑰、§4/§5 兩個構造與證明的完整拆解、shared-randomness 歸約技巧），歸納證明真正消耗的介面 E1–E4，發現 Def 16 過強、trapdoored 可模糊化（RNCE 形狀）即足夠——**並查證出此觀察已被 [HKNY24]（TCC 2024）Appendix E 實現**（RNCE 填槽、unclonable-IND 保持），修正三份既有報告的盲點。盤點剩餘真空格 V1–V3 與第二輪候選 W1–W6（首選 W1：unclonable IBE）。**（W1 的出處；其 §4+ 是現行主軸的起點）**
- [白板配方解碼：「otUE + PQC」對應到 AK21 §1.2 的混合構造、PQC 槽的真實介面、四個換槽方向 R1–R4 與白板線／路線甲的分界](./Reports/Whiteboard_Recipe_Decoded_and_PQC_Slot_Swap_Directions.md)
  — 把老師白板公式對應到 AK21 §1.2 的 hybrid approach（otUE 量子核心 + 可替換的 PQC 槽），釐清 PQC 槽的真實介面（後量子安全 + 偽金鑰性質）。在「輸出仍是 UE」的正確讀法下盤點四個換槽方向 R1–R4（公鑰偽金鑰不經 FE／compiler 解耦／unclonable-IND 版／一對多偽金鑰）與已關閉格子，並釐清白板線與路線甲是輸出物不同的兩條線、共用偽金鑰零件。含 meeting 用的定錨問題與題目提案。（R1/R3 新穎性判斷已被 v3 更正註記推翻）

### 論文草稿（Paper drafts/）

各 PDF 的 LaTeX 原始檔未收入本儲存庫。

- [主定理稿 v3.1：Unclonable IBE 主定理與證明——一條 Hybrid 鏈、兩個身分相反的歸約](./Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（2026-09-27）
  — **現行主定理文件**：語法、Definition 2／3、Construction 1、Theorem 1／2 與完整證明、修訂一～五的拍板狀態與下一步。
- [UIBE 兩個歸約圖解](./Paper%20drafts/UIBE%20兩個歸約圖解.html)
  — 主定理稿 v3.1 第 3 節兩個歸約（Claim 1、Claim 2）的圖解，以瀏覽器開啟。
- [KT18 機制 × 不可複製加密：構造、定理與證明](./Paper%20drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf)（2026-09-28，討論用草稿）
  — Construction 2/2′/3、Theorem 3–6、Proposition 1 的正式陳述與證明；設計面的討論見 KT18 報告。
- [Unclonable IBE 安全性定義草稿：上層 Cloning Game 與下層模擬器介面的分層設計](./Paper%20drafts/Draft_of_Security_Definition_of_Unclonable_IBE.pdf)（2026-08-14）
  — 定義章草稿的 PDF 版（定義 A/B 為揭露 msk 的版本，待同步為 v3.1 的 Definition 2／3）。

### 論文（papers/）

主軸相關的核心讀物：

- **Non-Committing Identity Based Encryption: Constructions and Applications**（GKK25，PKC 2025）— Construction 1 的槽位（RNC-IB-KEM：Def 8；LWE 實例：Thm 3/14）。另有繁中全文詳解 `RNC-IBE_全文詳解_繁中.pdf`（涵蓋 Def 1–8、三個構造、Thm 1–15 與全部 hybrid 論證；未收入本儲存庫）
- **Key Dependent Message Security and Receiver Selective Opening Security for Identity-Based Encryption**（KT18，PKC 2018）— Construction 2（RSO 雙重加密）與 Construction 3（KDM 槽）的機制來源
- **Unclonable Encryption, Revisited**（AK21，TCC 2021）— 配方與證明模板來源
- **Uncloneable Quantum Encryption via Oracles**（BL20，TQC 2020）— otUE 的實例（conjugate encryption）
- **Cloning Games: A General Framework for Unclonable Primitives**（AKL23）— 定義語言；§8.3 含 UE with certified deletion
- **Unclonable Functional Encryption**（MM24）— 路線甲（備援）的對象

### 簡報（slides/）

- `slides/` — Lab meeting 與 thesis proposal 簡報。目前收錄四份論文簡報（pptx）：Cloning Games、Unclonable Encryption Revisited、Unclonable Functional Encryption、Simple Functional Encryption Schemes for Inner Products（ALS16；其原文 PDF 已自 `papers/` 移除）

## 背景

不可複製加密利用量子不可複製定理（no-cloning theorem）產生無法被兩個獨立解密者同時複製的密文，是後量子密碼學中近年最活躍的子領域之一。Ananth-Kaleoglu (TCC 2021) 建立的「量子核心 + 古典可模糊化槽」配方，把不可複製性的來源與存取結構的來源乾淨地分離開來；本研究的主軸即是把該槽位換成非承諾的身分基加密，得到**不可複製身分基加密**——把不可複製性帶進「有金鑰管理結構」的世界的第一步。
