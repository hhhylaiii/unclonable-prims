# Unclonable Primitives — 碩士論文研究儲存庫

本儲存庫為個人**碩士論文研究**用途，主題聚焦於**不可複製密碼學原語（Unclonable Cryptographic Primitives）**，特別是不可複製加密（Unclonable Encryption, UE）與量子函數加密（Quantum Functional Encryption, QFE）兩條主線。日後將陸續放入研究報告、簡報投影片、論文草稿與相關文件。

## 研究近況（2026-07-13 更新）

兩條候選主線經過 2026-07 的精讀與查證後，現況如下：

- **路線甲（主軸候選一）：Unclonable IPFE**——把 MM24 升級階梯的底層換成受限 function class（inner product）。引用地圖確認至今零競爭者；技術路徑以直接合成（P1：`IPFE.Enc(x+r) ⊗ UE.Enc(r)`）先行，最硬的技術點是 D2（合法解密者經 r 洩漏 x）。詳見[路線甲深度分析](./Reports/Deep_Dive_Route_A_Unclonable_IPFE.md)。
- **白板線（原形已關閉，剩餘格子重估中）**：老師白板的「otUE + PQC 換槽」配方，其正解（RNCE 填槽、不經 FE 的公鑰 UE、unclonable-IND 保持）**已被 HKNY24（TCC 2024）Appendix E 先行做掉**。剩餘空間為 V1–V3（compiler 基礎化等）；第二輪盤點發現 **W1：unclonable IBE（RNC-IBE 填槽）** 這格全空，成為與路線甲並列的第二候選主軸。詳見 [AK21 精讀報告](./Reports/AK21_Close_Reading_and_PQC_Slot_Survey.md)。
- **不可複製加密的整體地圖**：從 Broadbent-Lord (2020) 的奠基性定義，到 Bhattacharyya-Broadbent-Culf (2026) 無條件不可複製位元的里程碑結果；資訊論端已被高速收割完畢，QFE 構造端仍是全地圖最大空白。

論文的具體題目尚未定案，目前的優先序：
1. **路線甲**：Unclonable IPFE（主軸候選一，零競爭、保底較厚）
2. **W1**：Unclonable IBE via RNC-IBE 換槽（主軸候選二，證明可套 HKNY24 模板）
3. V1：UE compiler 的介面刻畫與可模糊化必要性（適合當論文一章，非主軸）

> 📖 **報告閱讀指南**：報告數量已多且部分結論被後續報告更正，請先看 [Reports/README.md](./Reports/README.md) 的閱讀順序、報告關係圖與關鍵事實速查表。

## 內容索引

### 研究報告

完整的閱讀順序、報告間關係與更正狀態，見 [Reports/README.md](./Reports/README.md)。

- [Meeting 討論報告：兩條候選主軸（2026-07-15）](./Reports/Meeting_2026-07-15_Two_Routes_Discussion.md)
  — 給老師的 high-level 討論文件：Unclonable IPFE 與 Unclonable IBE 兩條路線各自「要做什麼、為什麼可以、大概的方式」、兩線對照表、三個論文骨架選項與待拍板問題清單。**（meeting 前唯一必讀）**
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
  — 逐頁精讀 AK21 全文（Def 16 偽金鑰、§4/§5 兩個構造與證明的完整拆解、shared-randomness 歸約技巧），歸納證明真正消耗的介面 E1–E4，發現 Def 16 過強、trapdoored 可模糊化（RNCE 形狀）即足夠——**並查證出此觀察已被 [HKNY24]（TCC 2024）Appendix E 實現**（RNCE 填槽、unclonable-IND 保持），修正三份既有報告的盲點。盤點剩餘真空格 V1–V3 與第二輪候選 W1–W6（首選 W1：unclonable IBE），建議主軸轉回路線甲、以 HKNY24 為 P1 證明模板。**（最新結論所在）**
- [白板解碼：AK21 的「otUE + PQC」配方與重構 UE 的換槽方向](./Reports/AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md)
  — 把老師白板公式對應到 AK21 §1.2 的 hybrid approach（otUE 量子核心 + 可替換的 PQC 槽），釐清 PQC 槽的真實介面（後量子安全 + 偽金鑰性質）。在「輸出仍是 UE」的正確讀法下盤點四個換槽方向 R1–R4（公鑰偽金鑰不經 FE／compiler 解耦／unclonable-IND 版／一對多偽金鑰）與已關閉格子，並釐清白板線與路線甲是輸出物不同的兩條線、共用偽金鑰零件。含 meeting 用的定錨問題與題目提案。（R1/R3 新穎性判斷已被 v3 更正註記推翻）

### 簡報與論文（規劃中）

- `slides/` — Lab meeting 與 thesis proposal 簡報
- `thesis/` — 論文草稿與相關 LaTeX 檔案（待新增）
- `papers/` — 相關論文

## 背景

不可複製加密利用量子不可複製定理（no-cloning theorem）產生無法被兩個獨立解密者同時複製的密文，是後量子密碼學中近年最活躍的子領域之一。Ananth-Kaleoglu (TCC 2021) 提出的**偽金鑰性質**至今仍是公鑰 UE 中最核心的開放挑戰——在不使用函數加密或不可區分混淆等重型工具的情況下，目前尚無任何構造能透過直接偽金鑰論證實現公鑰 UE。
