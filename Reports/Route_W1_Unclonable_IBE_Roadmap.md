# 路線 W1 技術路線圖：Unclonable IBE

> **日期**：2026-08-01
> **狀態**：**現行主軸**。碩論方向已定案為 W1（不可複製身分基加密），本文取代 Meeting 報告 §4 的「三個骨架選項」，成為主軸的技術總圖。
> **前身**：本路線在 [AK21 精讀報告](./AK21_Close_Reading_and_PQC_Slot_Survey.md) §4+ 首次提出（編號 W1），在 [Meeting 討論報告](./Meeting_2026-07-15_Two_Routes_Discussion.md) §3 以「路線二」與 Unclonable IPFE 並列。現在並列狀態結束。
> **一句話**：把 AK21／HKNY24 的 KEM-DEM 配方裡的古典槽換成 RNC-IBE，得到文獻中不存在的 unclonable IBE；證明套 HKNY24 App E 模板，實質工作在定義設計與後量子零件。

---

## 0. TL;DR（六點）

1. **主軸定案**：W1（Unclonable IBE）。路線甲（Unclonable IPFE）降為備援／延伸章，其 Deep Dive 與 Clifford no-go 副產品保留。
2. **構造已定形**：`ct = (RNCIBE.Enc(mpk, id, k), otUE.Enc(k, m))`，KEM-DEM 骨架不變，古典槽吃 all-or-nothing 加密的可模糊化版本（槽位規則見精讀報告 §4+.1）。
3. **精讀 RNC-IBE 的第一個好消息**：其安全定義（Def 6）在挑戰階段直接把 **msk 整把交給對手**——不只是 sk_{id*}。這比我們原先預期的介面**更強**，意味著 W1 可以做出「即使 B、C 分裂後兩人都拿到 master secret key 仍不可複製」的版本（§3.3）。
4. **後量子零件的實況已釘死**（§4）：主構造（Thm 1，SXDH 雙線性）不可用；**唯一現成的後量子選項是 Thm 3 取 X = LWE 的 relaxed 版**——adaptive 安全、mpk compact、但身分空間僅多項式大小 T，且密文尺寸 poly(T, |m|, λ)。iO 版（Thm 5/6）不列入考慮。
5. **pq 策略兩案並陳、待拍板**（§4.3）：甲案「poly-ID 誠實起步」／乙案「自造 LWE selective RNC-IBE 當技術核心」。本文列出兩案的判準與一週試探設計，不預先選邊。
6. **兩項對既有報告的更正**（§6）：HMNY21 的正確發表處是 **ASIACRYPT 2021（非 TCC 2021）**；「RNC-IBE 零量子應用」的說法需精確化——RNC-ABE 這個概念**本來就是為了量子應用（ABE with certified deletion）而生的**，競速風險比原評估高。

---

## 1. 目標原語：Unclonable IBE 的語義

### 1.1 功能與新性質

- **功能**（與古典 IBE 相同）：寄件人用收件人的身分字串 id 直接加密；收件人向金鑰機構（PKG）領取 sk_id 後解密。
- **新性質**（不可複製性）：對身分 id\* 的密文含量子態。持有該密文的對手 A 把它分裂成兩份交給 B、C，即使
  - 兩人事後都拿到 sk_{id\*}（甚至整把 msk，見 §3.3），且
  - 全程可查詢任意多個**非挑戰身分**的金鑰，

  兩人仍無法**同時**解出訊息。單人可以解（那是功能性），兩人同贏被 no-cloning 擋住。
- **訊息與身分皆為古典**，「量子」只在密文——與 BL20／AK21 一脈相承。
- **定位**：HKNY24 App E 佔掉 PKE 這格；IBE 是 PKE 之後最基本的存取結構，這是把不可複製性帶進「有金鑰管理結構」的世界的第一步。

### 1.2 定義設計的自由度（本路線最需要自己決定的部分）

