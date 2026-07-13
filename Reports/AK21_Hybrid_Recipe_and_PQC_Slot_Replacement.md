# 白板解碼：AK21 的「otUE + PQC」混合配方，與「換掉 PQC、重構一個 UE」的可行方向

> ⚠️ **v3 更正（2026-07-12 深入查證後）**：本文 §3 的 R1（公鑰偽金鑰不經 FE，標為「5 年 open、主推」）與 R3（unclonable-IND 版 compiler，標為「檢查點」）的新穎性判斷**已被推翻**——[HKNY24]（arXiv:2311.09487，TCC 2024）Appendix E 已用 RNCE 構造 unclonable PKE 並證明 unclonable-IND 保持。剩餘真空與修訂後的方向建議見 `AK21_Close_Reading_and_PQC_Slot_Survey.md`。R2 的「統一刻畫」部分（該報告的 V1）仍存活。
> **日期**：2026-07-12（v2，同日修訂）
> **修訂註記**：v1 把白板公式讀成路線甲（unclonable IPFE）的前奏，過度綁定。經確認，老師白板的意圖是**重新構造一個 UE**——換掉 AK21 配方裡的古典槽，輸出物仍然是 UE 本身，而不是把配方推廣成新原語。本版以此讀法為主軸重寫；與路線甲的關係降為 §4 的對照節。
> **緣起**：老師最初講解本方向時在白板寫下「Unclonable Encryption … otUE + PQC … revisited」，並把 PQC 框起來。本文回答：(1) 白板公式的精確出處與含義；(2) 在「輸出仍是 UE」的前提下，「把 PQC 換成其他的」還有哪些可做的方向。
> **依據**：AK21 原文 §1.1–1.2 精讀；偽金鑰性質調查報告；UE 五方向報告；2026-07 文獻總盤點。

---

## 0. TL;DR

1. **白板寫的就是 AK21（TCC 2021）的核心配方**：`可重用 UE = otUE（一次性不可複製加密，量子核心）+ PQC（後量子古典加密，可替換槽）`。「otUE」是論文 §1.2 自己的記號，「revisited」是論文標題。
2. **關鍵前提事實：AK21 的假設已經最優**（私鑰版 pq-OWF、公鑰版 pq-PKE，兩者都被 UE 蘊含，論文自己指出 "best one can hope for"）。所以「換 PQC」**不可能是降假設**——可換的維度在：構造路徑（公鑰版繞道 FE 的重量）、介面刻畫（偽金鑰性質的一般化）、安全目標（unclonable-IND 版 compiler）、證明技術（一對多偽金鑰）。
3. **最有實質空間的換法**：把公鑰版 PQC 槽裡的「bounded FE + trojan」換成**更輕量的可模糊化加密**（RNCE／somewhere equivocal／equivocal LWE）——這正是偽金鑰報告確認的**五年 open problem**（方向一），也是五方向報告結論中「最深入的未探索領域」（方向一×三×五叢集）的入口。
4. **已關閉、不要進的格子**：降假設（AK21 已最優）、microcrypt／無條件端（BBC26 + BG26b 收割完畢）、多複製安全的「結果」（CGKNY26 上 CRYPTO 2026）。

---

## 1. 白板逐項解碼

| 白板元素 | 對應物 |
|---|---|
| "Unclonable Encrypt[ion] … revisited" | 論文標題《Unclonable Encryption, Revisited》，Ananth-Kaleoglu，TCC 2021（已精讀，PDF 在 `papers/`） |
| "otUE"（雙底線） | 論文 §1.2 的記號：**one-time unclonable encryption**，即 Broadbent-Lord (TQC 2020) 的一次性方案，BB84 共軛編碼、資訊論安全。這是**量子核心，不動** |
| "PQC"（方框 + 勾） | 論文 §1.2 的 ε：**後量子安全的古典加密方案**（私鑰版用 pq-PRF、公鑰版歸到 pq-PKE）。方框 = **可替換槽** |
| "+" | 混合式（hybrid）組合：兩個元件黑盒拼裝 |

### AK21 的混合構造原文（§1.2 "Naive Attempt: A Hybrid Approach"）

```
rUE.Enc(k_ε, m):  取新鮮一次性金鑰 k_otUE
                  輸出 (CT_otUE, CT_ε) = ( otUE.Enc(k_otUE, m),  ε.Enc(k_ε, k_otUE) )
```

