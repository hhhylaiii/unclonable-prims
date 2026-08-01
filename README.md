# Unclonable Primitives — 碩士論文研究儲存庫

本儲存庫為個人**碩士論文研究**用途，主題聚焦於**不可複製密碼學原語（Unclonable Cryptographic Primitives）**。碩論主軸已於 2026-08 定案為 **Unclonable IBE（不可複製身分基加密）**；儲存庫同時保留通往此定案的完整探索紀錄（不可複製加密 UE 與量子函數加密 QFE 兩條線的 survey 與深度分析）。

## 研究近況（2026-08-01 更新）

### 主軸已定案：路線 W1 —— Unclonable IBE（不可複製身分基加密）

造一個文獻中不存在的原語：**不可複製的身分基加密**。密文含量子態，即使兩個分裂的對手事後都拿到挑戰身分的金鑰（甚至整把 master secret key），也無法同時解密。

```
Enc(mpk, id, m):  k ← otUE.Setup
                  ct = ( RNCIBE.Enc(mpk, id, k) ,  otUE.Enc(k, m) )
```

沿用 AK21／HKNY24 的 KEM-DEM 配方，古典槽換成**接收者非承諾 IBE（RNC-IBE，PKC 2025）**。證明套 HKNY24 App E 的現成模板；實質工作在**定義設計**（cloning game 中身分金鑰查詢的時機）與**後量子零件**（RNC-IBE 主構造是雙線性的，唯一現成的後量子選項僅支援多項式大小身分空間）。

當前決策點：後量子零件走「poly-ID 誠實起步」還是「自造 LWE 版 RNC-IBE 當技術核心」——**兩案並陳，待一週試探後拍板**。

完整技術地圖見 **[路線 W1 技術路線圖](./Reports/Route_W1_Unclonable_IBE_Roadmap.md)（現行主軸文件）**。

### 其他線的狀態

- **路線甲：Unclonable IPFE**（**降為備援／延伸章**）——把 MM24 升級階梯的底層換成受限 function class（inner product）。引用地圖確認零競爭者，但技術風險較高（D2：合法解密者經 r 洩漏 x）。保留為 W1 撞牆時的退路與論文延伸章；其副產品 Clifford 直構 no-go 短文仍是隨時可交付的成果。詳見[路線甲深度分析](./Reports/Deep_Dive_Route_A_Unclonable_IPFE.md)。
- **白板線（原形已關閉）**：老師白板的「otUE + PQC 換槽」配方，其正解（RNCE 填槽、不經 FE 的公鑰 UE、unclonable-IND 保持）**已被 HKNY24（TCC 2024）Appendix E 先行做掉**。但第二輪盤點發現 **W1：unclonable IBE（RNC-IBE 填槽）** 這格全空——**現行主軸即由此長出**。詳見 [AK21 精讀報告](./Reports/AK21_Close_Reading_and_PQC_Slot_Survey.md)。
- **V1：UE compiler 的介面刻畫與可模糊化必要性**——仍空，適合作為論文的理論章，與 W1 共用全部定義與工具。
- **不可複製加密的整體地圖**：從 Broadbent-Lord (2020) 的奠基性定義，到 Bhattacharyya-Broadbent-Culf (2026) 無條件不可複製位元的里程碑結果；資訊論端已被高速收割完畢，構造端仍有空白。

### 目前的工作順位

1. **W1 定義章**：unclonable IBE 的 cloning game（身分金鑰查詢在 split 前／後的給法）
2. **D-W1 驗證**：「reveal 給 msk ⇒ split 後查詢自足」這個命題成不成立
3. **pq 零件試探**：ABB／GPV 陷門能否配上 RNCE 式的 Fake/Reveal（決定甲案／乙案）
4. 主定理 W1 的正式陳述與三段證明
5. （平行）路線甲的 Clifford no-go 短文——最接近完成的可交付成果

> **報告閱讀指南**：報告數量已多且部分結論被後續報告更正，請先看 [Reports/README.md](./Reports/README.md) 的閱讀順序、報告關係圖與關鍵事實速查表。

## 內容索引

### 研究報告

完整的閱讀順序、報告間關係與更正狀態，見 [Reports/README.md](./Reports/README.md)。

- [路線 W1 技術路線圖：Unclonable IBE](./Reports/Route_W1_Unclonable_IBE_Roadmap.md)
  — **現行主軸文件**。目標原語語義、定義設計的五個維度（A1–A5）、構造與主定理形狀、三段證明骨架與唯一硬點 D-W1、RNC-IBE 精讀的關鍵發現（Def 6 交出的是 msk）、後量子零件的兩案並陳、與 KN23／HMNY21 的邊界、執行計畫與退場條件。**（想知道現在在做什麼，只讀這一份）**
- [Meeting 討論筆記：RNC-IBE 精讀（2026-08）](./Reports/Meeting_2026-08_W1_RNCIBE_Discussion.md)
  — 下次個人 meeting 的討論素材：RNC-IBE 精讀長出的五個討論點（介面對上、Def 6 給 msk、D-W1、後量子零件實況、KEM 介面）、四個待拍板問題（混淆電路的範圍、pq 策略、定義取捨、佔位時機）與論文閱讀範圍表。
- [Meeting 討論報告：兩條候選主軸（2026-07-15）](./Reports/Meeting_2026-07-15_Two_Routes_Discussion.md)
  — 給老師的 high-level 討論文件：Unclonable IPFE 與 Unclonable IBE 兩條路線各自「要做什麼、為什麼可以、大概的方式」、兩線對照表、三個論文骨架選項與待拍板問題清單。（歷史紀錄：主軸已於 2026-08-01 定案為 W1，其 §4 的骨架選項由路線圖取代）