證明模板是現成的，**定義不是**。以下五個維度各有自然變體，論文必須選定並論證：

| # | 維度 | 選項 | 傾向與理由 |
|---|---|---|---|
| **A1** | 不可複製性強度 | (a) t-unclonable（搜尋型，AK21 Def 11：挑戰訊息由 challenger 均勻抽樣）<br>(b) unclonable-IND（AK21 Def 12：A 自選 m₀, m₁） | **兩者都陳述**。(a) 可從資訊論 otUE 無條件得到；(b) 需要 one-time unclonable-IND 原料（目前僅 QROM 有 AKLL22）。HKNY24 App E 的 Lemma E.2 證的是 (b) 的保持，模板直接可用 |
| **A2** | 身分金鑰查詢的時機 | (a) 只在 split **前**（A 查詢）<br>(b) split **後** B、C 各自也能查詢非挑戰身分<br>(c) 兩階段都可 | **目標定 (c)**，起步版本可先做 (a)。(b)/(c) 是 IBE 特有、PKE 版不存在的難度，也是本路線相對 HKNY24 的**定義層貢獻**所在 |
| **A3** | reveal 階段給什麼 | (a) 只給 sk_{id\*}<br>(b) 給整把 msk | **給 msk**（更強且免費）。RNC-IBE 的 Def 6 模擬器 Sim₄ 本來就輸出 msk，所以歸約做得到——見 §3.3 |
| **A4** | 身分的選定方式 | (a) selective-ID（開場即 commit id\*）<br>(b) adaptive-ID | **目標 (b)**。RNC-IBE 的兩個主構造都是 adaptive 安全，所以瓶頸不在槽位。（另注意 Thm 3 的構造有一個為 adaptive 量身打造的性質，見 §4.2） |
| **A5** | 挑戰身分是否可被查詢 | 標準 IBE 規則：id\* 不得在 pre-challenge 查詢 | 沿用。但要注意 reveal 階段給 msk 之後，post-challenge 查詢自動變成 trivial——定義敘述要處理這個交互作用 |

> **交付形態**：這張表本身就是論文「定義章」的骨架。與 KN23 的比較（§5）也放在這一章。

---

## 2. 構造與主定理

### 2.1 構造

```
Setup(1^λ, 1^n):     (mpk, msk) ← RNCIBE.Setup(1^λ, 1^n)
KeyGen(msk, id):     sk_id ← RNCIBE.Keygen(msk, id)

Enc(mpk, id, m):     k ← otUE.Setup(1^λ)                    // 一次性 UE 金鑰
                     ct = ( RNCIBE.Enc(mpk, id, k) ,  otUE.Enc(k, m) )
                            └── 古典側：全有全無地傳 k ──┘   └─ 量子側：裝訊息 ─┘

Dec(sk_id, ct):      k ← RNCIBE.Dec(sk_id, ct₁)
                     m ← otUE.Dec(k, ct₂)
```

古典側可任意複製（複製了也沒用，因為分裂後的兩人在 reveal 前都拿不到金鑰）；不可複製性完全由量子側的 otUE 承擔，並經由「無損傳遞」型定理傳到整體。

**變體（值得順手做）**：RNC-IBE 論文自己的 Thm 11 說明 RNC-IB-KEM ＋ OTP 即得 RNC-IBE。我們的構造其實只需要**封裝一把 k**——所以可以直接把槽換成 **RNC-IB-KEM**，跳過一層 OTP。這樣密文更短，且能吃到 Thm 1「密文尺寸與 session key 長度無關」的好處（雖然 Thm 1 本身非後量子）。**建議主定理以 RNC-IB-KEM 為介面陳述，RNC-IBE 版當 corollary。**

### 2.2 主定理形狀（候選）

