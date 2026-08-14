# Unclonable IBE 定義章草稿：game-based 上層與 simulation-based 槽位如何分層、四個模擬器對應 AK21 的哪一步、cloning game 定義 A/B 與 KEM 版槽位介面

> **日期**：2026-08-07。回應「AK21 是 game-based、GKK25 是 simulation-based，我們該怎麼定義」
> **上位文件**：[Unclonable IBE 主軸路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)——本文細化其 §1（定義維度 A1–A5）與 §2（證明骨架與 D-W1）。

---

## 0. 定位：兩種風格不是對立，是分層

| 層級 | 原語 | 風格 | 為什麼 |
| --- | --- | --- | --- |
| 上層（本文的目標原語） | Unclonable IBE | **Game-based**（cloning game） | 不可複製性是「兩人同時成功」這個**事件**的機率上界，沒有自然的 ideal functionality；且 BL20／AK21／AKL23 全系列都是 game-based |
| 下層（槽位） | RNC-IB-KEM | **Simulation-based** | 因為有 adaptive 金鑰查詢預言機。AK21 的 fake-key（Def 16）是「兩個分布 ≈c」的一次性寫法，一旦加上互動就撐不住，**必須**升級成 stateful 模擬器 |

> **一句話**：simulation-based 不是另一種哲學，它是 fake-key 在「原語帶有 adaptive 查詢」之後的**必要推廣**。AK21 的 Def 16 之所以能寫成兩個分布，只因為私鑰加密的 Setup 是平凡的、沒有預言機。

論文的接法：**上層定義用 game，下層槽位用 simulation，compiler 定理把後者當成一個 hybrid step 吃掉。** 這正是 AK21 §4.2.1 的結構（Def 16 是 simulation-in-disguise，只在證明的 Hybrid 1 → Hybrid 2 出現一次），我們只是把那一步換成一個真正 stateful 的版本。

---

## 1. 四個模擬器對應到 AK21 的什麼

AK21 的 fake-key 只有兩個動作：

- **(a) 不知道訊息就先生密文** → `Enc(k, 0)`
- **(b) 事後生一把金鑰，把 ct₀ 解釋成加密 m** → `FakeGen(ct₀, m)`

GKK25 把它拆成四個，是因為 IBE 多了 Setup 與查詢預言機：

| GKK25 | AK21 對應 | 角色 |
| --- | --- | --- |
| `Sim₁(1^λ,1^T) → (mpk, st₁)` | **無**（私鑰版 Setup 平凡） | 生出帶陷門的 mpk |
| `Sim₂(st₁, id) → sk_id`（stateful） | **無** | 維持非挑戰身分的一致性 |
| `Sim₃(st₂, id\*) → (ct\*, st₃)` | `Enc(k, 0)` | **(a)** 訊息無關生成 |
| `Sim₄(st₃, m\*) → msk` | **`FakeGen`** | **(b)** 事後解釋 |

**更正一個容易搞混的地方**：fake-key 的對應物是 **Sim₄，不是 Sim₂**。

兩者都「生假金鑰」，但角色相反：

- `Sim₄` 是**事後解釋（post-hoc equivocation）**——密文已經在對手手上、訊息才被指定，金鑰要回頭把它圓起來。這才是 fake-key 的定義性特徵。
- `Sim₂` 是**一致性維護**——它產生的是**挑戰之前**、**非挑戰身分**的金鑰，時間順序是正的，沒有任何「回頭解釋」。

在 §6 的具體構造裡兩者的差別看得更清楚：`Sim₂` 把非挑戰身分的 NCE 密文解釋成**隨機垃圾訊息**，`Sim₃`(§6 的 explain) 才把挑戰身分解釋成**真正的 k**。

**這個區分是整個定義設計的樞紐**：Sim₄ 是 AK21 已經處理過的東西（照抄 FakeGen 的隨機性技巧即可），**Sim₂ 才是 IBE 帶進來的全新負擔**，也是 D-W1 的唯一來源。

---

## 2. 設計主軸：用 split 線切開模擬器序列

cloning game 相對於一般安全遊戲，只多了一件事：**中間有一條 split 線，線右邊 B 和 C 不能通訊。**

於是設計規則只有一條：

> **凡是必須在 split 右邊執行的模擬器，B 和 C 都得各跑一次，而且必須跑出同一個結果。**

把四個模擬器按執行時點排開：

| 模擬器 | 執行時點 | 在 split 右邊？ | 施加的定義約束 |
| --- | --- | --- | --- |
| `Sim₁` | 遊戲開場 | 否 | 無 |
| `Sim₂` | 金鑰查詢 | **看 A2 怎麼選** | ← **D-W1 的全部內容** |
| `Sim₃` | 產生 ct\* | 否 | 無 |
| `Sim₄` | **reveal** | **是**（B、C 各跑一次） | 必須：確定性給定 r／只吃古典可複製的 st₃／r 在 split 前抽好並複製進兩個 register |