量子部分裝訊息（all-or-nothing），古典部分裝一次性金鑰（KEM-DEM 式）。兩條定理：

- **Thm 1**：pq 單向函數 ⇒ 私鑰可重用 UE
- **Thm 2**：pq 公鑰加密 ⇒ 公鑰可重用 UE

（附註：AK21 明言可重用性只對 distinguishing 攻擊成立，cloning 攻擊者仍只拿一份密文——多密文 cloning 是後來 CGKNY26 才解決的獨立問題。）

### 天真混合為什麼不夠：reveal-phase 問題與偽金鑰性質

不可複製性遊戲中，**解密金鑰 k_ε 會在 splitting 之後亮給 Bob 和 Charlie**——語意安全在金鑰公開後就作廢了，歸約卡死。AK21 的解法是要求 ε 額外滿足**偽金鑰性質**（fake-key property）：

> 存在 FakeGen，給定「0 的加密」CT₀ 與訊息 m，輸出偽金鑰 fk，使 (CT_m, k) ≈c (CT₀, fk)。

證明中把 CT_ε 換成加密 0，reveal 時交出內嵌 k_otUE 的偽金鑰，之後才能乾淨地歸約到 otUE。兩個實例化：

- **私鑰**：PRF 加密 `CT = (r, PRF_k(r) ⊕ m ⊕ otp)` 免費滿足——偽金鑰就是 `(k′, otp′ = θ ⊕ PRF_{k′}(r) ⊕ m)`。（論文腳註：somewhere equivocal encryption [HJO+16] 的特例）
- **公鑰**：一般 PKE **沒有**偽金鑰性質，AK21 繞道（單金鑰/有界）**函數加密**：真金鑰 = 恆等函數的功能金鑰、偽金鑰 = 內嵌密文的 trojan 式功能金鑰 [ABSV15]；有界 FE 可從 pq-PKE 構造，所以定理層級假設仍是 pq-PKE。

**所以 PQC 這一格的真實介面規格是：後量子安全 ＋ 可模糊化（偽金鑰）。** 這是白板上看不到、只有讀進證明才看得到的一行字，也是整個「換槽」問題的支點。

---

## 2. 換槽的前提：哪裡沒有空間、哪裡有

**沒有空間的維度——假設強度。** AK21 p.4 自陳假設最優：私鑰 UE ⇒ pq 私鑰加密 ⇒ pq-OWF；公鑰 UE ⇒ pq-PKE。往下換弱到 microcrypt／無條件的路，2026 上半年也被收割完畢（BBC26 不可複製位元；BG26b「不可複製位元 + SKE ⇒ many-time UE」且 tight——那正是白板公式在假設軸上的完成式）。

**有空間的維度（輸出仍是 UE）：**

| # | 換什麼 | 換成什麼 | 得到什麼 |
|---|---|---|---|
| R1 | 公鑰版的構造路徑 | FE+trojan → 輕量可模糊化 PKE | 不經 FE 的公鑰 UE（方向一，5 年 open） |
| R2 | 介面刻畫 | 「PRF 加密」→「任何滿足性質 X 的 pq 加密」 | 一個獨立的 compiler 定理 + X 的精確刻畫（方向三） |
| R3 | 安全目標 | unclonability → unclonable-IND | 強安全版 compiler（需先查 AKLL22 覆蓋範圍） |
| R4 | 證明技術 | 單一偽金鑰 → 一對多偽金鑰 | 多複製安全的直接證明路徑（方向五，技術面仍空） |

---

## 3. 四個換槽方向的詳細評估

### R1（主推）：公鑰偽金鑰不經 FE——「換掉重的、放進輕的」

**現況**（偽金鑰調查報告確認）：2021–2026 沒有任何工作在不用 FE 的情況下把偽金鑰性質推到公鑰；所有公鑰 UE 都走六種替代範式（FE/trojan、iO+陪集態、QROM、UPO、量子金鑰 Haar、QFE）。結構障礙：FE 版的「偽金鑰是功能金鑰、真金鑰是 MSK」，兩者結構不同；且公鑰公開了金鑰結構資訊，與可模糊性天然張力。

**換槽候選**（X = 放進 PQC 槽的新元件）：