> **定理 W1．** 若存在後量子安全的 RNC-IB-KEM（Def 8 意義下）與 one-time unclonable encryption（AK21 Def 13），則存在 unclonable IBE，滿足：
> 1. **IND-ID-CPA 語意安全**，承自 RNC-IB-KEM；
> 2. **t-unclonability 無損承自 otUE**（AK21 Thm 7/8 型的安全度傳遞）；
> 3. 對任意多的**非挑戰身分**金鑰查詢安全；
> 4. 且不可複製性在「reveal 階段交出整把 msk」的強化遊戲下仍成立。
>
> **推論．** 取 X = LWE 的 relaxed RNC-IBE（原論文 Thm 3），得到後量子的 unclonable IBE，身分空間為多項式大小 T。
>
> **一般化定理（把 W 系列收編，論文骨架的可選加分項）**：otUE ＋ 任何 NC 化的 all-or-nothing 加密 ⇒ 該原語的 unclonable 版。PKE 版重現 HKNY24 App E（sanity check）、IBE 當主要實例、CCA 版（W4）與 registered 版（W2）當 corollary。

### 2.3 證明骨架

三段，前兩段是模板、第三段是自己的活：

1. **古典側換假密文**（Hyb₀ → Hyb₁）：用 RNC-IBE 的 Def 6 安全性，把 `(RNCIBE.Enc(mpk, id*, k), msk)` 換成 `(Sim₃ 產生的 ct̃*, Sim₄(st₃, k) 產生的 msk̃)`。此後古典側與 k 無關。
2. **歸約到 otUE**（Hyb₁ → otUE cloning game）：歸約者 Ã 自己跑 Sim₁；收到 otUE 挑戰密文後，把 **Sim 的內部狀態 st₃（古典）附到 B、C 兩個 register 上**；otUE challenger 在 phase 2 亮出 k 時，B̃／C̃ **各自本地**跑 `Sim₄(st₃, k)` 得到**同一把** msk̃，交給 B、C。這正是 HKNY24 App E Prop E.3/E.4 的管線，把「NCE 的 Reveal」換成「RNC-IBE 的 Sim₄」即可。
3. **身分金鑰查詢的 hybrid**（本路線新增）：非挑戰身分的查詢在兩個世界分別由 `Keygen(msk, ·)` 與 `Sim₂(st₁, ·)` 回答——**Def 6 的遊戲已經內建了 pre-challenge 查詢階段**，所以 split 前的查詢是免費的。**真正要自己做的是 A2(b)：split 之後 B、C 各自查詢**。此時歸約者已經沒有真 msk（它在模擬世界），必須靠 Sim₂ 回答，而 Sim₂ 是 stateful 的——兩個分裂的對手各自呼叫同一個 stateful 模擬器，狀態如何協調是這段證明的**唯一硬點**，記為 **D-W1**。

> **D-W1（本路線目前唯一被識別的實質技術風險）**：RNC-IBE 的 Sim₂ 帶內部狀態 st₂ 並會更新它；不可複製遊戲要求 B、C 分裂後各自獨立行動。若允許 split 後查詢，歸約需要「同一個 stateful 模擬器被兩條互不通訊的分支同時呼叫」。三條出路：(i) 起步版只做 A2(a)，把它當 open problem 誠實寫出；(ii) 論證在 reveal 已交出 msk 的情況下，split 後查詢可由 msk̃ 本地自足產生，狀態協調自動消失；(iii) 要求槽位滿足「無狀態模擬」的加強版介面，並檢查 Thm 3 的構造是否恰好滿足。**(ii) 看起來最有機會，且與 A3 的「給 msk」決定互相加強——這是第一個該驗證的技術命題。**

### 2.4 介面對照：E1–E4 vs RNC-IBE Def 6

精讀報告 §2 歸納出槽位真正消耗的四條介面。逐條核對 RNC-IBE：