**於是 A1–A5 五個維度可以重新理解成「這條線畫在哪」：**

| 維度 | 它其實在問什麼 |
| --- | --- |
| **A2** 查詢時機 | `Sim₂` 要不要被推到線的右邊。推過去 ⇒ 兩條分支各自更新 st₂ ⇒ 分岔 ⇒ 定義不可證 |
| **A3** reveal 給什麼 | `Sim₄` 的輸出型別（msk／sk_{id\*}／setup 亂數） |
| **A4** adaptive vs selective | `Sim₁` 吃不吃 id\* |
| **A1** 強度 | 與模擬器無關，純粹是量子側 otUE 的原料問題 |
| **A5** id\* 不可查詢 | Def 6 的遊戲規則直接沿用 |

**A2 的三個解**（皆已驗證）：

1. **凍結 st₂**：歸約者在 A 宣告 id\* 之後、split 之前，主動把所有 id ≠ id\* 查完（T−1 次）。線右邊不再有 Sim₂。只在 poly-ID 成立。
2. **先 reveal 再查詢**：B、C 拿到 msk̃ 後自己跑 `Keygen(msk̃,·)`。因為 Keygen 是公開演算法、msk 本來就在 Def 6 的 view 裡，不可區分性原封不動傳遞。任何身分空間大小都成立，但定義較弱。
3. 只做 split 前查詢（起步版）。

---

## 3. 定義草稿

### 3.1 語法

```
Setup(1^λ, 1^T) → (mpk, msk)
KeyGen(msk, id) → sk_id
Enc(mpk, id, m) → ρ_ct        （量子密文）
Dec(sk_id, ρ_ct) → m'
```

### 3.2 Definition A（Unclonable Security，對應 AK21 Def 11）

QPT 對手 (A, B, C)，訊息長度 n。

```
Expt^{cloning}_{UIBE,(A,B,C)}(1^λ, 1^T):

  Setup:        (mpk, msk) ← Setup(1^λ, 1^T);  送 mpk 給 A
  Query-1:      A 取得預言機 O(·) := KeyGen(msk, ·)
  Challenge:    A 輸出 id*（Query-1 中未查詢過）
                m ← M 均勻抽樣
                ρ ← Enc(mpk, id*, m);  送 ρ 給 A
  Split:        A 施加 CPTP φ: D(H_A) → D(H_B) ⊗ D(H_C)
                B register → B，C register → C
                【此後 B 與 C 不得通訊】
  Query-2:      B 與 C 各自獨立取得預言機 O(·) := KeyGen(msk, ·)，限 id ≠ id*
  Reveal:       B 與 C 皆收到 msk
  Guess:        B 輸出 m_B，C 輸出 m_C

  Win  ⟺  m_B = m_C = m
```

**t-unclonable 安全**：對任意 QPT (A,B,C)，`Pr[Win] ≤ 2^{-n+t} + negl(λ)`。

### 3.3 Definition B（Unclonable-Indistinguishable Security，對應 AK21 Def 12）

同上，但把 Challenge 與 Guess 改成：

```
  Challenge:    A 輸出 (m₀, m₁, id*)，|m₀| = |m₁|，id* 未被查詢
                b ← {0,1};  ρ ← Enc(mpk, id*, m_b);  送 ρ 給 A
  Guess:        B 輸出 b_B，C 輸出 b_C
  Win  ⟺  b_B = b_C = b
```

**unclonable-IND 安全**：`Pr[Win] ≤ 1/2 + negl(λ)`。

### 3.4 定義敘述時必須講清楚的五件事

1. **Query-2 必須排在 Reveal 之前**，否則它是冗餘的（拿到 msk 之後 B、C 自己就能生金鑰）。**Query-2 的存在與否＋它與 Reveal 的先後，就是本定義相對 HKNY24 PKE 版唯一多出來的自由度**，也是唯一有技術內容的地方。
2. **B 和 C 各自持有對「同一個挑戰者」的獨立預言機存取**。這句要明寫——它正是 stateful 模擬器會出事的地方，讀者需要看到我們知道自己在說什麼。
3. **id\* 的禁止查詢只約束 Query-1／Query-2**；Reveal 之後不設限（給了 msk 就無意義了）。
4. **Reveal 給整把 msk**（A3(b)）。可寫成 remark：由於 §6 的構造連 Setup 亂數都能事後解釋，可再強化成「reveal 交出 PKG 的 setup 隨機性」。
5. 金鑰查詢是**古典**查詢（不允許疊加態查詢）。要註明；疊加查詢版留 open problem。

---

## 4. 槽位定義：把 Def 6 重寫成 fake-key 的形狀

為了讓 compiler 定理跟 AK21 Thm 7 長得一模一樣，建議論文自己給一個介面定義，寫成「帶預言機的兩分布不可區分」：

