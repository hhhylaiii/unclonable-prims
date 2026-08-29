# 主定理證明設計：一條 Hybrid 鏈、兩個身分相反的歸約（自足版）

> **日期**：2026-08-28（自足版：不需搭配定義稿閱讀，所有用到的定義與構造在 §1 全文重述）
> **目標**：AK21 Thm 7/8 形狀的傳遞定理——「若一次性方案是 t-unclonable，則我們的構造也是 t-unclonable」——並處理上層 game-based 對下層 simulation-based 的歸約設計。
> **本文另有三項「證明反饋給定義／構造」的修訂建議（§7），寫全文前需先拍板。**

---

## 0. TL;DR

1. **證明只有兩步**：真實不可複製遊戲 →（用 KEM 槽位的模擬安全）→ 中間實驗 Hyb₁ →（用一次性方案的不可複製安全）→ 結束。第一步把古典側**整包**換成模擬器產物（一次換完，不逐層）；第二步把 Hyb₁ 整場遊戲包成一次性方案 cloning 遊戲的一個對手。t 無損傳遞，總損失只有兩個 negl 相加。
2. **兩步裡歸約者的身分相反**：第一步它是 KEM 模擬安全遊戲的「對手」（模擬器在挑戰者手裡，它碰不到）；第二步它是一次性方案遊戲的「對手」兼 KEM 模擬世界的「挑戰者」（模擬器自己跑）。「分裂後查詢的狀態協調問題」只可能在第二步出事，而它已被「凍結」殺掉。
3. **三項需要拍板的修訂**：
   - **修訂一**：不可複製遊戲的金鑰預言機必須寫成 **memoized**（全域一張表，重複查詢回同一把），否則第一步歸約模擬不出重複查詢的聯合分布。
   - **修訂二（最重要）**：一次性方案的揭露階段只亮金鑰 k、不亮其 Setup 的隨機帶 ⇒ 構造裡「把 session key 當 otUE.Setup 隨機帶」那一步在第二步歸約走不通。主定理必須改以 **key-transparent** 的一次性方案（Setup 直接輸出自己的均勻隨機帶當金鑰；BL20 即是）為前提、直接取「session key ＝ otUE 金鑰」；一般方案用一次性密碼本墊一層當推論。
   - **修訂三**：KEM 槽位的模擬安全必須對 QPT 對手陳述（定義稿已如此寫）；引用 GKK25 的 LWE 構造實例化時，其對 PPT 寫的證明要逐條檢查可提升（straight-line 清單見 §7）。

---

## 1. 本文用到的四個對象（全文重述）

記號：λ 安全參數；訊息空間 {0,1}^n；身分空間大小 T = poly(λ)；「≈c」表計算不可區分；QPT = 量子多項式時間。

### 1.1 構造（KEM-DEM）

零件一 **IBKEM**（receiver non-committing IB-KEM）：演算法 `Setup(1^λ,1^T) → (mpk, msk)`、`KeyGen(msk, id) → sk_id`、`Encap(mpk, id) → (ct, k̄)`（k̄ ∈ {0,1}^ℓ 為 session key）、`Decap(sk_id, ct) → k̄`。
零件二 **otUE**（一次性不可複製加密）：演算法 `Setup(1^λ) → k`、`Enc(k, m) → ρ`（量子密文）、`Dec(k, ρ) → m'`。

我們的方案 **UIBE**：

```
Setup(1^λ, 1^T):  (mpk, msk) ← IBKEM.Setup(1^λ, 1^T)
KeyGen(msk, id):  sk_id ← IBKEM.KeyGen(msk, id)

Enc(mpk, id, m):  (ct₁, k̄) ← IBKEM.Encap(mpk, id)
                  k := k̄                    ← 主定理採用的版本（見修訂二；
                                               定義稿原第 2 步「k := otUE.Setup(1^λ; k̄)」
                                               正是要被修訂的對象）
                  ρ ← otUE.Enc(k, m)
                  輸出 ct := (ct₁, ρ)

Dec(sk_id, ct):   k̄' ← IBKEM.Decap(sk_id, ct₁);  輸出 m' ← otUE.Dec(k̄', ρ)
```

### 1.2 要證的目標：UIBE 的不可複製遊戲