| 介面 | 要求 | RNC-IBE Def 6 是否滿足 |
|---|---|---|
| **E1** 模糊化不可區分 | (aux_pub, ct_真, key_真) ≈_c (aux_pub, ct_假, key_假)，對 QPT | 是。即 \|p_real − p_sim\| = negl，且 aux_pub 涵蓋 mpk 與所有已查詢金鑰 |
| **E2** 訊息無關生成 | ct_假 可在不知道 k 時生成 | 是。Sim₃(st₂, id\*) 不吃訊息 |
| **E3** 事後綁定金鑰 | key_假 由確定性 PT 從共享古典狀態算出，B、C 各自算得同一把 | 是。Sim₄(st₃, k)；**需固定其隨機性 r 並一併附到兩個 register**（與 AK21 §4.2.1 的 FakeGen 隨機性處理完全同構） |
| **E4** 後量子 | E1 對 QPT 成立 | 視零件而定。**這正是 §4 的全部問題所在**：SXDH 版不滿足；LWE relaxed 版滿足 |

**結論：介面層面完全對上，沒有隱藏的不匹配。** 風險集中在 E4（零件）與 D-W1（定義選 A2(b) 時的證明技術）。

---

## 3. RNC-IBE 精讀：與 W1 相關的關鍵發現

> 出處：Goyal, Kitagawa, Koppula, Nishimaki, Rajasree, Yamakawa，*Non-Committing Identity Based Encryption: Constructions and Applications*，**PKC 2025**（本地 PDF 35 頁；另有 [繁中全文詳解](../papers/RNC-IBE_全文詳解_繁中.pdf)，2026-07-18 整理）。

### 3.1 語法與安全定義

八個演算法：`(Setup, Keygen, Enc, Dec, Sim₁, Sim₂, Sim₃, Sim₄)`。模擬器分工：

- `Sim₁(1^λ, 1^n)` → (mpk, st₁)：**不需要 id\***（selective 版 Def 7 才把 id\* 餵給 Sim₁）
- `Sim₂(st₁, id)` → sk_id，**stateful**（更新 st₂）——這是 D-W1 的來源
- `Sim₃(st₂, id\*)` → (ct\*, st₃)：假密文，不吃訊息
- `Sim₄(st₃, m\*)` → **msk**

安全性是 real/ideal 兩世界的不可區分（Def 6 adaptive／Def 7 selective）。

### 3.2 四個構造與假設

| 構造 | 假設 | 安全性 | 身分空間 | 對 W1 可用？ |
|---|---|---|---|---|
| Thm 1（§5，dual system） | **SXDH**（雙線性） | adaptive，且**對手可拿到 Setup 的全部隨機性**；密文尺寸與 session key 長度無關 | 指數（一般 IBE） | 否：**非後量子** |
| Thm 3（§6，relaxed） | **DDH 或 LWE** | adaptive | **多項式大小 T**，mpk compact | 是：**取 LWE ⇒ 唯一現成的 pq 選項** |
| Thm 5（全文版） | iO + OWF | selective | — | 否：重型工具，違背本路線「不經 iO／FE」的賣點 |
| — | （前人）iO，HMNY21 的 NC-ABE | — | — | 否：同上 |

Thm 3 的構造 = **oblivious batch encryption（Thm 7：CDH／LWE，出自 BLSV18）＋ garbled circuits ＋ NCE（Thm 8：LWE／DDH）**，證明見其 Thm 14。三個零件都有 LWE 實例化，**整條鏈後量子封閉**。

**代價（必須誠實寫進論文）**：其參數為密文尺寸 poly(2^d, |m|, λ)、mpk/msk/sk 尺寸 poly(2^d, λ)，其中 2^d = T = 身分數量。即「多項式大小身分空間」的多項式因子會直接進到密文長度。

### 3.3 意外的好消息：Def 6 交出的是 msk，不是 sk_{id\*}

原論文特別強調（§1.2 相關工作段）：HMNY21 的 NC-ABE 設定就是「對手連同密文一起收到 **master secret key**」。RNC-IBE 的 Def 6 沿用此設定——挑戰階段回傳 (msk, ct\*)。