> **Definition（IBE with Post-hoc Equivocable Master Key）.**
> IBE = (Setup, KeyGen, Enc, Dec) 滿足此性質，若存在 PPT (Sim₁, Sim₂, Sim₃, Explain) 使得對所有 QPT 對手 A：
>
> ```
> ⎧ mpk,  O_KeyGen(msk,·),  ct* ← Enc(mpk, id*, k),  msk ⎫
> ⎩                                                       ⎭
>                            ≈_c
> ⎧ m̃pk,  O_Sim₂(st₁,·),   c̃t* ← Sim₃(st₂, id*),   Explain(st₃, k; r) ⎫
> ⎩                                                                      ⎭
> ```
>
> 其中 (m̃pk, st₁) ← Sim₁(1^λ,1^T)，`Sim₃` **不吃 k**，`Explain` 對固定的 r 是**確定性**的。

三個好處：

- 左右兩邊擺在一起，**視覺上就是 AK21 Def 16 加了一個預言機**——老師和審稿人一眼看懂脈絡。
- `Explain` 的確定性與 r 被寫進定義，E3 不再是證明裡臨時冒出來的技術要求。
- 這個介面**比 Def 6 弱**（見下），所以可以宣稱「多個構造都填得進來」。

### 4.1 我們需要的其實是 KEM 版，而且比 Def 6 弱

Def 6 讓**對手選** m\*；我們的 m\* 是 otUE 的金鑰 k，由**歸約者均勻抽樣**。也就是說我們只需要「隨機訊息版」的 RNC-IBE。

而這恰好就是 **Def 8（RNC-IB-KEM）**：其挑戰階段是 `msk ← Sim₄(st₃, seskey)，其中 seskey 是隨機生成的`，回傳 (msk, ct\*, seskey)。

> **所以主定理以 RNC-IB-KEM 為介面陳述，不只是「密文比較短」的效率理由，而是定義上就是對的介面。** RNC-IBE 版當 corollary（GKK25 Thm 11：RNC-IB-KEM + OTP ⇒ RNC-IBE）。

---

## 5. 兩步 hybrid 的結構對稱性（E3 的真正來源）

證明只有兩步，而這兩步裡歸約者的**身分是相反的**——這一點值得在證明前言講明，因為它解釋了為什麼只有 Sim₄ 有「複製到兩個 register」的要求：

| | Hyb₀ → Hyb₁ | Hyb₁ → otUE cloning game |
| --- | --- | --- |
| 歸約者是誰 | RNC-IB-KEM 遊戲的**對手** | otUE cloning game 的**對手**，同時**自己扮演** RNC-IB-KEM 的挑戰者 |
| 能不能碰模擬器 | **不能**（模擬器在挑戰者那邊） | **能**（自己跑 Sim₁–Sim₄） |
| B、C 的協調問題 | **不存在**。挑戰者一次交出 (msk, ct\*, k)，歸約者原封不動同時發給 B 和 C | **存在**。k 要到 reveal 才由 otUE 挑戰者亮出，`Sim₄` 只能在 split 右邊、由 B̃／C̃ 各跑一次 |
| 產生的技術要求 | 無 | **E3**：`Sim₄` 確定性、st₃ 古典可複製、r 在 split 前抽好並複製 |

第二列是關鍵：**E3 這條介面要求不是憑空提出的，它是「歸約者在第二步變成了模擬器」這件事的邏輯後果。** 這個說法比 roadmap 現在「照抄 AK21 FakeGen 隨機性處理」的說法有力得多，也是定義章可以拿來當論證骨幹的東西。

歸約者在 Hyb₀ → Hyb₁ 是**單一機器**（B、C 不能互通是對 (B,C) 的約束，不是對歸約者的約束），所以它可以把兩邊的查詢**串列**轉送給同一個 stateful 挑戰者——D-W1 在這一步本來就不存在。D-W1 只可能出現在第二步，而第二步的 Sim₂ 已經被 §2 的凍結技巧清空。

---

## 6. 一頁式的敘事（寫進 intro 的版本）

1. AK21 的配方需要古典槽提供一個性質：**密文先生、金鑰後圓**（fake-key）。
2. 私鑰加密沒有預言機，所以 fake-key 可以寫成兩個分布的不可區分——看起來是 game-based。
3. 一旦把槽位換成 IBE，多了 adaptive 金鑰查詢，兩分布的寫法就不夠了，必須升級成 stateful 模擬器——這就是 GKK25 的 Def 6 為什麼是 simulation-based。**風格的改變是原語結構逼出來的，不是品味問題。**
4. 但 cloning game 有一條 split 線，線右邊的模擬器要跑兩次且必須一致。四個模擬器裡只有 `Sim₄` 落在線右邊（`Sim₂` 可用凍結技巧移到左邊），而 `Sim₄` 恰好就是 fake-key 的對應物——**AK21 早就示範過怎麼處理它（固定 FakeGen 的隨機性、複製到兩個 register）。**
5. 所以：目標原語仍是 game-based，槽位是 simulation-based，兩者在 split 線上接合，而接合處的技術要求（E3）正是 AK21 已經解決的那一格。