**搜尋型**（對手 (A, B, C) 皆 QPT）：

```
建置：      (mpk, msk) ← Setup(1^λ, 1^T)；mpk 給 A
查詢階段 I： A 可適應性查詢 KeyGen(msk, ·)（古典查詢）
挑戰：      A 輸出未查詢過的 id*；挑戰者均勻抽 m ← {0,1}^n，
            ρ ← Enc(mpk, id*, m) 給 A
分裂：      A 施加 CPTP Φ: D(H_A) → D(H_B ⊗ H_C)，
            B 暫存器給 B、C 暫存器給 C；此後 B、C 不得通訊
查詢階段 II：B 與 C 各自獨立查詢 KeyGen(msk, ·)，不得查 id*
揭露：      B 與 C 皆收到整把 msk
猜測：      B 輸出 m_B，C 輸出 m_C
獲勝 ⟺ m_B = m_C = m
```

**t-unclonable 安全**：任意 QPT (A,B,C) 獲勝機率 ≤ 2^{−n+t} + negl(λ)。

**不可區分型**：同上，但挑戰階段改為 A 自選 (m₀, m₁, id*)（等長、id\* 未查詢），挑戰者抽 b ← {0,1} 回傳 Enc(mpk, id\*, m_b)；猜測階段 B、C 輸出位元 b_B、b_C，獲勝 ⟺ b_B = b_C = b；界為 1/2 + negl(λ)。

### 1.3 槽位假設：IBKEM 的模擬安全（simulation-based）

存在模擬器 `Sim₁(1^λ,1^T) → (mpk, st₁)`、`Sim₂(st₁, id) → sk_id`（**帶狀態**，更新 st₂）、`Sim₃(st₂, id*) → (ct*, st₃)`（**不吃 session key**）、`Sim₄(st₃, k̄) → msk`（事後解釋），使得任意 QPT 對手在下述兩個實驗中輸出 0 的機率差 ≤ negl(λ)：

```
真實世界                                模擬世界
─────────────────────────              ─────────────────────────
mpk 由 Setup 產生                       mpk 由 Sim₁ 產生
查詢 id → KeyGen(msk, id)              查詢 id → Sim₂(st₁, id)
A 送出未查詢過的 id*：                   A 送出未查詢過的 id*：
  (ct*, k̄) ← Encap(mpk, id*)            (ct*, st₃) ← Sim₃(st₂, id*)
  回傳 (msk, ct*, k̄)                     k̄ ← {0,1}^ℓ 均勻；msk ← Sim₄(st₃, k̄)
                                          回傳 (msk, ct*, k̄)
A 輸出位元                              A 輸出位元
```

注意：挑戰階段把**整把 msk**連同密文與 session key 交給對手，但不交出 Setup／Encap 的隨機性。此即 GKK25 的 RNC-IB-KEM 安全定義（其 Definition 8），以四模擬器一般形式陳述。

### 1.4 量子側假設：otUE 的不可複製遊戲

**搜尋型**（AK21 的 t-unclonable，其 Definition 11）：

```
phase 1：挑戰者抽 k ← otUE.Setup(1^λ)、m ← {0,1}^n 均勻，
         ρ ← otUE.Enc(k, m) 給 Ã
         Ã 施加 CPTP 分裂成兩暫存器，交給 B̃、C̃（此後不得通訊）
phase 2：金鑰 k 揭露給 B̃ 與 C̃；兩人各自輸出 m_B、m_C
獲勝 ⟺ m_B = m_C = m；t-unclonable：獲勝機率 ≤ 2^{−n+t} + negl
```

**不可區分型**（AK21 其 Definition 12）：Ã 先自選等長 (m₀, m₁) 交給挑戰者，挑戰者抽 b 回傳 otUE.Enc(k, m_b)；最後 B̃、C̃ 輸出位元，全對才贏；界 1/2 + negl。

**關鍵觀察（貫穿全文）**：phase 2 揭露的是**金鑰 k 本身**，不是產生 k 所用的隨機帶。修訂二由此而來。

---

## 2. 定理陳述（建議版本）

