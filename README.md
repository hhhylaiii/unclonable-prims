# Unclonable Primitives — 碩士論文研究儲存庫

本儲存庫為個人**碩士論文研究**用途，主題聚焦於**不可複製密碼學原語（Unclonable Cryptographic Primitives）**，特別是不可複製加密（Unclonable Encryption, UE）與量子函數加密（Quantum Functional Encryption, QFE）兩條主線。日後將陸續放入研究報告、簡報投影片、論文草稿與相關文件。

## 研究近況

目前正在閱讀並整理以下方向的文獻：

- **量子函數加密的「升級階梯」**：以 Mehta & Müller (2024) 的 *Unclonable Functional Encryption* 為起點，思考是否能把底層元件（QGC、TwoFE 等）替換為更受限的 function class（如 IPFE、quadratic FE），得到更具體、更有效率的 unclonable 構造。
- **公鑰偽金鑰性質（Public-Key Fake-Key Property）**：自 Ananth-Kaleoglu (TCC 2021) 提出私鑰偽金鑰以來，公鑰類比至今仍是 open problem。整理了 2021–2026 年間的進展，以及與接收者可否認加密、非承諾加密（NCE）等古典概念的關聯。
- **不可複製加密的整體地圖**：從 Broadbent-Lord (2020) 的奠基性定義，到 Bhattacharyya-Broadbent-Culf (2026) 無條件不可複製位元的里程碑結果，梳理該領域六年來的演進與剩餘的關鍵 open questions。

論文的具體題目尚未定案，預期會落在以下其中一條路徑：
1. QFE for restricted classes（如 inner-product、quadratic）並推到 unclonable 版本
2. 公鑰偽金鑰性質的正式定義與構造（可能借助量子技術繞過古典不可能性）
3. 研究不可複製加密構造

## 內容索引

### 研究報告

- [不可複製加密的論文方向：五個研究前沿](./Research_Directions_of_Unclonable_Encryption.md)
  — 涵蓋公鑰偽金鑰、UE 演進史、偽金鑰與 NCE 的三角關係、不可複製函數加密、一對多偽金鑰推廣等五個方向的詳細分析。
- [量子函數加密碩論方向 Survey](./Research_Directions_of_Quantum_Functional_Encryption.md)
  — 整理老師建議的兩個延伸方向、推薦閱讀清單、以及「升級階梯 compiler」的研究構想。
- [不可複製加密與難以捉摸的公鑰偽金鑰性質](./public_key_encryption_with_fake-key_property_report.md)
  — 深入調查為何偽金鑰性質至今未被推廣至公鑰設定的結構性障礙，與相關古典可模糊性概念在 UE 證明中的角色。
- [2026 年 7 月文獻總盤點：近期發展與可著手的研究路線](./Survey_Recent_Developments_and_New_Routes_2026-07.md)
  — 以已精讀的四篇論文為錨點，盤點 2025 下半年至 2026 年 7 月的最新進展（BMMS26 不可能性、不可複製位元、HROM UE、多複製安全上 CRYPTO 2026 等），分析空白區與擁擠區，並評估五條候選研究路線的優先序。
- [路線甲深度分析：從 IPFE 到 Unclonable IPFE](./Deep_Dive_Route_A_Unclonable_IPFE.md)
  — 釘死目標語義、逐條盤點 MM24 三個定理的介面需求（發現 Thm 7 lifting 消耗 universality）、提出三條技術路徑（直接合成 / 重建階梯 / Clifford 直構與其可逆性洩漏攻擊）、難點總表 D1–D10 與知識補完清單 K1–K12、12–16 週執行計畫。

### 簡報與論文（規劃中）

- `slides/` — Lab meeting 與 thesis proposal 簡報
- `thesis/` — 論文草稿與相關 LaTeX 檔案（待新增）
- `papers/` — 相關論文

## 背景

不可複製加密利用量子不可複製定理（no-cloning theorem）產生無法被兩個獨立解密者同時複製的密文，是後量子密碼學中近年最活躍的子領域之一。Ananth-Kaleoglu (TCC 2021) 提出的**偽金鑰性質**至今仍是公鑰 UE 中最核心的開放挑戰——在不使用函數加密或不可區分混淆等重型工具的情況下，目前尚無任何構造能透過直接偽金鑰論證實現公鑰 UE。
