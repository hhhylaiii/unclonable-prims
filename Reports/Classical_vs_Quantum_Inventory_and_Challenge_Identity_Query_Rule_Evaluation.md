# 老師的兩個提問：系統內的古典／量子物件盤點；「不得查詢 id\*」規則與 non-adaptive 選項的評估

> **⚠ 2026-09-28 狀態**：§2.4 的更正已併入[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（延後引理、凍結與表 K 皆已刪除，見[延後引理報告](./Deferred_Query_Lemma_Unnecessary_Who_Answers_Queries_Two_Reveals.md)）。§2.2 的「分裂後不設限」提案**尚未採用**：v3.1 的查詢階段 II／III 仍禁查 id\*；[KT18 報告](./KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md) §3.5 第 2 點指出 Construction 2 的證明需要這條禁令，若要解除需以 PRF 去隨機化 KeyGen（該報告 Q5）。§3 的「v4」修改建議：證明部分（刪 Lemma 1；Hyb₁、Claim 1、Claim 2 以 KeyGen(m̃sk, ·) 回答分裂後查詢；修訂五改寫）已進 v3.1，定義部分（查詢階段 II 不設限、揭露移到分裂當下、修訂六）未採用。**09-28 追記**：老師再次提出「揭露後禁查 id\* 怪怪的，或用 non-adaptive 就好」；完整評估（時序與蘊含引理、KT18 路線的去隨機化代價、non-adaptive 的三種讀法）見[揭露後禁查 id\* 評估](./Post_Reveal_Challenge_Identity_Query_Ban_vs_Non_Adaptive_Evaluation.md)。

> **⚠ 2026-09-27 更正（重讀全部資料時發現）**：本文 §1.3 與 §2.3 把 GKK25 的 SXDH 構造（Thm 12）與 DDH 版（Thm 3 取 X = DDH）當成可用零件——**這是錯的**。主定理的前提要求 RNC-IB-KEM 對 **QPT** 對手安全，而 cloning game 的對手本來就是量子的：古典槽若基於 SXDH／DDH，A 可在分裂前用 Shor 解出 session key、進而得到 k 與 m，把古典的 m 複製給 B、C 直接獲勝——這是直接攻擊，不是歸約失敗（見 `Meeting_2026-08_RNC_IBE_Interface_Check_and_Open_Decisions.md` §1.4）。**目前唯一合法的實例仍是 GKK25 Thm 3 取 X = LWE（poly-ID）**。§2.3 表中「adaptive 免費」的論點不受影響（LWE 版 Thm 3 本身就是 adaptive 安全）。

> **日期**：2026-09-13
> **議題來源**：老師在討論主定理稿時提出的兩點——(1) 底層 otUE 是量子加密系統，整個系統裡哪些是量子態、哪些是古典態；(2) 安全性實驗中「查詢階段不能查詢 id\*」怪怪的，或者改用 non-adaptive 就好。
> **本文結論一句話**：(1) 整個系統只有兩樣東西是量子的——otUE 的密文 ρ 與對手的內部暫存器；其餘（含所有金鑰、IBKEM 的一切、四個模擬器）全部古典，而且**必須**古典，否則 OTP 墊層、分裂後的一致性、以及「同一把金鑰交給兩人」都做不到。(2) 「不得查詢 id\*」只在**分裂前**是必要的（否則對手解密後把 m 複製給兩人，平凡獲勝）；**分裂後**的禁令沒有必要，建議解除（含 id\* 在內任意查詢）。adaptive 不必退成 selective：GKK25 的兩個構造本來就是 adaptive 安全，selective 只會白白弱化定理，而且並不消除那條規則。
> **附帶的重大更正**：重新檢視分裂後查詢的處理後發現，**D-W1（分裂後查詢 vs 帶狀態模擬器）從一開始就不是問題**：挑戰之後的所有金鑰都由 KeyGen(m̃sk, ·) 供應，而 Claim 2 的 B̃ 在 otUE 遊戲 Phase 2 一開始就拿到 k、算得出 m̃sk，因此 B 分裂後的**任何**查詢它都答得出來。昨天報告的延後引理（Lemma 1）是對的，但**不需要**；凍結表 K 更不需要。詳見 §2.4。

---

## 0. TL;DR

| 提問 | 結論 | 對文件的影響 |
|---|---|---|
| (1) 哪些是量子、哪些是古典 | 量子：ρ（otUE 密文）、對手的暫存器與分裂映射。古典：m、所有金鑰（msk、sk_id、otUE 的 k）、ct₁、c₂、所有查詢與答案、揭露物、Sim₁–Sim₄ 及其輸出、cls。 | 在定義章加一節「古典／量子物件表」與三條設計後果（otUE 必須古典金鑰、只加密古典訊息、查詢為古典查詢）。 |
| (2a) 分裂前不得查詢 id\* | **必要**：否則 A 解密 ρ 得 m，把古典的 m 複製給 B、C，機率 1 獲勝。這正是不可複製性的內容——挑戰身分的金鑰只能在分裂後出現——不是人為的 admissibility 條件。 | 保留；在定義後加一句說明理由。 |
| (2b) 分裂後不得查詢 id\* | **不必要、也確實怪**：v3 裡 B、C 手上已有 sk_{id\*}，卻不准再查 id\*。建議分裂後**不設限**（含 id\*、含重複）。揭露階段可保留（同一把 sk_{id\*} 交給兩人，老師的要求）並移到分裂當下。 | Definition 2/3 改寫；查詢階段 II／III 合併為一段不設限的查詢階段。 |
| (2c) 改用 non-adaptive（selective-ID）？ | **不建議**。GKK25 Def 8 是 adaptive，其 SXDH（Thm 12）與批次加密（Thm 14）構造都 adaptive 安全，歸約只是轉送，adaptive 免費；selective 版定理較弱，且「不得查詢 id\*」的規則在 selective 版一樣存在，只是變成靜態的。selective 留作備案（若日後乙案的 LWE 零件只有 selective 安全）。 | 主定義維持 adaptive；備註 selective 變體。 |
| 更正 | D-W1 不存在。後挑戰金鑰 = KeyGen(m̃sk, ·)；B̃ 一進 Phase 2 就有 k。延後引理與凍結表都不需要。 | 刪 Lemma 1（延後）；Hyb₁、Claim 1、Claim 2 直接回答分裂後所有查詢；修訂五改寫。 |

---

## 1. 提問 (1)：系統內的古典／量子物件盤點

### 1.1 三個層級的物件表

**目標原語 UIBE（Definition 1–3）**

| 物件 | 古典／量子 | 持有者／產生者 | 可複製？ | 備註 |
|---|---|---|---|---|
| mpk、msk | 古典 | 挑戰者（= IBKEM 的） | 可 | msk 在 v3 起不再交出 |
| sk_id（含 sk_{id\*}） | 古典 | 挑戰者以 KeyGen 產生 | 可 | **同一把 sk_{id\*} 能交給 B 與 C 兩人，只因它是古典的**；若金鑰是量子態，no-cloning 就禁止「同一把給兩人」這個敘述 |
| 訊息 m／位元 b | 古典 | 挑戰者抽樣（搜尋型：均勻 n 位元；不可區分型：A 選 m₀, m₁） | 可 | QECM = quantum encryption of **classical** messages |
| 密文 ct = (ct₁, c₂, ρ) | ct₁ 古典、c₂ 古典、**ρ 量子** | Enc（一個量子演算法：跑古典的 Encap、OTP，再製備 ρ） | ct₁、c₂ 可；ρ 不可 | 整個方案唯一的量子密文成分；不可複製性全在 ρ |
| Dec | 量子演算法（對 ρ 做測量） | 持 sk_id 者 | — | 測量具破壞性：一份 ρ 只能解一次；正確性以壓倒性機率 |
| 金鑰查詢 id 與答案 sk_id | 古典 | A、B、C 發出；挑戰者回答 | 可 | **古典查詢**（不允許疊加態查詢，定義稿 Remark 9）；GKK25 Def 8 也只有古典查詢，歸約 R 才能原樣轉送 |
| A 的內部狀態、分裂映射 Φ | 量子；Φ 為 CPTP | A | — | B、C 暫存器可含古典部分（A 想複製多少古典資料進兩邊都行） |
| B、C 的輸出 m_B、m_C | 古典 | B、C | — | 勝利條件是古典述詞 |

**建構元件**

| 物件 | 古典／量子 | 備註 |
|---|---|---|
| otUE 金鑰 k ∈ K ⊆ {0,1}^κ | **古典** | AK21 Def 7 的 QECM 語法明定「古典金鑰」。BL20：k = (x, θ) ∈ {0,1}ⁿ × {0,1}ⁿ，κ = 2n，均勻；AS26：k = (x, z) ∈ {0,1}ⁿ × {0,1}ⁿ 且 x₁ = 1，κ = 2n，**不均勻**（其 Definition 2.3 明寫「outputs a classical secret key」） |
| otUE 密文 ρ | 量子 | BL20：H^θ\|x ⊕ m⟩，n 個 qubit；AS26：n 個 qubit（以 k 決定的 Pauli 本徵基旋轉計算基向量），訊息 1 位元 |
| otUE.Dec | 測量 | BL20 在基 θ 測量後 XOR x（完美正確）；AS26 依 k 選觀測量測量 |
| IBKEM 的一切（mpk、msk、sk_id、ct₁、k̄、Encap、Decap） | **全部古典** | 一個古典原語，但要對 QPT 對手安全（後量子）——修訂三的來源 |
| Sim₁–Sim₄ 與其輸出（mpk、st₁–st₃、sk_id、ct\*、msk） | **全部古典**、PPT | GKK25 是古典論文；模擬器的隨機帶 r 也是古典字串 |

**證明中的物件**

| 物件 | 古典／量子 | 備註 |
|---|---|---|
| 歸約 R（Claim 1） | QPT 機器 | 內部跑量子對手與量子密文，對外與**古典**挑戰者只交換古典訊息——這就是 Def 8 必須對 QPT 對手陳述的原因 |
| (Ã, B̃, C̃)（Claim 2） | QPT | Ã 的輸出 ρ_B ⊗ \|cls⟩⟨cls\|、ρ_C ⊗ \|cls⟩⟨cls\|：ρ_B、ρ_C 量子，cls 古典 |
| cls = (st₃, r, r_sk, c₂, id\*) | 古典 | 可自由複製兩份——證明裡「一切古典資料都能複製到分裂線兩側」的形式化 |
| otUE 遊戲揭露的 k | 古典 | 兩側都拿到同一個 k ⇒ 同一個 k̄ = c₂ ⊕ k ⇒ 同一把 m̃sk、同一把 s̃k_{id\*} |
| no-cloning 用在哪 | 只在 otUE 的假設裡 | 我們的證明從頭到尾不直接論證任何量子態，只做分布比對與包裝 |

### 1.2 三條設計後果（建議寫進定義章）

1. **otUE 必須是古典金鑰的 QECM。** 構造裡 k 被 XOR（c₂ = k̄ ⊕ k）、被複製進 cls 的推導鏈、被同一份揭露給兩側——三件事都要求 k 是古典字串。文獻中另有「量子解密金鑰」的不可複製加密變體（例如 MM24 的 Definition 18，unclonable-indistinguishable encryption with quantum decryption keys），**不能**放進這個槽位。AK21 的 QECM、BL20、AS26 都是古典金鑰，剛好合用。
2. **只加密古典訊息。** QECM 的訊息是古典字串；量子訊息的不可複製加密是另一個原語，不在本文範圍。
3. **金鑰查詢是古典查詢。** 對手是量子的，但它送給挑戰者的是古典字串 id。若允許疊加態查詢，歸約 R 就無法把查詢轉送給 GKK25 的古典挑戰者，需要一個對量子查詢安全的 RNC-IB-KEM（目前沒有）。這是定義稿 Remark 9 的理由，建議明寫。

一個整體的講法，可以放在定義章開頭：**整個系統是「古典的 IBE 骨架 ＋ 一顆量子的 DEM」**。所有身分、金鑰、查詢、揭露都是古典的，跟一般 IBE 一樣；唯一的量子成分是密文裡的 ρ；不可複製性完全由 ρ 承擔，並經由主定理無損傳遞。古典部分被複製了也沒用（拿不到 k̄ 就拿不到 k），這件事在證明裡就是 cls 可以複製兩份。

### 1.3 實例化時的具體形狀

| | BL20（搜尋型，Thm 1） | AS26（不可區分型，Thm 2） |
|---|---|---|
| otUE 金鑰 | (x, θ)，2n 位元，均勻 | (x, z)，x₁ = 1，2n 位元，不均勻 |
| otUE 密文 | n 個 qubit | n 個 qubit |
| 訊息 | n 位元 | 1 位元 |
| session key 長度 ℓ | 2n | 2n |
| UIBE 密文 | (ct₁, c₂ ∈ {0,1}^{2n}, ρ ∈ (ℂ²)^{⊗n}) | 同左 |

GKK25 的 SXDH 構造把 session key 長度 ℓ 當參數（其 §5.1「ℓ := ℓ(λ) be a polynomial in λ」，seskey ∈ {0,1}^ℓ），取 ℓ = 2n 即可。**AS26 的金鑰不均勻**（x₁ = 1）這件事，正好是修訂二選 OTP 墊層而非 key-transparent 的一個具體例證：若走出路 1，AS26 就得另外處理。

---

## 2. 提問 (2)：「查詢階段不能查詢 id\*」與 non-adaptive

### 2.1 分裂前的禁令是必要的，而且就是不可複製性的內容

若 A 在分裂之前能拿到 sk_{id\*}（不論是在查詢階段 I 查到，或在收到 ρ 之後、分裂之前查到），它就對 ρ 執行 Dec 得到 m（不可區分型：得到 b），然後把**古典的** m 複製進 B、C 兩個暫存器——兩人不需要任何量子資訊就一起答對，獲勝機率 1（搜尋型）或 1（不可區分型）。任何允許這件事的定義都不可能被滿足。

所以這條規則不是 IBE 遊戲照抄過來的 admissibility 條件，而是 cloning game 的核心設計：**挑戰身分的金鑰只能在分裂之後出現**。這與 AK21 一次性遊戲裡「金鑰 k 在分裂後才揭露」是同一件事；IBE 多出來的只是「其他身分的金鑰隨時可查」——模型其他使用者的共謀。建議在 Definition 2 後面加一句這個理由，讓讀者知道邊界畫在「分裂」而非「挑戰」。

### 2.2 分裂後的禁令沒有必要——建議解除

v3 的查詢階段 II／III 都寫了「不得查詢 id\*」，而 B、C 在揭露後手上就有 sk_{id\*}。這條禁令的唯一作用是讓「揭露階段」在形式上有別於「查詢」，安全性上什麼都沒守住：不可複製性的主張本來就是「即使兩人都拿到 sk_{id\*} 也不能同時解密」，多給幾把獨立的 sk_{id\*}（KeyGen 隨機化時）不會改變這件事。證明也完全撐得住（§2.4）：Hyb₁、Claim 1 的 R、Claim 2 的 B̃／C̃ 都能以 KeyGen(m̃sk, id\*) 回答 id\* 的查詢。

**建議的定義形狀**（Definition 2）：

- 建置階段、查詢階段 I（A 任意查詢）、挑戰階段（A 輸出**未在查詢階段 I 查過**的 id\*；挑戰者抽 m，ρ ← Enc(mpk, id\*, m) 給 A）——與現行相同。
- 分裂階段：A 施加 CPTP 得到 B、C 暫存器交給 B、C；**同時**挑戰者計算 sk_{id\*} ← KeyGen(msk, id\*)，把同一把交給 B 與 C。此後兩人不得通訊。
- 查詢階段 II：B、C 各自獨立地對 KeyGen(msk, ·) 做多項式次古典的、適應性的查詢，**不設任何限制**（含 id\*、含重複）。
- 猜測階段。

要點：「不得查詢 id\*」只出現在查詢階段 I；揭露物 sk_{id\*}（老師的要求）保留，並移到分裂當下——既然分裂後可以查 id\*，把揭露放在任何更晚的時點都只是形式；同一把交給兩人這件事保留「PKG 對每個身分發一把金鑰」的語義，且在證明裡零成本（r_sk 預抽）。若想再精簡，也可以把揭露那一句拿掉、只留不設限的查詢（B、C 自己查 id\*）——差別只在 KeyGen 隨機化時兩人拿到的是同一把還是各自新鮮的一把，證明相同；本文建議保留揭露句。

### 2.3 adaptive vs selective（non-adaptive）

| | adaptive（現行） | selective-ID（A 開場就承諾 id\*） | non-adaptive 查詢（所有查詢一次交出） |
|---|---|---|---|
| 「不得查詢 id\*」規則 | 仍有（動態檢查） | 仍有（靜態檢查）——**規則沒有消失** | 仍有 |
| 零件支援 | GKK25 Def 8 即 adaptive；Thm 12（SXDH）、Thm 14（DDH／LWE）皆 adaptive 安全 | Def 7；所有構造都支援（更弱） | 無此定義，需自己證 |
| 歸約 R 的改動 | 無 | 轉送 A 的承諾；Sim₁ 多吃 id\* | 需重寫 |
| 定理強度 | 最強 | 弱；selective → adaptive 的 complexity leveraging 損失 2^d，指數身分空間下不可用 | 更弱 |
| 判斷 | **維持** | 備案：只在未來 pq 零件（乙案的 LWE 指數身分空間 RNC-IBE）只有 selective 安全時使用 | 不採 |

selective 並不回應老師覺得「怪」的地方——它只是把同一條規則從動態改成靜態；真正怪的是分裂後的禁令，解法是 §2.2。

### 2.4 附帶的重大更正：D-W1 從來不是問題，延後引理不需要

重新檢視 v3 的 Claim 2 時序後，發現我先前的一個混淆：**UIBE 遊戲的揭露**（B 拿到 sk_{id\*}）與 **otUE 遊戲的揭露**（B̃ 拿到 k）是兩個不同的事件。B̃ 的整段計算都發生在 otUE 遊戲的 Phase 2，而 Phase 2 的第一件事就是把 k 交給 B̃。所以 B̃ 在開始模擬 B 的分裂後生涯之前，就已經有 k，就已經算得出 k̄ = c₂ ⊕ k、m̃sk = Sim₄(st₃, k̄; r)、s̃k_{id\*}——**B 分裂後任何時點的任何查詢，B̃ 都能用 KeyGen(m̃sk, ·) 回答**，包括 UIBE 揭露之前的查詢。「wrapper 在揭露前算不出 m̃sk」這句話是錯的。

另一半：Claim 1 的 R 在 IBKEM 遊戲的挑戰階段就收到 m̄sk，分裂發生在挑戰之後，所以分裂後任何查詢 R 都能用 KeyGen(m̄sk, ·) 回答——真實世界它是真 msk（＝Hyb₀ 的預言機），模擬世界它是 m̃sk（＝Hyb₁）。

於是整個 D-W1 的前提——「分裂後的查詢必須靠 Sim₂ 回答，而 Sim₂ 帶狀態、兩側會分岔」——是錯的。在 GKK25 Def 8 的模擬世界裡，Sim₂ 只負責**挑戰前**的查詢；挑戰之後對手手上有 m̃sk，任何金鑰都是它自己用真正的 KeyGen 算出來的，RNC 安全性涵蓋對手拿 (m̃sk, ct\*, k̄) 做的一切計算。Hyb₁ 只要照這個形狀定義——挑戰前用 Sim₂（單一挑戰者依序呼叫，帶狀態無妨），挑戰後一律 KeyGen(m̃sk, ·)——兩個歸約就都是逐位元完美模擬。凍結、表 K、字典序、memoized、延後引理，全部不需要。

昨天報告（`Replacing_Frozen_Key_Table_in_Hyb1_...`）的主結論「表 K 整個刪掉、身分空間任意」不變，但機制從「延後引理」換成「根本沒有需要延後的東西」；該報告文首已加更正註記。延後引理本身仍然成立，可以留作一則備註（說明揭露前後的查詢沒有差別），但不進證明。

**這個更正對「同一把 sk_{id\*} 給兩人」的影響**：仍需 r_sk 預抽並附掛 cls（兩側算出同一把）；對「不設限的查詢」：兩側各自新鮮抽樣即為正確分布（標準預言機）。r、r_sk、c₂ 的預抽是證明裡唯一真正需要跨側協調的地方。

---

## 3. 建議的文件修改（v4）

**Definition 2／3**：如 §2.2；查詢階段 I 保留「不得查詢 id\*」並加一句理由；分裂階段同時交出同一把 sk_{id\*}；查詢階段 II 不設限。刪 Lemma 1（延後）或降為備註。

**Hyb₁**：挑戰步驟不變（Sim₃、k̄、r、r_sk、m̃sk、s̃k_{id\*}、OTP）；分裂時交出 s̃k_{id\*}；查詢階段 II 的每個查詢（含 id\*）以新鮮隨機性計算 KeyGen(m̃sk, id) 回答。

**Claim 1**：R 分裂後以 KeyGen(m̄sk, ·) 回答所有查詢；「第 0 步」刪除。

**Claim 2**：B̃ 收到 k 後先算 m̃sk、s̃k_{id\*}，再跑 B；把 s̃k_{id\*} 隨暫存器一起交給 B；所有查詢以 KeyGen(m̃sk, ·) 回答。時序核對一段可以刪掉（沒有先後問題了）。

**第 4 節**：修訂五改寫為「凍結與表 K 的刪除：後挑戰金鑰由 KeyGen(m̃sk, ·) 供應」，並記錄 D-W1 誤判的原因；新增修訂六「分裂後查詢不設限、揭露移到分裂當下」；定義章加古典／量子物件表與三條設計後果（§1.2）。

**路線圖／README**：D-W1 結案（不是解掉，是不存在）；定義稿 §3「分裂線切開模擬器序列」一節的 Sim₂ 列要改寫——Sim₂ 從不在線的右邊被呼叫。

### 可直接貼進 Overleaf 的片段（沿用主定理稿的巨集）

```latex
% ---------- Definition 2 的實驗框（取代 v3 版） ----------
\begin{expbox}{$\Expt^{\mathrm{cloning}}_{\UIBE,(\cA,\cB,\cC)}(1^\lambda, 1^d)$}
\begin{itemize}[itemsep=2pt,topsep=1pt]
  \item \textbf{建置階段}：挑戰者執行 $(\mpk,\msk) \leftarrow \Setup(1^\lambda,1^d)$，將 $\mpk$ 送給 $\cA$。
  \item \textbf{查詢階段 I}：$\cA$ 可對預言機 $\KeyGen(\msk,\cdot)$ 做多項式次（古典的、適應性的）金鑰查詢。
  \item \textbf{挑戰階段}：$\cA$ 輸出一個查詢階段 I 中未曾查詢過的 $\id^*$。
        挑戰者均勻抽樣 $m \leftarrow \{0,1\}^n$，計算 $\rho \leftarrow \Enc(\mpk, \id^*, m)$ 並送給 $\cA$。
  \item \textbf{分裂階段}：$\cA$ 對其持有的量子態施加 CPTP 映射
        $\Phi : \Dm(\cH_A) \to \Dm(\cH_B \otimes \cH_C)$，將 $B$ 暫存器交給 $\cB$、$C$ 暫存器交給 $\cC$。
        同時挑戰者計算 $\sk_{\id^*} \leftarrow \KeyGen(\msk,\id^*)$，將\textbf{同一把} $\sk_{\id^*}$ 交給 $\cB$ 與 $\cC$。
        \textbf{此後 $\cB$ 與 $\cC$ 不得通訊。}
  \item \textbf{查詢階段 II}：$\cB$ 與 $\cC$ \emph{各自獨立地}對預言機 $\KeyGen(\msk,\cdot)$
        做多項式次（古典的、適應性的）金鑰查詢，\textbf{不設任何限制}（可查詢 $\id^*$、可重複）。
  \item \textbf{猜測階段}：$\cB$ 輸出 $m_B$，$\cC$ 輸出 $m_C$。
\end{itemize}
實驗輸出 $1$（對手獲勝）$\iff m_B = m_C = m$。
\end{expbox}

\begin{remark}[為何只有查詢階段 I 禁查 $\id^*$]
若 $\cA$ 在分裂之前取得 $\sk_{\id^*}$，它可解密 $\rho$ 得到古典的 $m$，再把 $m$ 複製進兩個暫存器，
以機率 $1$ 獲勝；任何允許這件事的定義都無法被滿足。因此邊界畫在\textbf{分裂}：挑戰身分的金鑰只能在
分裂之後出現——這與 AK21 一次性遊戲中「$k$ 在分裂後才揭露」是同一件事。分裂之後則不需要任何限制：
不可複製性的主張正是「即使兩人都持有 $\sk_{\id^*}$（甚至多把），也不能同時解密」。
\end{remark}

% ---------- Hyb_1 框中分裂與查詢階段 II 兩步 ----------
  \item \textbf{分裂}：同 $\Hyb_0$；同時把 $\tsk_{\id^*} := \KeyGen(\tmsk,\id^*;\,\rsk)$ 交給 $\cB$、$\cC$。
  \item \textbf{查詢階段 II}：$\cB$、$\cC$ 的每個查詢 $\id$（含 $\id^*$）以新鮮隨機性計算
        $\KeyGen(\tmsk,\id)$ 回答。

% ---------- Claim 2 中 B̃ 的步驟 ----------
$\tB$（$\otUE$ 遊戲 Phase 2，收到揭露的 $k$）：
\begin{enumerate}[itemsep=1pt]
  \item 本地還原 $\bk := c_2 \oplus k$；本地計算 $\tmsk := \Sim_4(\stt_3,\bk;\,r)$ 與
        $\tsk_{\id^*} := \KeyGen(\tmsk,\id^*;\,\rsk)$。
  \item 內部執行 $\cB$：餵入 $\rho_B$ 與 $\tsk_{\id^*}$；$\cB$ 的每個查詢 $\id$（含 $\id^*$）
        以新鮮隨機性計算 $\KeyGen(\tmsk,\id)$ 回答。
  \item 輸出 $\cB$ 的 $m_B$。
\end{enumerate}
```

---

## 參考

- AK21 Def 7（QECM：古典金鑰、古典訊息、量子密文）、Def 11/12。
- BL20：conjugate encryption，k = (x, θ)，H^θ|x ⊕ m⟩。
- AS26：Ananth, Sahai. *Unconditional Unclonable Encryption*. ePrint 2026/1511——Definition 2.3「Gen(1ⁿ) outputs a classical secret key k」；k = (x, z)，x₁ = 1；n-qubit 密文；1-bit 訊息；Definition 2.4 把同一把 k 揭露給 B、C。
- GKK25 Def 6/7/8（adaptive／selective；Def 8 挑戰後無查詢階段）、§5.1（ℓ 為參數；Sim₂ 無狀態）、Thm 12、Thm 14。
- MM24 Definition 18（量子解密金鑰的變體，不合用）。
- 本儲存庫：主定理稿 [`Paper drafts/Proof_of_Main_Theory_in_UIBE.pdf`](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（本文依據的是 2026-09-13 第三版 `Proof_of_Main_Theory_in_UIBE_2026-09-13_v3.tex`，其 LaTeX 原始檔未收入本儲存庫；現存 PDF 已更新為 v3.1）；`Reports/Replacing_Frozen_Key_Table_in_Hyb1_Deferred_Query_Lemma_and_Alternatives.md`（文首已加更正註記）；定義稿 Remark 9（古典查詢）。