> **主定理（搜尋型）.** 設 IBKEM 滿足 §1.3 的模擬安全（對 QPT 對手），otUE 為 **key-transparent**（見下）且滿足 §1.4 的 t-unclonable 安全，且 session-key 長度 ℓ 等於 otUE 金鑰長度。則 §1.1 的 UIBE 滿足 §1.2 的 t-unclonable 安全。具體地，對任意 QPT (A,B,C)：
>
> Pr[獲勝] ≤ 2^{−n+t} ＋ Adv^{模擬安全}_{IBKEM}(λ) ＋ negl(λ)。

> **主定理′（不可區分型）.** 同上前提，把兩側的搜尋型都換成不可區分型，結論界為 1/2 ＋ Adv ＋ negl。

**key-transparent**：`otUE.Setup(1^λ)` 的輸出恰為其隨機帶本身——金鑰是均勻 κ-bit 字串且 `Setup(1^λ; k) = k`。BL20 conjugate encryption 滿足（金鑰 (x, θ) 就是均勻抽樣）；AS26 的 1-bit 方案待核對。

---

## 3. 證明總圖

```
Hyb₀ ＝ §1.2 的真實 UIBE 不可複製遊戲
  │
  │  Claim 1：|p₀ − p₁| ≤ Adv^{模擬安全}_{IBKEM}
  │  歸約 R：把整場遊戲（含量子對手）包成 §1.3 的單一分辨者
  ▼
Hyb₁ ＝ 古典側全部換成模擬器產物（精確定義見 §4）
  │
  │  Claim 2：p₁ ≤ 2^{−n+t} + negl
  │  歸約 (Ã, B̃, C̃)：把 Hyb₁ 包成 §1.4 的對手；Ã 自己扮演模擬世界的挑戰者
  ▼
otUE 的 t-unclonable 安全
```

兩步的結構對稱性（寫進證明前言）：

| | Claim 1 | Claim 2 |
| --- | --- | --- |
| 歸約者對誰 | IBKEM 模擬安全遊戲的挑戰者 | otUE 遊戲的挑戰者 |
| 模擬器誰跑 | 挑戰者跑（歸約者碰不到） | **歸約者自己跑** Sim₁–Sim₄ |
| B、C 的協調問題 | 不存在——歸約者是單一機器，串列處理兩邊 | 存在——Sim₄ 必須在分裂線右邊各跑一次 |
| 由此產生的要求 | 無 | st₃、r、金鑰表 K 附掛到兩個暫存器；Sim₄ 給定 r 後確定性 |

---

## 4. 中間實驗 Hyb₁ 的精確定義

與 Hyb₀ 的差異**只在挑戰者內部**，對手介面完全相同：

1. **建置**：`(mpk, st₁) ← Sim₁(1^λ, 1^T)`（取代 Setup）。
2. **查詢階段 I**：查詢 id 由 `Sim₂(st₁, id)` 回答，**寫入全域金鑰表 K**（memoized：重複查詢回表）。
3. **挑戰**：A 輸出 id\*（未查詢過）。挑戰者先做**凍結**：對所有 `id ∉ dom(K) ∪ {id*}` 依**字典序**逐一呼叫 Sim₂ 補滿 K（至多 T−1 項；此處用掉 T = poly(λ)）。然後 `(ct₁*, st₃) ← Sim₃(st₂, id*)`；均勻抽 `k̄ ← {0,1}^ℓ`；抽定 Sim₄ 的隨機帶 r；`m̃sk := Sim₄(st₃, k̄; r)`。令 otUE 金鑰 `k := k̄`（key-transparent），抽 `m ← {0,1}^n`，`ρ ← otUE.Enc(k, m)`。把 `(ct₁*, ρ)` 給 A。
4. **分裂**：同 Hyb₀。
5. **查詢階段 II**：B、C 的查詢（≠ id\*）**一律回表 K**，不再呼叫 Sim₂。
6. **揭露**：把 m̃sk 給 B、C。
7. **猜測**：同 Hyb₀。

三個設計註記：