對 W1 的三重意義：

1. **A3 直接選 (b)**：cloning game 的 reveal 階段可以把整把 msk 給 B 和 C，歸約仍模擬得出來。定理因此更強，且**不花額外代價**。
2. **D-W1 的出路 (ii) 因此變得可信**：既然 B、C 手上有 msk̃，split 後想要哪個身分的金鑰都能自己生，不必回頭呼叫 stateful 的 Sim₂。
3. **賣點清楚**：「即使金鑰機構的主金鑰事後全面洩漏，密文仍然不可複製」——這是一句可以寫進 abstract 的話，也是 IBE 版相對 PKE 版真正多出來的東西。

### 3.4 Thm 3 構造的另一個附加性質

原論文指出，relaxed 構造「非承諾密文可以**與 mpk 一起生成**，不需要知道目標身分 id\*」，並稱此為首見、may be of independent interest。對 W1 的意義：這讓 adaptive-ID（A4(b)）的歸約更寬鬆——假密文不必等到對手宣告 id\* 才產生。**若走甲案，這個性質應該在證明中明確用上並致謝。**

---

## 4. 後量子零件：問題陳述與兩案並陳

### 4.1 問題

W1 的全部技術風險幾乎集中在一句話：**我們要的是後量子 RNC-IBE，而論文最好的構造是雙線性的。**

### 4.2 現成可用的唯一選項

Thm 3 取 X = LWE：adaptive 安全、mpk compact、後量子、且有 §3.4 的附加性質。**唯一缺點是身分空間 poly(λ) 大小、密文含 poly(T) 因子。**

「多項式大小身分空間」在 IBE 語境下是實質的弱化（真實 IBE 的賣點就是拿 email 當身分、身分空間指數大），論文必須誠實標示，不能含混帶過。

### 4.3 兩案並陳（**待拍板**）

| | **甲案：poly-ID 誠實起步** | **乙案：自造 LWE selective RNC-IBE** |
|---|---|---|
| 內容 | 第一定理直接引用 Thm 3 (LWE)，把 poly-ID 寫進定理敘述；全部力氣放在定義、D-W1、與 KN23 的分離 | 用 GPV／ABB 陷門＋可模糊化，自造指數身分空間的 selective-ID RNC-IBE，當論文的技術核心 |
| 論文重心 | 定義層＋compiler 層貢獻（「首個 unclonable IBE」） | 構造層貢獻（「首個 pq RNC-IBE」＋其量子應用） |
| 風險 | 低。零件現成，證明套模板；風險僅在 D-W1 | 中-高。ABB 的 selective 陷門與 RNCE 式可模糊化能否相容未經驗證；失敗則退回甲案 |
| 時程 | 4–6 週可有第一份完整定理 | 難以預估，8 週起跳 |
| 對外強度 | 中。審稿人會問「為什麼只有多項式個身分」 | 高。同時填兩個空格 |
| 競速抗性 | 弱。NTT＋Goyal 產線若要補這格，甲案的內容他們一週就能寫 | 強。技術門檻自帶護城河 |

**判準（建議用一週試探再定）**：
- 試探題目：**ABB (2010) 的 selective-ID 陷門 IBE 能否配上 RNCE 式的 Fake/Reveal？** 具體要檢查的是——Sim₄ 要能事後產生一把「看起來像真 msk」的 trapdoor basis，而 lattice trapdoor 的統計性質（GPV 的 preimage sampling）是否允許這種事後解釋。
- **若一週內能寫出可信的 Sim₁–Sim₄ 骨架 → 走乙案；若卡在陷門的事後解釋 → 走甲案，並把「pq 指數身分空間 RNC-IBE」列為論文的 open problem。**
- 兩案不互斥：甲案的所有產出（定義、主定理、D-W1）在乙案下原封不動可用。**所以無論如何先做甲案的定義與定理陳述，試探平行進行。**

---