- [不可複製加密的論文方向：五個研究前沿](./Reports/Research_Directions_of_Unclonable_Encryption.md)
  — 涵蓋公鑰偽金鑰、UE 演進史、偽金鑰與 NCE 的三角關係、不可複製函數加密、一對多偽金鑰推廣等五個方向的詳細分析。（方向一/三的新穎性判斷已被 2026-07 精讀報告更正，見文首更正註記）
- [量子函數加密碩論方向 Survey](./Reports/Research_Directions_of_Quantum_Functional_Encryption.md)
  — 整理老師建議的兩個延伸方向、推薦閱讀清單、以及「升級階梯 compiler」的研究構想。（部分構想已被路線甲深度分析更新，見文首狀態註記）
- [不可複製加密與難以捉摸的公鑰偽金鑰性質](./Reports/public_key_encryption_with_fake-key_property_report.md)
  — 深入調查為何偽金鑰性質至今未被推廣至公鑰設定的結構性障礙，與相關古典可模糊性概念在 UE 證明中的角色。（核心判斷已被 HKNY24 App E 發現更正，見文首更正註記）
- [2026 年 7 月文獻總盤點：近期發展與可著手的研究路線](./Reports/Survey_Recent_Developments_and_New_Routes_2026-07.md)
  — 以已精讀的四篇論文為錨點，盤點 2025 下半年至 2026 年 7 月的最新進展（BMMS26 不可能性、不可複製位元、HROM UE、多複製安全上 CRYPTO 2026 等），分析空白區與擁擠區，並評估五條候選研究路線的優先序。
- [路線甲深度分析：從 IPFE 到 Unclonable IPFE](./Reports/Deep_Dive_Route_A_Unclonable_IPFE.md)
  — 釘死目標語義、逐條盤點 MM24 三個定理的介面需求（發現 Thm 7 lifting 消耗 universality）、提出三條技術路徑（直接合成 / 重建階梯 / Clifford 直構與其可逆性洩漏攻擊）、難點總表 D1–D10 與知識補完清單 K1–K12、12–16 週執行計畫。
- [AK21 精讀與 PQC 槽替換品總盤點](./Reports/AK21_Close_Reading_and_PQC_Slot_Survey.md)
  — 逐頁精讀 AK21 全文（Def 16 偽金鑰、§4/§5 兩個構造與證明的完整拆解、shared-randomness 歸約技巧），歸納證明真正消耗的介面 E1–E4，發現 Def 16 過強、trapdoored 可模糊化（RNCE 形狀）即足夠——**並查證出此觀察已被 [HKNY24]（TCC 2024）Appendix E 實現**（RNCE 填槽、unclonable-IND 保持），修正三份既有報告的盲點。盤點剩餘真空格 V1–V3 與第二輪候選 W1–W6（首選 W1：unclonable IBE）。**（W1 的出處；其 §4+ 是現行主軸的起點）**
- [白板解碼：AK21 的「otUE + PQC」配方與重構 UE 的換槽方向](./Reports/AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md)
  — 把老師白板公式對應到 AK21 §1.2 的 hybrid approach（otUE 量子核心 + 可替換的 PQC 槽），釐清 PQC 槽的真實介面（後量子安全 + 偽金鑰性質）。在「輸出仍是 UE」的正確讀法下盤點四個換槽方向 R1–R4（公鑰偽金鑰不經 FE／compiler 解耦／unclonable-IND 版／一對多偽金鑰）與已關閉格子，並釐清白板線與路線甲是輸出物不同的兩條線、共用偽金鑰零件。含 meeting 用的定錨問題與題目提案。（R1/R3 新穎性判斷已被 v3 更正註記推翻）

### 論文（papers/）

主軸相關的核心讀物：

- **Non-Committing Identity Based Encryption: Constructions and Applications**（GKKNRY，PKC 2025）— W1 的零件來源。另有 [繁中全文詳解](./papers/RNC-IBE_全文詳解_繁中.pdf)（涵蓋 Def 1–8、三個構造、Thm 1–15 與全部 hybrid 論證）
- **Unclonable Encryption, Revisited**（AK21，TCC 2021）— 配方來源
- **Cloning Games: A General Framework for Unclonable Primitives**（AKL23）— 定義語言；§8.3 含 UE with certified deletion
- **Unclonable Functional Encryption**（MM24）— 路線甲（備援）的對象
- **Simple Functional Encryption Schemes for Inner Products**（ALS16）／**KDM & RSO Security for IBE** — 支援讀物

### 簡報與論文草稿（規劃中）

- `slides/` — Lab meeting 與 thesis proposal 簡報
- `thesis/` — 論文草稿與相關 LaTeX 檔案（待新增）

## 背景

不可複製加密利用量子不可複製定理（no-cloning theorem）產生無法被兩個獨立解密者同時複製的密文，是後量子密碼學中近年最活躍的子領域之一。Ananth-Kaleoglu (TCC 2021) 建立的「量子核心 + 古典可模糊化槽」配方，把不可複製性的來源與存取結構的來源乾淨地分離開來；本研究的主軸即是把該槽位換成非承諾的身分基加密，得到**不可複製身分基加密**——把不可複製性帶進「有金鑰管理結構」的世界的第一步。