- **凍結寫進 Hyb₁ 的定義**（而不是只寫在歸約裡），並固定字典序——這樣 Claim 1 的歸約在模擬世界跑出來的分布跟 Hyb₁ **逐位元相同**，不需要另證「懶惰生成 ≡ 急切生成」對帶狀態的 Sim₂ 成立（那件事根本不見得對，繞開它）。
- 真實世界那一側不需要對稱的凍結：對固定的 msk，`KeyGen(msk, id)` 各次呼叫條件獨立，多生成幾把從未交付的金鑰不改變對手視野（Claim 1 裡一句話帶過）。
- r 在挑戰階段抽定、m̃sk 只算一次發給兩人——在 Hyb₁ 這個「實驗」層級沒有一致性問題；一致性問題到 Claim 2 的歸約才出現，靠「r 預抽＋確定性」解決。

---

## 5. Claim 1：Hyb₀ ≈ Hyb₁（歸約到 IBKEM 的模擬安全）

**歸約 R 的構造**（R 作為 IBKEM 遊戲的**對手**、UIBE 不可複製遊戲的**挑戰者**；是單一 QPT 機器，內部跑完整場不可複製遊戲）：

| 不可複製遊戲階段 | R 對 IBKEM 挑戰者做什麼 |
| --- | --- |
| 建置 | 從挑戰者收 mpk，轉給 A |
| 查詢階段 I | A 的每個查詢轉送挑戰者，回答寫入表 K（memoized） |
| A 宣告 id\* | **【凍結】**對所有 `id ∉ dom(K) ∪ {id*}` 依字典序逐一查詢挑戰者，補滿 K（仍在其查詢階段內——合法：適應性、多項式次、未碰 id\*） |
| 挑戰 | R 送 id\*，收 `(msk̄, ct₁*, k̄)` |
| （R 自己） | `k := k̄`；抽 `m ← {0,1}^n`；`ρ ← otUE.Enc(k, m)`；把 `(ct₁*, ρ)` 給 A |
| 分裂 | 內部執行 A 的 CPTP；之後把兩暫存器分別餵給 B、C，**不讓兩者互動**（no-communication 是對對手結構的約束，R 作為模擬環境照章執行即可） |
| 查詢階段 II | B、C 的查詢一律回表 K |
| 揭露 | 把收到的 msk̄ 原封不動給 B、C |
| 猜測 | 收 m_B、m_C；**R 輸出 1 ⟺ m_B = m_C = m** |

**正確性論證分三個檢查點**：

1. **合法性**：凍結發生在挑戰之前的查詢階段，次數 ≤ T−1 = poly；id\* 從未被查詢（A 被遊戲規則禁止，凍結明確排除）。R 全程 straight-line、不回捲、跑 (A,B,C) 各一次。
2. **真實世界 ⇒ 完美模擬 Hyb₀**：挑戰者用真 KeyGen 回答 ⇒ 表 K 每項恰為 `KeyGen(msk, id)` 的一個樣本；`(ct₁*, k̄) ← Encap` 加上 R 的後續計算恰為 `Enc(mpk, id*, m)` 的逐步展開；msk̄ ＝ 真 msk。**前提：Hyb₀ 的預言機是 memoized 的**（修訂一——否則 B、C 重複查同一 id 在 Hyb₀ 拿到獨立新鮮金鑰、在 R 拿到同一把，聯合分布不同）。
3. **模擬世界 ⇒ 完美模擬 Hyb₁**：挑戰者用 Sim₂ 回答、凍結順序與 Hyb₁ 的字典序一致 ⇒ st₂ 的演化逐位元相同；挑戰階段挑戰者正是「Sim₃、均勻 k̄、Sim₄」；其餘計算與 Hyb₁ 相同。

**機率記號**：

- `p₀`：(A, B, C) 在 Hyb₀ 中獲勝的機率。
- `p₁`：(A, B, C) 在 Hyb₁ 中獲勝的機率。
- `p_real`：IBKEM 挑戰者處於**真實世界**時，R 輸出 1 的機率。
- `p_sim`：IBKEM 挑戰者處於**模擬世界**時，R 輸出 1 的機率。

（§1.3 的安全定義以「輸出 0 的機率差」陳述；因 Pr[輸出 0] = 1 − Pr[輸出 1]，兩種寫法的機率差相同。）

於是 `p_real = p₀`、`p_sim = p₁`，由模擬安全得 `|p₀ − p₁| = |p_real − p_sim| ≤ negl`。∎

**QPT 註記**：R 肚子裡有量子對手與量子密文，故 R 是 QPT 分辨者——這就是 §1.3 必須對 QPT 對手陳述的原因（修訂三談實例化時怎麼交代）。