## 5. 與既有工作的邊界

| 工作 | 做了什麼 | 與 W1 的關係 |
|---|---|---|
| **AK21**（TCC 2021） | otUE + 古典槽 ⇒ 可重用 UE（私鑰、公鑰） | 配方來源；我們換槽 |
| **HKNY24 App E**（TCC 2024） | RNCE 填槽 ⇒ unclonable PKE，Lemma E.2 保 unclonable-IND | **證明模板**；我們是它的 IBE 版 |
| **HMNY21**（ASIACRYPT 2021） | NC-ABE（iO）⇒ ABE with **certified deletion** | 同配方族譜的先例；做的是**憑證刪除**不是不可複製性。需在 related work 明確區分兩者 |
| **AKL23 §8.3** | UE with certified deletion | 憑證刪除 × UE 這格已滿 |
| **KN23**（TCC 2023） | one-out-of-many unclonable **predicate** encryption（LWE） | **最接近的競品**。它是弱化的「一對多」概念；我們是 AK21 型完整 cloning game。**必須證明蘊含或分離，這是論文的必要章節** |
| **RNC-IBE**（PKC 2025） | 四個 RNC-IBE 構造，應用只做 incompressible IBE | 零件來源 |

**新穎性複查紀律**：「unclonable IBE 文獻中不存在」的判斷來自 2026-07-13 的 ePrint 全文檢索。依 HKNY24 藏在附錄的教訓，**動筆前與投稿前各複查一次**，檢索詞至少涵蓋 unclonable / uncloneable / no-cloning × IBE / identity-based / attribute-based，並追蹤引用 RNC-IBE 與 HKNY24 的論文清單。

---

## 6. 對既有報告的更正與情報更新

| # | 既有敘述 | 更正 |
|---|---|---|
| **C1** | 精讀報告 §6 引用「[HMNY21] *Quantum Encryption with Certified Deletion, Revisited*. **TCC 2021**」 | 正確發表處是 **ASIACRYPT 2021**（依 RNC-IBE 論文參考文獻 [53]，完整標題含 "Public Key, Attribute-Based, and Classical Communication"）。與 AK21 同年但**不同會議**——精讀報告 §3.3 的「同年同會」敘述一併更正 |
| **C2** | 「RNC-IBE 的應用清單只有 incompressible IBE、**零量子應用**」 | 對該篇論文自身成立，但**概念層面不成立**：非承諾的 IBE／ABE 這個概念是 HMNY21 為了**量子應用（憑證刪除）**才引入的。應改寫為「RNC-IBE 這篇論文未做任何量子應用，但其前身 NC-ABE 出自量子動機」——**競速風險應上調** |
| **C3** | Meeting 報告 §4「三個骨架選項、我方傾向方案 A」 | 已由本文取代：主軸定為 W1（≈ 原方案 B），路線甲降為備援／延伸章 |
| **C4** | 「後量子 RNC-IBE 僅 relaxed 版」 | 精確化：relaxed 版是 **Thm 3，X ∈ {DDH, LWE}，adaptive 安全**（不是 selective），零件鏈為 oblivious batch encryption ＋ GC ＋ NCE，密文尺寸 poly(T, \|m\|, λ) |

---

## 7. 執行計畫

**階段一：定義與第一定理（第 1–4 週）**

- 第 1 週：定案 §1.2 的 A1–A5 五個維度，寫出 unclonable IBE 的正式 cloning game（兩個版本：t-unclonable 與 unclonable-IND）。
- 第 1–2 週（平行）：HKNY24 App E 逐行精讀筆記，把 Prop E.3/E.4 的管線改寫成以 Sim₁–Sim₄ 為介面的模板；同時執行 §4.3 的一週試探（ABB × 可模糊化）。
- 第 2 週：**驗證 D-W1 出路 (ii)**——「reveal 給 msk ⇒ split 後查詢自足」這個命題成立與否，決定 A2 能否直接做到 (c)。
- 第 3 週：主定理 W1 的正式陳述與三段證明寫完（以 RNC-IB-KEM 為介面）。
- 第 4 週：**決策點**——甲案／乙案拍板；同時完成與 KN23 的蘊含／分離論證。