- **RNCE（接收者非承諾加密）**：lattice 上有標準構造；量子密文版已存在（arXiv:2311.09487）。偽金鑰性質與 RNCE 的精確蘊含/分離關係本身無人寫過
- **equivocal LWE**（ePrint 2026/1293，AFRICACRYPT 2026 用它做 NC-RFE）：已被命名、被使用的代數假設，是「正式化偽金鑰」現成的地基
- **損失式/雙模式加密**（dual Regev）：在可撤銷密碼學中已扮演同角色（APV23、AHH24 的單射↔損失模式切換），但只會「銷毀」訊息、不會解到指定訊息——需要補上「指定訊息」那一半，這正是研究內容
- **量子繞道**：Bendlin et al.（ASIACRYPT 2011）證明非互動接收者可否認 PKE 需超多項式金鑰——但偽金鑰性質只需可模糊化「Enc(0) 這種特殊密文」（somewhere 版本），比完整接收者可否認**弱**；精確量出這個距離、判斷古典不可能性咬不咬得到，是定義性貢獻的核心。若咬得到，量子金鑰/量子密文是逃生門（Coladangelo-Gunn cUE、AKY 同時 Haar 是「概念相近的部分進展」）

**保底與風險**：定義 pk fake-key + 「pk fake-key ⇒ pk-UE」compiler 定理 + 與 RNCE/somewhere-equivocal/可否認性的蘊含分離圖 = 可發表的定義性貢獻（NC-RFE 上 AFRICACRYPT 是同題型先例）；構造若成立是大貢獻，不成立則收縮到不可能性/分離。風險中-高，但保底清楚。

### R2（骨架/保底）：把 compiler 從構造裡解耦出來

AK21 的 compiler 隱含在構造裡、與 PRF 實例耦合。做成獨立陳述：

> **Compiler 定理**：otUE + 任何滿足〔偽金鑰性質〕的 pq 加密 ⇒ 可重用 UE（私鑰/公鑰兩版）。

然後刻畫〔偽金鑰性質〕的最小充分條件，證明它與 somewhere equivocal encryption、RNCE 的精確關係（偽金鑰↔NCE↔可否認的三角，方向三），並給出多重實例化（PRF 版 = AK21 原例；equivocal-LWE 版；RNCE 版）。**這就是把白板的「+」號變成論文的主定理**。工作量可控、定義性貢獻保底、且是 R1 的必經前置——建議與 R1 打包成同一條研究線的兩章。

### R3（檢查點）：unclonable-IND 版的 compiler

AK21 的定理只對 unclonability（搜尋型安全）陳述。強概念 unclonable-IND 的一次性方案 QROM 已有（AKLL22 陪集態、AKL23 簡化成 BB84），問題是：**同一個偽金鑰混合論證能否把 one-time unclonable-IND 升到可重用/公鑰？** 動手前先查 AKLL22/AKL23 是否已寫掉（它們有公鑰段落）；若無人寫，這是一個引理尺寸的補充結果，適合當 R2 論文的一節。標準模型+古典金鑰+IND 是領域大魔王，只作敘事、不正面進攻。

### R4（延伸）：一對多偽金鑰

多複製安全的**結果**被 CGKNY26 拿走（CRYPTO 2026，其 compiler 刻意繞開偽金鑰路徑），但「一對多偽金鑰給出更直接證明」的**技術**仍空（方向五）。私鑰情形可能平凡（獨立採樣 k′ᵢ + hybrid），公鑰情形 = R1 的多查詢版。定位：R1/R2 做完後的自然延伸章，或與 CGKNY26 的比較節。

### 已關閉的格子（避免誤入）

| 格子 | 被誰關閉 |
|---|---|
| 降假設（OWF/PKE 以下） | AK21 自身（假設最優性論證） |
| microcrypt／無條件 UE、boosting | BBC26、BG26a/b、BC26（半年七篇，survey §2.2） |
| 多複製安全的結果 | CGKNY26（CRYPTO 2026） |
| QROM 強安全一次性方案 | AKLL22、AKL23 |

---

## 4. 與路線甲（unclonable IPFE）的關係：兩條不同的線，共用一個零件

**輸出物不同，是兩個研究目標**：

| | 白板線（本文） | 路線甲（QFE 線） |
|---|---|---|
| 源頭 | 老師最初白板（AK21 配方） | 報告完 MM24 後老師的「升級階梯」討論 |
| 輸出物 | **一個新的 UE 構造**（更輕的公鑰路徑／更一般的 compiler／更強安全概念） | **一個新原語**：unclonable IPFE |
| 對應 README 路徑 | 路徑 2 + 3 | 路徑 1 |
| 地圖文件 | 偽金鑰報告 + 五方向報告（方向一/三/五） | Deep_Dive_Route_A + QFE survey |