---

## 6. Claim 2：p₁ ≤ 2^{−n+t} + negl（歸約到 otUE 的不可複製遊戲）

**證明策略**：把整場 Hyb₁ 塞進 otUE 遊戲裡，包裝成該遊戲的一個對手。包裝若做到完美（內部視野逐位元等於 Hyb₁），那麼「Hyb₁ 的獲勝率」就等於「這個包裝對手在 otUE 遊戲的獲勝率」，而後者被假設壓住——界就這樣轉嫁過來。

**兩組人馬的身分**：

- **(A, B, C)**：跟 Claim 1 是**同一組人**——攻擊我們 UIBE 方案的那個任意對手。它以為自己在打 Hyb₁，不知道自己被包了起來（全稱量詞：對所有 (A, B, C) 都要成立）。
- **(Ã, B̃, C̃)**：**我們親手造的 otUE 遊戲對手**（存在量詞：我們展示一個）。每一位都是「外殼＋內芯」——內芯是對應的 A、B、C；外殼把 otUE 遊戲翻譯成 Hyb₁ 給內芯看，並分擔 Hyb₁ 挑戰者的職務：Ã 負責分裂前（Sim₁、Sim₂、凍結、Sim₃、預抽 r），B̃、C̃ 各自負責分裂後（查表答詢、本地跑 Sim₄）。

**Wrapper (Ã, B̃, C̃) 的構造**：

**Ã**（otUE 遊戲 phase 1）：

1. 自己跑 `Sim₁ → (mpk, st₁)`，mpk 給 A；用 `Sim₂` 回答查詢階段 I（建表 K）。
2. A 宣告 id\* 後：凍結補滿 K；`(ct₁*, st₃) ← Sim₃(st₂, id*)`；**抽定 r**。
3. 從 otUE 挑戰者收 `ρ ← otUE.Enc(k, m)`（k、m 由 otUE 挑戰者抽——分布恰與 Hyb₁ 相同：k 均勻 ⟸ key-transparent ＋ k̄ 均勻；m 均勻）。
4. 把 `(ct₁*, ρ)` 給 A，跑 A 的分裂 CPTP 得 ρ_BC。
5. **輸出的分裂映射**：`ρ_B ⊗ |cls⟩⟨cls|` 交給 B̃、`ρ_C ⊗ |cls⟩⟨cls|` 交給 C̃，其中 **cls := (st₃, r, K)** 是古典字串、可自由複製兩份。整段是一個合法 CPTP。

**B̃**（otUE 遊戲 phase 2，收到揭露的 k）：

1. 內部跑 B（餵 ρ_B）；B 在查詢階段 II 的查詢用附掛的 K 回答。
2. otUE 挑戰者亮出 k 後：**本地計算 `m̃sk := Sim₄(st₃, k; r)`**，交給 B 作為揭露。
3. 輸出 B 的 m_B。

**C̃** 對稱（用**同一份** st₃、r 的拷貝）。由 Sim₄ 給定 r 後的確定性，B̃、C̃ 算出**同一把** m̃sk——這正是定義稿把「確定性＋顯式隨機帶」寫進槽位介面的兌現點。

**視野比對**：(A, B, C) 在 wrapper 內部看到的一切與 Hyb₁ 逐位元同分布（唯一重排是「m̃sk 從挑戰階段延後到揭露階段才計算」——它在 Hyb₁ 裡本來就只在揭露階段被用到，且其輸入 (st₃, k̄, r) 在分裂前已全部定案）。勝利條件相同，故

`Pr[(Ã,B̃,C̃) 贏 otUE 遊戲] = p₁ ≤ 2^{−n+t} + negl`。∎

**與 Claim 1 合併（主定理的界）**：

`p₀ ≤ p₁ + negl₁ ≤ 2^{−n+t} + negl₁ + negl₂ = 2^{−n+t} + negl(λ)`

（negl₁ 來自 Claim 1 的 IBKEM 模擬安全、negl₂ 來自 Claim 2 的 otUE 不可複製安全；兩個可忽略函數相加仍可忽略。）