**階段二：實例化與強化（第 5–10 週）**

- 後量子實例化（依拍板結果）；unclonable-IND 版本的條件式陳述（需 QROM 的 AKLL22 當 one-time 原料，誠實標示假設）；效率分析與參數表。

**階段三：擴張（第 11 週起，時間允許）**

- 一般化主定理（otUE ＋ NC 化 all-or-nothing ⇒ unclonable 版），把 W2（registered）／W4（CCA）收成 corollary；NC-ABE ⇒ unclonable ABE 列為 future work（NC-ABE 的非 iO 構造尚不存在）。

**平行、隨時可交付**：路線甲的 Clifford 直構 no-go 短文（2–3 頁），目前最接近完成的成果。

---

## 8. 風險與退場條件

| 風險 | 徵兆 | 應對／退場 |
|---|---|---|
| **競速**（最高）：NTT＋Goyal 產線自己補 unclonable 應用 | ePrint 出現 RNC-IBE 的量子應用論文 | 第 4 週前把「定義＋poly-ID 版定理」寫成 note 掛 ePrint 佔位 |
| D-W1 無解，A2 只能做 (a) | 第 2 週的命題驗證失敗 | 起步版做 split-前查詢，把 split-後查詢列為 open problem——定理仍成立，論文仍站得住 |
| 乙案卡死 | 一週試探無法寫出 Sim₁–Sim₄ 骨架 | 回甲案，零損失（§4.3 已設計為可回退） |
| poly-ID 被審稿人視為過弱 | — | 以 §3.4 的附加性質與「首個」定位辯護；並在 open problem 明確標示 |
| 定義被認為與 KN23 重疊 | — | §5 的分離論證是必要章節，不可省 |

---

## 9. 引用

- **[GKKNRY25]** Goyal, Kitagawa, Koppula, Nishimaki, Rajasree, Yamakawa. *Non-Committing Identity Based Encryption: Constructions and Applications*. **PKC 2025**.（本路線的零件來源；Def 6–8、Thm 1/3/11/14）
- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.（配方與 Def 11–13）
- **[HKNY24]** Hiroka, Kitagawa, Nishimaki, Yamakawa. *Robust Combiners and Universal Constructions for Quantum Cryptography*. TCC 2024, arXiv:2311.09487. **Appendix E**（證明模板）
- **[HMNY21]** Hiroka, Morimae, Nishimaki, Yamakawa. *Quantum Encryption with Certified Deletion, Revisited: Public Key, Attribute-Based, and Classical Communication*. **ASIACRYPT 2021**.（NC-ABE 的出處；憑證刪除，非不可複製性）
- **[KN23]** Kitagawa, Nishimaki. one-out-of-many unclonable predicate encryption. TCC 2023.（最接近的競品）
- **[AKL23]** §8.3 UE with certified deletion.
- **[AKLL22]** unclonable-IND 的 QROM 構造（one-time 原料）。
- **[BLSV18]** Brakerski, Lombardi, Segev, Vaikuntanathan.（oblivious batch encryption，RNC-IBE Thm 7 的來源）
- **[ABB10] / [GPV08]**（乙案的陷門工具，待精讀）

---

> **相關報告**：[AK21 精讀報告](./AK21_Close_Reading_and_PQC_Slot_Survey.md)（W1 的提出與槽位介面 E1–E4）｜[Meeting 討論報告](./Meeting_2026-07-15_Two_Routes_Discussion.md)（兩線對照的歷史紀錄）｜[路線甲 Deep Dive](./Deep_Dive_Route_A_Unclonable_IPFE.md)（已降為備援）｜[Reports 閱讀指南](./README.md)