**為什麼白板公式不能直接通到路線甲**：若把 IPFE 放進 PQC 槽而輸出仍要是 UE，功能金鑰 sk_y 會從古典部分解出 ⟨k_otUE, y⟩——洩漏一次性金鑰的線性函數，UE 安全性反而被 FE 的功能性破壞；要讓 IPFE 的功能性有意義，就必須把目標原語從 UE 換成 FE、並把訊息載荷搬到古典側（`IPFE.Enc(x+r) ⊗ UE.Enc(r)`）——那一步就離開白板線、進入路線甲了。**這個「一放進 FE 就被迫改目標」的觀察，正說明兩條線是真正不同的題目。**

**共用零件**：偽金鑰／可模糊化介面。R1/R2 把偽金鑰性質做成獨立原語之後，路線甲的公鑰化與路線丁（IPFE 的 fake function key）可以直接消費它。所以先做哪條線，另一條都不白費——但這是零件層的共享，不是題目層的等同。

---

## 5. 給老師的討論包

### 第一個要問的問題（定錨題）

> 「老師，白板上『把 PQC 換掉』的意思，我想確認輸出物：是 (a) **重新構造一個 UE**——換掉 AK21 古典槽、讓公鑰版不用繞 FE 或讓 compiler 更一般；還是 (b) 把配方推廣到別的原語（例如 unclonable IPFE）？兩條我都盤點過了，(a) 對應偽金鑰那個五年沒人做的 open problem，(b) 對應之前 MM24 那條升級階梯。」

### 若確認是 (a)（白板線），提案題目

> **「公鑰偽金鑰性質：正式定義、與非承諾/可否認加密的關係、以及不經函數加密的公鑰不可複製加密」**
>
> 執行輪廓：R2（compiler 解耦 + 三角關係，定義性保底）→ R1（構造嘗試：equivocal LWE / RNCE / 量子繞道；或反向的不可能性）→ R3/R4 作延伸章。
> 動機句：AK21 的假設已最優，但公鑰版的構造繞道 FE——「假設輕、工具重」的落差就是本題的存在理由；所有後續公鑰 UE（iO、QROM、UPO、量子金鑰）工具更重，五年沒人回頭補這個洞。

### 若確認是 (b)，沿用路線甲

Deep_Dive_Route_A 的 12–16 週計畫與三個既有問題照舊，本文 §4 的對照表可當開場圖。

### 誠實的風險對照

- 白板線：新穎性窗口大（零人做）、競速壓力低；風險在**構造是否存在**（Bendlin 型古典不可能性壓頂，可能收縮成定義+分離的論文）。
- 路線甲：構造路徑已 sketch（P1）、保底較厚；風險在坑 1（D2）與原作者團隊競速。
- 兩條都可答辯；差別是白板線賭「定義+構造」、路線甲賭「構造+證明細節」。

---

## 6. 本文引用

- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.（Thm 1/2、hybrid approach、fake-key property、trojan 公鑰構造、假設最優性）
- **[BL20]** Broadbent, Lord. *Uncloneable Encryption*. TQC 2020.（otUE 本體）
- **[HJO+16]** Hemenway et al. somewhere equivocal encryption.（偽金鑰的古典脈絡）
- **[Bendlin+11]** Bendlin, Nielsen, Nordholt, Orlandi. ASIACRYPT 2011.（非互動接收者可否認 PKE 的不可能性——R1 必須繞過的牆）
- **[NC-RFE]** Sarkar. ePrint 2026/1293.（equivocal LWE；定義性題型的可發表先例）
- **[CG24]** Coladangelo, Gunn. STOC 2024.（cUE——概念相近的部分進展）
- **[AKLL22]** Ananth, Kaleoglu, Li, Liu, Zhandry. CRYPTO 2022.（QROM unclonable-IND；R3 的查證對象）
- **[CGKNY26]** *Multi-Copy Security in Unclonable Cryptography*. CRYPTO 2026.（R4 的對照組）
- 其餘見 `public_key_encryption_with_fake-key_property_report.md`、`Research_Directions_of_Unclonable_Encryption.md` 與 `Survey_Recent_Developments_and_New_Routes_2026-07.md`。