**時序核對**（審稿人必問）：查詢階段 II 在揭露之前，而 otUE 遊戲 phase 2 開頭就亮 k——不衝突：B̃ 先跑 B 的查詢階段 II（只用 K，不需要 k），跑到 B 等待揭露時才把 k 代入。otUE 遊戲只看 B̃ 最後輸出的字串，看不到內部步驟。

---

## 7. 三項需要拍板的修訂

### 修訂一：預言機必須 memoized（定義稿修訂）

**問題**：不可複製遊戲現行文字的查詢預言機是 `KeyGen(msk,·)`。若 KeyGen 有隨機性、且 B 與 C 各查了同一個 id（或同一人查兩次），真實實驗給出**獨立**兩把金鑰；而 Claim 1 的歸約只能從表 K 給出**同一把**。聯合分布不同 ⇒ 歸約不完美。

**修訂**：在兩個不可複製遊戲的定義各加一句（或 Remark）：挑戰者維護單一金鑰表，**同一身分的重複查詢（跨階段、跨 B/C）回傳同一把金鑰**。理由：(i) IBE 文獻的標準慣例之一，對應「PKG 對每個身分只發一把金鑰」；(ii) 對 KeyGen 為確定性的方案（含 GKK25 支援多項式身分空間的構造——其 KeyGen 是 msk 的投影）此句為空話；(iii) 槽位的 Sim₂ 本來就 memoized，上下層正好對齊。

### 修訂二：reveal-computability——第二步歸約真正的介面約束（構造修訂）

**問題**：otUE 遊戲的 phase 2 **只亮金鑰 k，不亮 Setup 的隨機帶**。Claim 2 裡 B̃ 要在線右邊本地算 `Sim₄(st₃, k̄; r)`，手上只有 k 與分裂前附掛的古典資料。若照定義稿構造原本的第 2 步 `k := otUE.Setup(1^λ; k̄)`（session key 當隨機帶），B̃ 需要從 k 反推 k̄——一般的 Setup 不可逆，**歸約卡死**。

一般化的教訓（值得寫成定義章的一條 remark）：

> **凡是揭露階段要在分裂線右邊重算的東西，其全部輸入必須是「k 本身 ＋ 分裂前可複製的古典資料」的高效確定函數。** Sim₄ 的第二個輸入 k̄ 因此必須從 k 高效可得。

**三條出路**：

1. **（主定理採用）key-transparent otUE**：`Setup(1^λ; k) = k`，取 `k := k̄`，則 `Sim₄(st₃, k̄; r) = Sim₄(st₃, k; r)`，k 亮出即可算。BL20 滿足；把定義稿構造的 Remark（「若金鑰均勻可直接取 k := k̄」）從效率備註**升級為主定理前提**。
2. **（一般 otUE 的推論）一次性密碼本墊一層**：`ct := (ct₁, c₂ := k̄ ⊕ k, ρ)`，其中 k ← otUE.Setup(1^λ)。模擬時 c₂ 抽均勻；揭露時 `k̄ := c₂ ⊕ k` 可從 k 算出，餵 Sim₄。這其實就是走 GKK25「KEM ＋ OTP ⇒ 訊息版」的介面、以 k 為挑戰訊息——代價是密文多 |k| bits。
3. 注意 **key-transparency 不是無害正規化**：想把任意 otUE 改造成「金鑰＝隨機帶」，會把其遊戲的揭露物從 k 換成隨機帶，是**更強**的揭露，原方案的 t-unclonability 不自動傳過去（同一個不可逆問題）。所以出路 1 是真前提——定理敘述要誠實。

**待辦**：核對 AS26 的 1-bit 方案是否 key-transparent；若否，主定理′ 的 1-bit 實例化走出路 2。

### 修訂三：QPT 提升檢查清單（實例化時）

§1.3 對 QPT 陳述是我們的**假設**；GKK25 對 LWE 實例的證明是對 PPT 寫的。引用前逐條檢查其安全證明（該文 Lemma 12–14）：歸約是否全部 straight-line（無回捲、無 forking）；三個零件（oblivious batch encryption、Yao garbled circuit、非承諾加密）是否都有 LWE／後量子 PRG 實例。預期成立（都是標準 hybrid），但論文要一段附錄明寫。

---

## 8. 主定理′（不可區分型）的平行版本

只有三處不同：

1. **Claim 1 的 R**：A 在挑戰階段交出 (m₀, m₁, id\*)；R **自己抽 b**、算 `ρ ← otUE.Enc(k, m_b)`；最後輸出 1 ⟺ b_B = b_C = b。
2. **Claim 2 的 Ã**：跑內部 A 到它吐出 (m₀, m₁, id\*)，把 (m₀, m₁) 作為自己在 otUE 不可區分遊戲的選擇送給挑戰者，收 `ρ ← otUE.Enc(k, m_b)`（b 由挑戰者抽）。其餘照舊；B̃、C̃ 輸出 b_B、b_C。時序無衝突：該遊戲本來就是「對手先交訊息對、再收密文」。
3. **界**：1/2 ＋ Adv ＋ negl。

Hyb₁ 與凍結、附掛、Sim₄ 的處理**一字不改**——這是把 Hyb₁ 設計成與挑戰訊息選法無關的紅利。

---

## 9. 附帶引理與使用點對照

**附帶引理（IND-ID-CPA，一段話）**：把 §1.3 兩個世界的視野投影掉 msk 後仍不可區分；模擬世界裡 k̄ 均勻且與 (m̃pk, 金鑰, ct̃₁\*) 獨立 ⇒ IBKEM 是 IND-ID-KEM-CPA ⇒ 配 otUE 的一次性不可區分安全做兩步標準 hybrid，得 UIBE 的 IND-ID-CPA。（放論文附錄。）

**使用點對照表**（每個定義選擇在證明哪一步被用掉；也是「定義沒有多餘零件」的自我檢查）：

| 定義選擇 | 被用在 |
| --- | --- |
| adaptive（挑戰時才宣告 id\*） | 凍結排在宣告之後、送挑戰之前——不需猜 id\*、不需 complexity leveraging |
| T = poly(λ) | 凍結的查詢次數 ≤ T−1 |
| 查詢階段 II 在揭露前 | 兩個 Claim 均由表 K 回答；若移到揭露後（弱定義）證明反而更簡單 |
| 揭露給整把 msk | 恰為槽位挑戰階段的交付物型別——Sim₄ 輸出 msk |
| 古典金鑰查詢 | 查詢階段 I 要逐筆轉送古典的槽位挑戰者 |
| memoized 預言機（修訂一） | Claim 1 檢查點 2 |
| Sim₄ 確定性＋顯式隨機帶 r | Claim 2 的 B̃/C̃ 一致性 |
| key-transparent otUE（修訂二） | Claim 2 的 reveal-computability |

---

## 10. 下一步

1. **拍板修訂一、二**（下次 meeting 第一個議題）：遊戲定義加 memoized 句；主定理前提加 key-transparent、構造 Remark 升級；OTP 推論要不要寫。
2. 照 §4–§6 把 LaTeX 證明寫出（順序：Hyb₁ 定義 → Claim 1 → Claim 2 → 不可區分型差異段 → 附帶引理）。直接對 §1.3 的模擬安全證，「事後可解釋主金鑰」介面留作說明性橋接。
3. 修訂三的 QPT-lift 附錄（對 GKK25 Lemma 12–14 逐條）。
4. 核對 AS26 是否 key-transparent。

---

## 11. 引用

- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.（一次性方案的兩個遊戲＝其 Definition 11/12；證明模板＝其 Theorem 7/8：先換 fake-key、再包成 otUE 對手）
- **[GKK25]** Goyal, Kitagawa, Koppula, Nishimaki, Rajasree, Yamakawa. *Non-Committing IBE*. PKC 2025.（槽位模擬安全＝其 Definition 8；LWE 實例＝其 Theorem 3/14；KEM＋OTP 轉換＝其 Theorem 11）
- **[HKNY24]** *Robust Combiners and Universal Constructions for Quantum Cryptography*. TCC 2024.（附錄的 unclonable 公鑰加密：Claim 2 wrapper 的先例）
- **[BL20]** Broadbent, Lord. TQC 2020.（key-transparent 的 otUE 實例）
- **[AS26]** Ananth, Sahai. *Unconditional Unclonable Encryption*. ePrint 2026/1511.（1-bit 不可區分型原料；key-transparency 待核對）
