# KT18 × 不可複製加密：RSO 機制直接填槽（任意 IBE ⇒ UIBE）、KDM 機制的雙槽設計、兩個新安全概念與證明骨架

> **日期**：2026-09-28
> **議題來源**：老師希望下一步把 Kitagawa–Tanaka〈Key Dependent Message Security and Receiver Selective Opening Security for Identity-Based Encryption〉（PKC 2018，下稱 **KT18**）的機制，像目前把 RNC-IB-KEM 接上 otUE 那樣，與不可複製加密結合。
> **上位文件**：[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（Construction 1、Definition 2/3、Claim 1/2、Remark 2「時間邊界是分裂」、Remark 3「揭露 msk 的強化版」）。
> **配套 LaTeX 稿**：`W1_KT18結合_構造定理與證明_LaTeX_2026-09-28.tex`（claude.ai 專案內；編譯後的 PDF 收在 [`Paper drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf`](../Paper%20drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf)）——本文 §3、§4 的定義、構造、定理與完整 hybrid 證明的正式版。
> **一句話結論**：可以，而且 KT18 的兩個機制各有去處。**RSO 機制（雙重加密）可以直接取代 GKK25 當古典槽**——因為 09-13 起揭露物改成 sk_{id\*}，槽位只需要「對使用者金鑰非承諾」，而 KT18 從**任意** IND-ID-CPA IBE 就給得出這個性質；得到的 UIBE 在 LWE 下支援**指數大小身分空間**，並順帶對任意身分空間解除 v3.1 Remark 2 的時間邊界。**KDM 機制不能直接填 k 的槽**（KDM 與事後模糊化互相衝突），但可以當**第二個古典槽**裝「被遮罩的訊息」，得到同時具備 KDM 安全、且「在 key-dependent 密文存在下仍不可複製」的 UIBE——文獻中沒有的新組合。

---

## 0. TL;DR

1. **KT18 其實只有一個核心技術**（§2.3）：把 SKE 的某種「證明時需要模擬金鑰」的安全性，經由 garbled circuit ＋「依金鑰位元分身分」的 IBE，搬到 IBE 上。填 KDM-SKE 得到 KDM-IBE（KT18 §4）；填一次性密碼本得到**對使用者金鑰非承諾**的 IBE，而後者可化簡成不用 GC 的**雙重加密**（KT18 §5，RSO）。
2. **最重要的觀察**（§3.1）：v3.1 的定義揭露的是 sk_{id\*}，不是 msk。證明真正消耗的槽位性質因此只是「**使用者金鑰層級的 receiver non-committing**」，嚴格弱於 GKK25 Def 8（交出 msk）。KT18 §5 的雙重加密正好提供它，而且**只假設 IND-ID-CPA IBE**。這條分界與 GKK25 自己在 §1 描述的 incompressible IBE 兩個版本相呼應：揭露 sk_{id\*} 的 regular 版在 GKK25 之前就有構造（GKRV25），揭露 msk 的 strong 版要到 GKK25 以 RNC-IBE 才首次做出。
3. **Construction 2（新）**：`ct = ( {IBE.Enc((id, j, α), k_j)}_{j∈[κ], α∈{0,1}} , otUE.Enc(k, m) )`，`sk_id = (s, {sk_{(id,j,s_j)}}_j)`。**Theorem 3**：IBE 對 QPT 為 IND-ID-CPA ＋ otUE 為 t-unclonable ⇒ Construction 2 為 t-unclonable（unclonable-IND 版平行），界 `2^{−n+t} + 2κ·Adv^{ibe} + negl`（Adv 取 |Pr[β′=β] − 1/2| 的慣例）。證明仍是兩步（κ 個 IND-ID-CPA hybrid ＋ 重參數化，再包成 otUE 對手），**但整條證明沒有任何模擬器**——wrapper 自己持有真的 MSK。
4. **對主軸的直接影響**（§3.6）：取標準模型下 LWE 的 adaptive IBE（ABB10 的 adaptive 版、CHKP10、Yam17）即得**後量子、指數身分空間**的 UIBE——路線圖 §4 的甲案／乙案困境，對現行定義而言**消失**。GKK25 路線保留為「揭露整把 msk」強化版（v3.1 Remark 3）的專用實例。代價：msk 強化版不在 Construction 2 的證明範圍；分裂後仍須禁查 id\*（或把 KeyGen 以 PRF 去隨機化）。
5. **RSO 延伸**（§3.7）：Construction 2 的所有非挑戰金鑰都來自真 MSK，所以 **A 在挑戰之後、分裂之前查詢金鑰也證得出來，且對任意身分空間成立**（v3.1 Remark 2 的邊界，Construction 1 只能在 poly-ID 下以 Sim₂ 預答繞過）。據此可定義「多目標＋選擇性開啟」的不可複製遊戲（Theorem 4，損失因子 q）：密文送出之後才腐化部分接收者，正是 RSO 的情境；再加一層遮罩的變體 2′ 同時達到 KT18 意義的 SIM-RSO 機密性（Proposition 1）。
6. **KDM 放進 cloning game 有三個陷阱**（§4.1；(i) 是真正的平凡攻擊，(ii)(iii) 是證明技術上的障礙、不是不可能性結果）：(i) 挑戰訊息若依賴 sk_{id\*}，搜尋型定義平凡可破（揭露後兩人都算得出訊息）；(ii) UIBE 本身是 KEM–DEM，KT18 §1.2 指出的「混合加密死結」原封不動出現；(iii) 被事後模糊化的金鑰材料（Construction 2 的選擇字串 s\*）不能出現在分裂前就必須產生的 KDM 訊息裡。
7. **Construction 3（新，雙槽）**（§4.3）：量子層只裝均勻的遮罩 y；otUE 金鑰 k 走 RSO 槽（非承諾）、被遮罩的訊息 z = y ⊕ m 走 KT18 的 KdmIBE 槽（KDM）。**Theorem 5**：KDM-CPA 安全（只用到 KdmIBE）。**Theorem 6**：**KDM-unclonable**——即使對手在分裂前拿到 key-dependent 密文（函數在函數族 F 中、可依賴所有登錄金鑰，包括把目標金鑰 sk_{id\*} 加密給其他身分、金鑰循環；唯一限制是目標本身不當 KDM 接收者），挑戰密文仍 t-unclonable。證明的關鍵是**順序**：先用 KDM 安全把輔助密文中和（它們的接收者金鑰永不揭露），再模糊化目標。全部零件都有 LWE 實例。
8. **Open**（§4.7）：挑戰訊息本身依賴目標金鑰的 KDM-unclonable-IND；揭露 msk 的 KDM 版；相關訊息分布下的 simulation-based RSO 不可複製性。

---

## 1. 出發點：v3.1 的形狀與三個結構性弱點

v3.1（2026-09-27）的構造、定義與證明：

```
Construction 1:  (ct₁, k̄) ← IBKEM.Encap(mpk, id);  k ← otUE.Setup;  c₂ := k̄ ⊕ k;  ρ ← otUE.Enc(k, m)
                 ct := (ct₁, c₂, ρ)                    IBKEM = GKK25 的 RNC-IB-KEM（Def 8）

Definition 2:    查詢 I（A，挑戰前）→ 挑戰 → 分裂 → 查詢 II → 揭露 sk_{id*} → 查詢 III → 猜測
                 （II、III 禁查 id*；標準預言機）

證明:            Hyb₀ ─[Claim 1：Def 8 模擬安全]→ Hyb₁ ─[Claim 2：包成 otUE 對手]→ 2^{−n+t}
```

三個弱點都已在既有報告裡出現，但從來沒有被放在一起看：

| # | 弱點 | 出處 | 根源 |
|---|---|---|---|
| **W-a** | 後量子實例只有 GKK25 Thm 3/14 取 LWE，**身分空間只有多項式大小**；指數身分空間要自造 LWE 版 RNC-IBE（乙案，高風險） | 路線圖 §4、v3.1 修訂三 | 槽位要求「交出 msk 的非承諾」，這是很強的性質 |
| **W-b** | **A 在挑戰後、分裂前不能查詢**（v3.1 Remark 2）；若要允許，需要 poly-ID 或更強的槽位介面 | v3.1 Remark 2、延後引理報告 §4 | Def 8 的模擬器在挑戰後不回答查詢；wrapper 在分裂前算不出 m̃sk |
| **W-c** | 定義揭露的是 sk_{id\*}（修訂四），槽位卻被要求交出**整把 msk** | v3.1 Remark 3、修訂四 | 修訂四改了定義，但沒有回頭檢查「槽位還需不需要這麼強」 |

**本文的切入點就是 W-c**：一旦揭露物只是 sk_{id\*}，槽位需要的只是「對**使用者金鑰**非承諾」。而 KT18 的 RSO 構造，恰恰是從最弱的假設（任意 IND-ID-CPA IBE）得到這個性質的標準方法。W-a、W-b 隨之一起解掉（§3）。

---

## 2. KT18 精讀：兩個機制，一個骨架

### 2.1 KDM 機制（KT18 §4，Theorem 5）

**構造 KdmIBE**（零件：KDM-SKE `(G, E, D)`、IND-ID-CPA IBE（身分空間 ID × [len_K] × {0,1}）、garbled circuit）：

```
Kdm.KG(MSK, id):   K_id ← G(1^λ);  sk_{id,j} ← KG(MSK, (id, j, K_id[j]))  (j ∈ [len_K])
                   Kdm.sk_id := (K_id, {sk_{id,j}}_j)            ← 只有「在位元」的 IBE 金鑰
Kdm.Enc(PP, id, m): (Ẽ, {lab_{j,α}}) ← Garble(E(·, m; r_E))       ← 把「SKE 加密電路」garble，m 寫死
                   CT_{j,α} ← Enc(PP, (id, j, α), lab_{j,α})      ← 每個 label 用各自的身分加密
Kdm.Dec:           用在位元的金鑰解出 {lab_{j,K[j]}} → Eval 得 E(K_id, m) → 用 K_id 解 SKE
```

**證明鏈（三步）**：Game 0 →（IND-ID-CPA，**只對「離位元」身分 (id, j, 1−K[j])**）→ Game 1：離位元 label 換成在位元 label →（GC 模擬）→ Game 2：GC 只由輸出 `E(K_id, m_b)` 模擬 →（歸約到 SKE 的 KDM 安全）。

兩個關鍵：
- **為什麼 IND-ID-CPA 就夠**：user key 只含在位元的 IBE 金鑰；KDM 函數 f(sk) 需要的也只有這些，歸約者向 IBE 挑戰者查得到。證明只對「離位元」身分用 IBE 安全——它們的金鑰**永遠不在任何人的 user key 裡**。
- **函數轉換**：A 查詢的 f 吃的是 (K, 在位元 IBE 金鑰)，歸約者要把它改寫成只吃 K 的 g。KT18 用 `sel_{k,j}(γ) = γ·(sk_{·,j,1} ⊕ sk_{·,j,0}) ⊕ sk_{·,j,0}` 把在位元金鑰寫成 K 位元的投影，證明 projection 與有界電路類都保持。
- **尺寸不依賴使用者數**（KT18 Remark 1）：SKE 的實例大小不能依賴 KDM 遊戲中的使用者數，否則只能支援 a-priori 有界個挑戰身分。ACPS09（LWE）、BHHO08（DDH）滿足。

**KT18 自己的警告**（§1.2）：「直接用 hybrid encryption 從 KDM-SKE 做 KDM-PKE 會遇到金鑰的死結（dead-lock）」——這句話在 §4.1 會變成我們的陷阱 (ii)。

### 2.2 RSO 機制（KT18 §5，Theorem 6）

**構造 RsoIBE**（只需 IND-ID-CPA IBE，身分空間 ID × {0,1}，1 位元訊息）：

```
Rso.KG(MSK, id):    r ← {0,1};  Rso.sk_id := (r, sk_{id,r}),  sk_{id,r} ← KG(MSK, (id, r))
Rso.Enc(PP, id, m): CT_α ← Enc(PP, (id, α), m)  for α ∈ {0,1}      ← 同一個 m 加密兩次
Rso.Dec:            用 sk_{id,r} 解 CT_r
```

**非承諾性**：KT18 Game 0 → 1 先把 CT_{1−r} 改成加密 1 − m（IND-ID-CPA，只對 (id, 1−r)——這把金鑰永遠不會被交出）；Game 1 → 2 再把 user key 裡的位元換成 r ⊕ m（r 均勻 ⇒ r ⊕ m 也均勻，分布不變）。換完之後，不論 m 為何，都是「(id, r) 那半加密 0、(id, 1−r) 那半加密 1」——密文與 m 無關；要打開成 m 時交出 `(r ⊕ m, sk_{id, r⊕m})` 即可。

**為什麼 IBE 版反而比 PKE 版簡單**（KT18 §1.3「Secret key vs random coins」）：IBE 的 user key 由金鑰機構產生，被腐化的使用者**手上根本沒有 KeyGen 的隨機性**，所以 RSO 只需要考慮揭露 sk 本身；PKE 若要連 KeyGen 隨機性一起揭露，就需要 key-simulatable 之類的代數結構。

> **這句話對我們的意義**：UIBE 的揭露階段正是「金鑰機構交出 sk_{id\*}」，不交任何隨機性（v3.1 Def 2）。所以 UIBE 的揭露**就是一次 receiver opening**，而且落在 KT18 說「簡單」的那一側。

### 2.3 共同骨架：KT18-lifting

KT18 §1.3 明說：「上述構造似乎可以把 SKE 的強安全概念搬到 IBE——凡是證明時需要以某種方式模擬 secret key 的概念。……例如，若底層 SKE 是非承諾的（如一次性密碼本），得到的 IBE 似乎就有 SIM-RSO 安全。但這個構造是多餘的，雙重加密就夠了。」

所以兩個機制是**同一個 lifting 填入不同 SKE**：

| 填入的 SKE | 搬到 IBE 的性質 | 化簡 | 在本文的用途 |
|---|---|---|---|
| 一次性密碼本（非承諾） | 對使用者金鑰非承諾（⇒ SIM-RSO；KT18 原文用 "seems"，本文 §3 直接用雙重加密，不依賴這一點） | 可化簡成**雙重加密**（不用 GC） | **k 的槽**（§3） |
| KDM-SKE | KDM | 無（GC 是打破死結的關鍵） | **遮罩訊息的槽**（§4） |

這張表是本文的地圖：**不可複製性需要非承諾（給 k），KDM 需要 KDM（給訊息）——兩者放在同一個 SKE 上時，在本文的證明策略下互相衝突（§4.1(iii)），所以用兩個槽。**

### 2.4 對照 UIBE 證明實際消耗的槽位介面

把精讀報告的 E1–E4 與 v3.1 後來發現的兩條介面（reveal-computability、挑戰後金鑰）一起列：

| 需求 | 證明中的位置 | GKK25 Def 8（Construction 1） | KT18 雙重加密（Construction 2） | KT18 KdmIBE |
|---|---|---|---|---|
| **E1** 模糊化不可區分（對 QPT） | v3.1 Claim 1／本文 Claim R1 | 是（real/sim 兩世界） | 是（κ 個 IND-ID-CPA hybrid，只對離位元身分） | 僅當 SKE 非承諾；填 KDM-SKE 時**否** |
| **E2** 不知 k 就能生密文 | wrapper 在分裂前產生挑戰密文 | 是（Sim₃） | 是（兩半分別加密 r_j、¬r_j） | — |
| **E3** 事後解釋：確定性、只吃可複製的古典狀態 | B̃、C̃ 兩側各算一次 | 是（Sim₄(st₃, k̄; r)） | 是（s\* := r ⊕ k，兩把候選金鑰預先產生） | — |
| **E4** 後量子 | 否則 Shor 直接攻擊（2026-08 meeting §1.4） | **僅 Thm 3 取 LWE，poly-ID** | **任意 pq IBE：LWE，指數 ID** | LWE（IBE＋ACPS09＋GC） |
| reveal-computability（修訂二） | B̃ 只拿到 k | 需 OTP 墊層 c₂ | **直接**：s\* = r ⊕ k，不需 c₂ | — |
| 挑戰後、分裂前的金鑰（Remark 2） | Ã 在分裂前答詢 | **僅 poly-ID**（挑戰前以 Sim₂ 預答，即被刪掉的「凍結」） | **是，任意 d**（真 MSK 在 Ã 手上） | — |
| 揭露物 | Def 2 揭露階段 | msk（≥ sk_{id\*}） | **只到 sk_{id\*}** | — |
| 模擬器 | — | Sim₁–Sim₄（Sim₂ 帶狀態） | **無**（真 Setup） | — |

**結論**：對現行定義（揭露 sk_{id\*}），KT18 雙重加密在每一欄都不輸 GKK25，並在 E4、Remark 2 兩欄嚴格勝出；唯一的損失是「揭露 msk」那一欄。

---

## 3. 路線 R：RSO 機制直接當 k 的槽

### 3.1 關鍵觀察

v3.1 Claim 2 裡，B̃／C̃ 在分裂後要做的只有兩件事：(1) 算出**同一把** sk_{id\*} 交給 B／C；(2) 回答 id ≠ id\* 的查詢。v3.1 用 m̃sk 同時做這兩件事，所以需要 GKK25 的 Sim₄ 輸出整把 msk。但若槽位能直接「事後解釋 sk_{id\*}」，而非挑戰身分的金鑰與 k 無關，那麼 (1) 只需要使用者金鑰層級的非承諾、(2) 只需要一把真的 MSK。

KT18 雙重加密正是如此：挑戰身分的金鑰由選擇字串 s\* 決定，s\* 可以事後設成 r ⊕ k；其他所有金鑰都用真 MSK 產生，與 k 無關。

**文獻上的同構**：GKK25 §1 對 incompressible IBE 的描述——「regular 版：壓縮後收到 **sk_{id\*}**；strong 版：收到**整把 msk**。Goyal 等人（ITCS 2025）已給出 regular 版的構造，strong 版在 GKK25 之前沒有構造」——與我們的處境一模一樣：

| | 揭露 sk_{id\*} | 揭露 msk |
|---|---|---|
| incompressible IBE | regular 版，GKRV25 已有構造 | strong 版，GKK25 經 RNC-IBE 首次做出 |
| unclonable IBE（本研究） | **Construction 2：任意 IBE 即可** | Construction 1：用 GKK25 的 RNC-IB-KEM（目前唯一有證明的途徑） |

### 3.2 Construction 2

零件：IND-ID-CPA IBE `(Setup, KG, Enc, Dec)`，身分空間 ID × [κ] × {0,1}，訊息 1 位元；otUE（金鑰 κ 位元，分布任意）。

```
Setup(1^λ, 1^d):  (pp, MSK) ← IBE.Setup;  mpk := pp,  msk := MSK
KeyGen(MSK, id):  s ← {0,1}^κ;  sk_{id,j} ← IBE.KG(MSK, (id, j, s_j))   (j ∈ [κ])
                  sk_id := (s, {sk_{id,j}}_j)
Enc(pp, id, m):   k ← otUE.Setup(1^λ)
                  CT_{j,α} ← IBE.Enc(pp, (id, j, α), k_j)      (j ∈ [κ], α ∈ {0,1})   ← 2κ 個古典密文
                  ρ ← otUE.Enc(k, m)                                                  ← 量子部分
                  ct := ({CT_{j,α}}, ρ)
Dec(sk_id, ct):   k'_j := IBE.Dec(sk_{id,j}, CT_{j,s_j});  m' := otUE.Dec(k', ρ)
```

注意：**不需要 OTP 墊層 c₂**。雙重加密本身就是逐位元的一次性密碼本（s\* = r ⊕ k），「k 本身＋分裂前的古典資料 r」就能算出揭露物——修訂二的 reveal-computability 自動滿足，對 otUE 的金鑰分布同樣不作任何假設。

### 3.3 定理

先把定義加強一格（本路線對任意身分空間做得到；Construction 1 只在 poly-ID 下能以 Sim₂ 預答做到）：

> **Definition 2⁺**：Definition 2 再加一段「**查詢階段 I′**（挑戰後、分裂前）：A 可繼續查詢 id ≠ id\*」。其餘不變（II、III 仍禁查 id\*，揭露 sk_{id\*}）。Definition 2⁺ 蘊含 Definition 2。

> **Theorem 3（搜尋型）**：設 IBE 對 QPT 對手為 IND-ID-CPA（adaptive-ID）安全，otUE 為 t-unclonable。則對任意身分長度 d，Construction 2 滿足 Definition 2⁺ 的 t-unclonable 安全：
>
> `Pr[win] ≤ 2^{−n+t} + 2κ · Adv^{ind-id-cpa}_{IBE}(λ) + negl(λ)`（Adv := |Pr[β′=β] − 1/2|）。
>
> **Theorem 3′（不可區分型）**：同上前提、otUE 為 unclonable-IND ⇒ Construction 2 為 unclonable-IND（Definition 3 的 2⁺ 版），界 `1/2 + 2κ·Adv + negl`。

### 3.4 證明骨架（完整版見 LaTeX 稿 §2）

```
H₀     真實遊戲（Definition 2⁺）
 │  完全相同：把揭露金鑰的選擇字串 s* 提前到挑戰時抽（它在揭露前與一切獨立）
H₀'
 │  Claim R1ᵢ（i = 1..κ）：|p_{1,i−1} − p_{1,i}| = 2·Adv^{ind-id-cpa}
 │  第 i 位元的「離位元」密文 CT_{i,1−s*_i} 由加密 k_i 改成加密 1 − k_i
H₁ = H_{1,κ}   此時 CT_{j,α} 加密 k_j ⊕ s*_j ⊕ α
 │  完全相同（Lemma：逐位元 OTP 重參數化，即 v3.1 Lemma 1）：
 │  先抽均勻 r，令 CT_{j,α} 加密 r_j ⊕ α；揭露時才令 s* := r ⊕ k
H₂     古典部分與 k 無關；k 只出現在 ρ 與「揭露時的選擇字串」
 │  Claim R2：p₂ ≤ 2^{−n+t} + negl（包成 otUE 對手）
2^{−n+t}
```

**Claim R1ᵢ 的歸約 Rᵢ**（IND-ID-CPA 對手、UIBE 挑戰者；單一 QPT 機器）：
- 所有金鑰查詢（查詢 I、I′、II、III，皆 id ≠ id\*）：Rᵢ 自抽 s，向 IBE 挑戰者查 `(id, j, s_j)`。
- 挑戰：Rᵢ 抽 s\*、k、m；j ≠ i 的 2(κ−1) 個密文與 `CT_{i,s*_i}` 自己加密（j < i 的離位元加密 1−k_j）；**`CT_{i,1−s*_i}` 是 IBE 挑戰**，挑戰身分 `(id*, i, 1−s*_i)`、訊息對 `(k_i, 1−k_i)`。
- 揭露：向 IBE 挑戰者查 `(id*, j, s*_j)`（j ∈ [κ]），同一把交給 B、C。
- 合法性：Rᵢ 查過的身分只有 `(id, ·, ·)`（id ≠ id\*）與 `(id*, j, s*_j)`——**從不查 `(id*, i, 1−s*_i)`**。IND-ID-CPA 允許挑戰後查詢，所以查詢 I′、II、III 都能轉送。

**Claim R2 的 wrapper**（沒有任何模擬器）：

| 角色 | 做什麼 |
|---|---|
| Ã（Phase 1） | 自己跑 `IBE.Setup` 得 (pp, MSK)；用真 KeyGen 回答查詢 I、I′；挑戰時抽 r，令 `CT_{j,α} ← Enc((id*, j, α), r_j ⊕ α)`；**預先產生兩把候選金鑰** `K_{j,b} := IBE.KG(MSK, (id*, j, b))`（j ∈ [κ]、b ∈ {0,1}）；從 otUE 挑戰者收 ρ；跑 A 的分裂；附掛 `cls := (pp, MSK, id*, r, {K_{j,b}})` 到兩側 |
| B̃（Phase 2，一開始就收到 k） | `s* := r ⊕ k`；`sk_{id*} := (s*, {K_{j,s*_j}})`；執行 B：查詢 II、III 以新鮮隨機性 `KeyGen(MSK, ·)` 回答，揭露時交 sk_{id\*}；輸出 m_B |
| C̃ | 對稱；同一份 cls、同一個 k ⇒ **同一把** sk_{id\*} |

視野與 H₂ 同分布（揭露金鑰在 H₂ 裡是以新鮮隨機性產生的 `KG(MSK, (id*, j, s*_j))`，這裡是預先以獨立隨機性產生、再依 s\* 挑選——同分布）；勝利事件相同 ⇒ `p₂ = Pr[wrapper 贏 otUE 遊戲] ≤ 2^{−n+t} + negl`。

**誰在回答查詢**（對照延後引理報告 §2 的表）：

| 遊戲 | 分裂後答詢與揭露的一方 | 手上持有 |
|---|---|---|
| H₀、H₂ | 挑戰者 | 真 MSK |
| Claim R1ᵢ 的 Rᵢ | Rᵢ（轉送 IBE 金鑰預言機） | 無 MSK，但所有需要的身分都不是 IBE 挑戰身分 |
| Claim R2 的 B̃、C̃ | B̃、C̃ 各自 | 真 MSK（附掛在 cls）＋ Phase 2 收到的 k |

### 3.5 代價與限制（要誠實寫進論文）

1. **揭露 msk 的強化版（v3.1 Remark 3）不在此證明範圍**：拿到 MSK 就能產生兩半的金鑰，假密文（兩半加密不同位元）一眼可辨，事後解釋失效。Construction 2 在 msk 揭露下是否仍不可複製：**沒有證明、也沒有攻擊**。若論文要保留 msk 版，就用 Construction 1（GKK25）當它的實例。
2. **分裂後禁查 id\* 是必要的**（對本證明而言）：若 B 在揭露前查到一把新鮮的 sk_{id\*}（新的選擇字串 s′），在假世界裡它會解出 `r_j ⊕ s′_j`，與真 k 不一致——可分辨，且 Rᵢ 也拿不到 `(id*, i, 1−s*_i)` 的金鑰。v3.1 本來就禁查，所以沒問題。若日後採用 09-13 的「分裂後不設限」提案：把 KeyGen 以 PRF 去隨機化（KT18 footnote 6 的做法），重複查詢回同一把；證明先多一個「PRF → 真隨機函數」的 hybrid，再把 RF(id\*) 程式化成 r ⊕ k，wrapper 對其他身分用放在 cls 裡的 2Q-wise independent 函數（古典查詢下與 RF 同分布）。**注意不能把 PRF 金鑰本身放進 cls**，否則 s\* 在分裂前就被決定（LaTeX 稿 Remark「分裂後禁查 id\* 的角色」有完整寫法）。
3. **損失因子 κ**：κ 個 IND-ID-CPA hybrid（κ = otUE 金鑰長，BL20 為 2n），多項式、無礙。
4. **密文尺寸**：2κ 個 1 位元 IBE 密文 ＋ ρ，**與身分數無關**（GKK25 Thm 3 的密文含 T 個 garbled circuit，尺寸 poly(T, |m|, λ)）。
5. **selective-ID 的 IBE ⇒ selective-ID 的 UIBE**（Rᵢ 需在開場知道挑戰身分 `(id*, i, 1−s*_i)`，而 s\* 可在開場就抽）。本證明要得到 adaptive UIBE，用的是 adaptive IBE。

### 3.6 對論文主軸的影響

| | 現況（v3.1，只有 Construction 1） | 加入 Construction 2 之後 |
|---|---|---|
| 現行定義（揭露 sk_{id\*}）的後量子實例 | GKK25 Thm 3 取 LWE，**poly-ID** | **任意標準模型 LWE adaptive IBE，指數 ID**（ABB10 adaptive 版、CHKP10、Yam17）；QROM 的 GPV08＋Zha12 需另行處理隨機預言機在 cloning 遊戲中的模擬 |
| 甲案／乙案（路線圖 §4.3） | 待拍板 | **對現行定義而言消失**；乙案（自造 pq RNC-IBE）只剩「msk 版指數 ID」這個用途 |
| Remark 2 時間邊界 | 必要（poly-ID 時可用 Sim₂ 預答繞過） | **可移除，任意 d**（Definition 2⁺） |
| msk 強化版 | 有（Remark 3） | 仍只由 Construction 1 提供 |
| 證明工具 | GKK25 Def 8 四模擬器 | 只有 IND-ID-CPA ＋ OTP 重參數化 |

**建議的論文呈現**：主定理維持「槽位 ＋ otUE ⇒ UIBE」的形狀，給兩個實例化定理——
- **Theorem A**（Construction 2，本文 Theorem 3）：現行定義、任意 pq IBE、指數 ID、允許挑戰後查詢；
- **Theorem B**（Construction 1，v3.1 主定理）：揭露 msk 的強化版、GKK25 RNC-IB-KEM、目前 pq 只到 poly-ID。

進一步可以抽象出「**使用者金鑰層級 RNC**」介面（事後解釋 sk_{id\*}、挑戰後的金鑰由可複製狀態無記憶地產生），讓兩個構造都成為它的實例——留作論文定義章的整理工作。

**競速提醒**：Construction 2 的想法很短。Kitagawa 同時是 KT18、GKK25、HKNY24、KN23 的作者——最自然的競爭者就在這條產線上。「unclonable IBE」一詞截至今日（2026-09-28）的網路檢索仍無結果，但**建議投稿前照路線圖 §5 的紀律重做 ePrint 全文檢索**，並考慮盡早把「定義＋Theorem A」掛 ePrint。

### 3.7 RSO 延伸：真的把「receiver selective opening」帶進不可複製遊戲

**(a) 多目標＋選擇性開啟的不可複製性**

把「密文發出去之後，部分接收者**才**被腐化」的 RSO 情境放進 cloning 遊戲，定義如下（搜尋型）：

```
Expt^{so-cloning}:
  建置；A 可隨時查詢非挑戰身分
  挑戰：A 輸出 q 個相異、未查詢過的身分 id^(1..q)；挑戰者獨立均勻抽 m^(i)，送出 ρ^(i) ← Enc(mpk, id^(i), m^(i))
  開啟：A 適應性地選 i，收到 sk_{id^(i)}（每個 i 一次）
  目標：A 選一個未開啟的 i*
  分裂；B、C 查詢非挑戰身分；揭露 sk_{id^(i*)}（同一把）；猜測
  獲勝 ⟺ m_B = m_C = m^(i*)
```

> **Theorem 4**：Construction 2 滿足 `Pr[win] ≤ q · (2^{−n+t} + 2κ·Adv^{ibe} + negl)`。

證明：開場均勻猜 g ∈ [q]，令 E = {i\* = g}，則 `Pr[win ∧ E] = Pr[win]/q`。之後只把第 g 個密文做成假的（κ 個 hybrid；若 A 開啟 g 則 ¬E，歸約直接輸出 0），wrapper 把 otUE 挑戰嵌在第 g 位，其他 q−1 個密文與所有開啟都用真 MSK 產生（A 要求開啟 g 時 E 已不成立，wrapper 放棄即可）。

與 Construction 1 的比較要精確：**「開啟」本身對 Construction 1 不是障礙**——被開啟的都是挑戰時就公布的身分，v3.1 Claim 1 的歸約可以在送出 Def 8 挑戰之前先查它們，v3.1 Claim 2 的 Ã 可以在 Sim₃ 之前先用 Sim₂ 答它們。真正擋住 Construction 1 的，是定義同時允許「挑戰後、分裂前對**其他**非挑戰身分的任意查詢」；這在 poly-ID 時仍可用 Sim₂ 預答，超多項式身分空間則不行。所以 Construction 2 的優勢是「**任意身分空間、不需預答**」。

損失因子 q 來自「只有一個 otUE 挑戰可嵌」；不可區分型（每個密文各自獨立的挑戰位元 b_i）用「猜錯時 B̃、C̃ 輸出同一個共享隨機位元」可得 `1/2 + q·(O(κ)·Adv + negl)`。

**(b) 同時達到 KT18 的 SIM-RSO 機密性：Construction 2′（遮罩變體）**

Construction 2 的量子部分直接裝 m，模擬器在不知道 m 時產生不了 ρ——除非 otUE 本身可事後解釋（BL20 可以：依古典／量子盤點報告 §1.1 的形式 ρ = H^θ|x ⊕ m⟩、k = (x, θ)，先備好 H^θ|w⟩、事後令 x := w ⊕ m 即可；一般 otUE 不行）。一般化的解法是把訊息移出量子層：

```
Construction 2′:  y ← {0,1}^n;  z := y ⊕ m
                  ct := ( 雙重加密 (k ∥ z)（κ + n 位元）,  ρ ← otUE.Enc(k, y) )
                  Dec：由槽位得 (k, z)，y := otUE.Dec(k, ρ)，m := z ⊕ y
```

> **Proposition 1**：若 IBE 為 IND-ID-CPA（對 QPT），Construction 2′ 滿足 KT18 Def 11 的 SIM-RSO（adaptive-ID、挑戰身分數無先驗上界；對手為 QPT、密文為量子態），且 Theorem 3、4 對 2′ 同樣成立（hybrid 數由 κ 變成 κ + n）。

SIM-RSO 的證明與 KT18 Thm 6 同構：把所有挑戰密文的離位元改成互補位元 → 重參數化 → 此時未開啟密文的古典部分與 (k, z) 無關，而 ρ 只裝與 m 無關的 y。模擬器 S 自己跑 Setup、以隨機 k、y、r 產生全部密文，收到開啟集合的 m^(i) 後令 `z^(i) := y^(i) ⊕ m^(i)`、`s^(i) := r^(i) ⊕ (k^(i) ∥ z^(i))`。**完全不需要 otUE 的任何安全性**。

不可複製性：揭露後兩人都得到 (k, z)，要贏就得都拿到 y——正是 otUE 的遊戲（wrapper 自抽均勻 z、隱式令 m := z ⊕ y；z 那 n 位可誠實加密，所以仍只需 κ 個 hybrid）。SIM-RSO 部分的模擬器 S 要執行量子對手、產生量子密文，因此是 QPT（KT18 原定義是 PPT）。

**(c) 沒做、也不建議現在做的**：KT18 的 SIM-RSO 允許**相關訊息分布**。放進 cloning game 會碰到本質問題：若 Dist 讓某個被開啟的訊息等於目標訊息，A 在分裂前就知道目標訊息、直接複製古典字串；界必須相對於「已開啟資訊下目標訊息的條件最小熵」，而 otUE 的 t-unclonability 只對均勻訊息陳述，歸約嵌不進去。需要 simulation-based 的不可複製定義，列為 open。

---

## 4. 路線 K：KDM 機制與不可複製加密

### 4.1 KDM 放進 cloning game 的三個陷阱

**(i) 搜尋型遇到依賴 sk_{id\*} 的訊息會平凡可破。** 若挑戰訊息 m = f(sk_{id\*})，揭露後 B、C 都拿到 sk_{id\*}，兩人各自計算 f(sk_{id\*}) 就同時「解出」訊息——根本不需要 ρ。所以**只有不可區分型**能談「挑戰訊息依賴目標金鑰」；搜尋型只能搭配與目標金鑰無關的挑戰訊息。

**(ii) UIBE 是 KEM–DEM，KT18 §1.2 的死結原封不動出現。** 以 Construction 2 加密 f(sk)：f(sk) 在 ρ 裡、ρ 的金鑰 k 在雙重加密裡、雙重加密用在位元金鑰解、在位元金鑰又可能在 f(sk) 裡。要證 KDM-CPA 就得把 ρ 換成加密 0，這需要 k 保密，這需要在位元的 IBE 密文保密，而歸約者為了算 f(sk) 必須持有在位元金鑰——以標準 hybrid 證不出來（KT18 原文也只說 "seems difficult"；這是證明障礙，並沒有對應的攻擊）。KT18 的解法是讓 key-dependent 的內容**只經過 KDM-SKE 那一層**。

**(iii) 被事後模糊化的金鑰材料，不能出現在分裂前就得產生的 KDM 訊息裡。** Construction 2 的 wrapper 在分裂前不知道 k，所以目標金鑰的選擇字串 s\* = r ⊕ k 在分裂前**尚未決定**。任何「分裂前必須由 Ã 產生、且內容依賴 s\*」的密文，Ã 都產生不出來。這是本 wrapper 的限制（事後才決定的金鑰材料，不能出現在它決定之前就得產生的內容裡），不是不可能性結果；它意味著「同一個 SKE 同時 KDM 又非承諾」在我們的證明策略下行不通。

### 4.2 設計原則（從三個陷阱反推）

- **量子層只裝一個均勻遮罩 y**，與訊息、與金鑰都無關 ⇒ 死結 (ii) 不會碰到量子層。
- **所有依賴訊息（因而可能依賴金鑰）的內容走 KDM 槽**：遮罩後的 z = y ⊕ m 用 KT18 KdmIBE 加密。
- **otUE 金鑰 k 走非承諾槽**（KT18 雙重加密），它與金鑰無關，所以 (iii) 只要求「輔助密文在目標被模糊化之前先被中和」——而輔助密文的接收者金鑰永不揭露，KDM 安全正好能中和它們。

### 4.3 Construction 3（雙槽）

零件：IBE₁（雙重加密用，身分空間 ID × [κ] × {0,1}）、KdmIBE（KT18 §4，建在 IBE₂、KDM-SKE、GC 上）、otUE。兩個 IBE 用獨立實例（證明較乾淨）。

```
Setup:            (pp₁, MSK₁) ← IBE₁.Setup;  (pp₂, MSK₂) ← KdmIBE.Setup
KeyGen(msk, id):  sk⁽¹⁾_id := (s, {IBE₁.KG(MSK₁, (id, j, s_j))}_j),  s ← {0,1}^κ
                  sk⁽²⁾_id := KdmIBE.KG(MSK₂, id; PRF(K_prf, id)) = (K_id, {在位元 IBE₂ 金鑰})   ← 確定性（見 §4.4 註）
                  sk_id := (sk⁽¹⁾_id, sk⁽²⁾_id)
Enc(mpk, id, m):  k ← otUE.Setup;  y ← {0,1}^n
                  CT⁽¹⁾_{j,α} ← IBE₁.Enc(pp₁, (id, j, α), k_j)       ← RSO 槽：裝 k
                  ct⁽²⁾ ← KdmIBE.Enc(pp₂, id, y ⊕ m)                  ← KDM 槽：裝 z = y ⊕ m
                  ρ ← otUE.Enc(k, y)                                  ← 量子層：只裝遮罩 y
Dec(sk_id, ct):   k' ← 槽 1；y' ← otUE.Dec(k', ρ)；z' ← KdmIBE.Dec(sk⁽²⁾, ct⁽²⁾)；m' := z' ⊕ y'
```

分工一目了然：**揭露 sk_{id\*} 之後兩人都得到 k 與 z**，要贏必須都拿到 y——不可複製性全在 ρ；**KDM 內容只存在於 ct⁽²⁾**——KDM 安全全在 KdmIBE。

### 4.4 兩個安全定義

**Definition K1（UIBE 的 KDM-CPA）**：KT18 Def 10 原樣，改為 QPT 對手、量子密文。登錄查詢（金鑰產生但不給）、擷取查詢、KDM 查詢 (id ∈ L_ch, f ∈ F) 回傳 `Enc(mpk, id, f(sk))`（b = 1）或 `Enc(mpk, id, 0^{|f|})`（b = 0）。f 吃的是**完整** user key（兩個槽的金鑰都算）。

**Definition K2（KDM-unclonable，搜尋型／不可區分型）**：

```
建置；A 在分裂前可做三種查詢（KT18 的形狀）：
  擷取  id ∉ L_ch                   → sk_id
  登錄  id ∉ L_ext ∪ L_ch           → 產生 sk_id 但不給；加入 L_ch
  KDM   (id ∈ L_ch, f ∈ F)          → ρ ← Enc(mpk, id, f(sk_{L_ch}))   ← 永遠是「真的」，沒有挑戰位元；加入 L_rcv
挑戰：A 輸出 id* ∈ L_ch \ L_rcv（不可區分型另給 (m₀, m₁)，與金鑰無關）；
      之後不得再以 id* 為 KDM 接收者
分裂；B、C 可擷取 id ∉ L_ch；揭露 sk_{id*}（就是登錄時產生、被 f 用過的那把）；猜測
```

語義：系統裡流通著 key-dependent 密文（函數在 F 中）——包括**把目標金鑰 sk_{id\*} 加密給其他使用者**、金鑰循環——挑戰密文仍然不可複製。

註（重複擷取）：KT18 的 KDM 遊戲規定每個身分至多擷取一次，而我們沿用 v3.1 的標準預言機（可重複查詢）。所以令 KdmIBE.KG 以放在 msk 裡的後量子 PRF 去隨機化（KT18 footnote 6），重複擷取時 sk⁽²⁾ 相同、sk⁽¹⁾ 仍新鮮抽樣；歸約以查表維持一致。PRF 由後量子單向函數即得，不增加假設。若方案不是 KDM 安全的，這些輔助密文可能讓 A 在分裂前推出 sk_{id\*}，先解密再複製古典訊息。

兩條限制的理由：挑戰訊息與金鑰無關（陷阱 (i)(iii)）；目標不可當過 KDM 接收者（陷阱 (iii)：目標名下的 key-dependent 密文無法被中和，因為目標金鑰會被揭露）。

### 4.5 Theorem 5：KDM-CPA

> **Theorem 5**：設 F ∈ {P（projection）, B（有界電路）}。若 KdmIBE 對 QPT 對手為 F-KDM-CPA 安全，則 Construction 3 為 F-KDM-CPA 安全（Definition K1，對完整 user key）。依 KT18 Thm 5，前提可由「pq IND-ID-CPA IBE ＋ 尺寸不依賴使用者數的 pq F-KDM SKE（LWE：ACPS09；B 類再經 App11 放大）＋ pq GC」得到。

證明（兩步、只用 KdmIBE）：以 G_b 表挑戰位元固定為 b 的遊戲，G\* 為「所有 KDM 回應的 ct⁽²⁾ 都換成 `KdmIBE.Enc(id, 0^n)`」。
- G₁ ≈ G\*：歸約 R′ 自己跑 IBE₁、otUE，持有所有 sk⁽¹⁾；對每個 KDM 查詢 (id, f) 自抽 k、y，向 KdmIBE 的 KDM 預言機查 `g(x) := y ⊕ f(sk⁽¹⁾ 寫死, x)`。g 仍在 F 裡：projection 對「寫死常數、與常數 XOR」封閉；有界電路類只需把 KdmIBE 的大小上界預留「寫死常數＋一層 XOR」的量。
- G₀ ≈ G\*：同上但查常數函數 `g ≡ y`。
- G\* 與 b 無關（m 只出現在 z，而 z 已不存在；y 與 m 無關）。

**有趣的一點**：KDM-CPA 完全不用到 IBE₁ 與 otUE 的安全性；不可複製性（Theorem 6 無 KDM 查詢的特例）完全不用到 KdmIBE 的安全性。兩個性質落在不相交的零件上。

### 4.6 Theorem 6：KDM-unclonable

> **Theorem 6**：設 IBE₁ 對 QPT 為 IND-ID-CPA、KdmIBE 對 QPT 為 F′-KDM-CPA（F′ 為 F 經「寫死常數、與常數 XOR」後的類；F = P 時 F′ = P，F = B 時只是大小上界略增）、otUE 為 t-unclonable。則 Construction 3 滿足 Definition K2 的 KDM-t-unclonable：
>
> `Pr[win] ≤ 2^{−n+t} + 2q_reg · Adv^{kdm}_{KdmIBE} + 2κ · Adv^{ind-id-cpa}_{IBE₁} + negl(λ)`，
>
> 其中 q_reg 為登錄次數上界。不可區分型平行（界 1/2 + …）。

證明骨架（完整版見 LaTeX 稿 §4）：

```
H₀   真實 KDM-cloning 遊戲
 │   Step 1（KDM，損失 q_reg）：所有輔助 KDM 密文的 ct⁽²⁾ 換成 KdmIBE.Enc(id, 0^n)
H₁   輔助密文不再攜帶任何金鑰資訊
 │   Step 2（κ 個 IND-ID-CPA on IBE₁）：挑戰密文的槽 1 離位元改成互補位元
H₂
 │   Step 3（完全相同）：逐位元 OTP 重參數化，s* := r ⊕ k 延到揭露時
H₃
 │   Step 4：wrapper（Ã 自抽均勻 z 並產生 ct⁽²⁾* ← KdmIBE.Enc(id*, z)，隱式令 m := z ⊕ y；
 │           B̃、C̃ 收到 k 後算 s*、組 sk_{id*}，最後輸出 m_B ⊕ z 當作對 y 的猜測）
2^{−n+t}
```

**Step 1 為什麼要猜**：KDM 函數可以依賴 sk_{id\*}，而歸約者 R_A 在 KdmIBE 的 KDM 遊戲裡**不能**把 id\* 登錄（它的金鑰最後要揭露）。所以 R_A 開場猜哪一次登錄會成為 id\*，對那一次改做「擷取」（因此知道完整 sk_{id\*}，可寫死進 g），其餘照常登錄；猜錯（A 以猜中的身分為 KDM 接收者，或挑戰身分不是它）就輸出 0。猜對時模擬完美，且猜測與 A 的視野獨立 ⇒ 損失因子 q_reg。挑戰密文的 ct⁽²⁾\* 由 R_A 自己公開加密（它知道 z），揭露物 sk_{id\*} 它也知道。

**Step 4 的一個細節**（搜尋型）：Hyb₃ 裡 (y, m) 獨立均勻、z = y ⊕ m；wrapper 裡 y 由 otUE 挑戰者抽、z 由 Ã 均勻抽、m := z ⊕ y——兩者的 (y, z, m) 聯合分布相同。B 贏 ⟺ m_B = m ⟺ `m_B ⊕ z = y`，所以 B̃ 輸出 `m_B ⊕ z`。不可區分型則由 Ã 把 `(z ⊕ m₀, z ⊕ m₁)` 當成自己在 otUE 遊戲選的訊息對。

**順序是整個證明的關鍵**：Step 1 必須在「把 s\* 延到揭露時才決定」（Step 3、4）之前——Step 1 時一切都是真的（s\* 已固定、歸約者知道 sk_{id\*}），輔助密文可以依賴 s\*；Step 1 之後輔助密文已與金鑰無關，s\* 才能安心地變成「事後才決定」。這正是陷阱 (iii) 的解法。（Step 3 還用到一點：Definition K2 的 sk_{id\*} 是在登錄時產生的，但 Step 1 之後揭露前沒有任何東西讀它，所以可以等價地改在揭露時才抽。）

**模組性**：把槽 1 換成 GKK25 的 RNC-IB-KEM（＋OTP），Step 1 原封不動，Step 2–4 改用 v3.1 的 Claim 1、2（代價是定義要拿掉「挑戰後、分裂前的擷取」，也就是回到 v3.1 Remark 2 的邊界）——KDM 那一步不綁定在哪一個 k 槽上。

### 4.7 限制與 open problems

| 問題 | 現況 | 障礙 |
|---|---|---|
| **挑戰訊息依賴目標金鑰**（不可區分型：(f₀(sk), f₁(sk)) 依賴 sk_{id\*}） | Open | 若只依賴 sk⁽²⁾_{id\*} 其實證得出來（Ã 自己產生 sk⁽²⁾）；依賴槽 1 的 s\* 就撞上陷阱 (iii)。後者需要「對同一把事後解釋的金鑰同時 KDM」的新工具 |
| 挑戰訊息依賴**其他**登錄身分的金鑰，同時又有輔助 KDM 密文 | Open | Step 1 的歸約算不出挑戰訊息（那些金鑰在 KDM 遊戲裡被登錄）；沒有輔助密文時則不需要 KDM 安全（無循環） |
| 揭露 msk 的 KDM-unclonable | Open | Step 1 無法交出 KdmIBE 的主金鑰 |
| 相關訊息分布下的 SIM 型 RSO 不可複製性 | Open | 見 §3.7(c) |
| 效率 | — | 密文 = 2κ 個 IBE₁ 位元密文 ＋ 一個 KdmIBE 密文（一個 garbled SKE 加密電路 ＋ 2·len_K 個 IBE₂ 密文）＋ ρ |

**文獻定位**：今日的網路檢索找不到任何「KDM × unclonable」或「selective opening × unclonable」的工作（檢索有限，需照 §3.6 的紀律複查）。KDM 與不可複製性分別都有長串文獻；把兩者放進同一個遊戲、並指出「中和先於模糊化」的順序，可以是論文延伸章的核心。

---

## 5. 三個構造總對照

| | **Construction 1**（v3.1） | **Construction 2**（§3） | **Construction 3**（§4） |
|---|---|---|---|
| 古典槽 | GKK25 RNC-IB-KEM ＋ OTP c₂ | KT18 雙重加密（裝 k） | 雙重加密（裝 k）＋ KT18 KdmIBE（裝 z = y ⊕ m） |
| 量子層裝什麼 | m | m | 均勻遮罩 y |
| 假設 | pq RNC-IB-KEM | **pq IND-ID-CPA IBE** | pq IBE ×2 ＋ pq KDM-SKE ＋ pq GC |
| 後量子實例的身分空間 | poly（GKK25 Thm 3, LWE） | **指數**（LWE） | **指數**（LWE） |
| 揭露 sk_{id\*}（現行定義） | ✓ | ✓ | ✓ |
| 揭露 msk（Remark 3） | ✓ | 無證明 | 無證明 |
| 挑戰後、分裂前查詢（Def 2⁺） | 僅 poly-ID（Sim₂ 預答） | ✓ 任意 d | ✓ 任意 d |
| 多目標選擇性開啟（Thm 4） | 開啟本身可；連同任意 I′ 查詢僅 poly-ID | ✓ 任意 d（損失 q） | 應可（同證法，未寫出） |
| SIM-RSO 機密性 | 需 otUE 可解釋 | 需 otUE 可解釋；2′ 變體則任意 otUE | —（z 走 KDM 槽，非承諾性不足） |
| KDM-CPA | 無證明（死結） | 無證明（死結） | ✓（Thm 5） |
| KDM-unclonable | 無證明 | 無證明 | ✓（Thm 6，損失 q_reg） |
| 證明中的模擬器 | Sim₁–Sim₄ | 無 | 無（KdmIBE 內部用 GC 模擬器，被 Thm 5 封裝） |

---

## 6. 建議與待老師拍板的問題

**建議的優先序**：

1. **先拿 Construction 2 跟老師談**（影響最大、最確定）。它直接改變主定理的實例化故事：現行定義下的後量子 UIBE 不再受限於 poly-ID。
2. **KDM 線當「結合另一個機制」的主要研究題目**。Construction 3 與 Definition K2 是新東西；Theorem 5/6 的證明已完整寫出（LaTeX 稿 §4），但定義本身需要老師認可（見 Q3）。
3. **RSO 延伸（Thm 4、Prop 1）當 Construction 2 的附加成果**，放在同一章。

**待拍板的問題**：

- **Q1**：論文主定理是否改為「兩個實例化定理」（Theorem A = Construction 2、Theorem B = Construction 1 的 msk 版）？或抽象成「使用者金鑰層級 RNC」介面？
- **Q2**：定義是否採用 Definition 2⁺（允許 A 在挑戰後、分裂前查詢）？Construction 2/3 對任意身分空間支援；Construction 1 只能在 poly-ID 下以 Sim₂ 預答支援（也就是先前被刪掉的「凍結」手法）。
- **Q3**：KDM-unclonable 的定義形狀是否可接受——輔助密文可以 key-dependent（F 中的函數，可依賴目標金鑰），但**挑戰訊息與金鑰無關**、目標不可當過 KDM 接收者？還是老師希望直接挑戰「挑戰訊息依賴目標金鑰」這個 open problem？
- **Q4**：RSO 方向做到獨立訊息＋損失 q 是否足夠？相關訊息分布需要 simulation-based 的不可複製定義，工作量大一個量級。
- **Q5**：分裂後禁查 id\* 維持不變（Construction 2 需要它）；若要解除，同意以 PRF 去隨機化 KeyGen 的方式處理？

---

## 7. 下一步

1. **Meeting**：報告 §3.1 的觀察與 Construction 2；請老師回答 Q1–Q5。
2. **LaTeX**：本次已附 `W1_KT18結合_構造定理與證明_LaTeX_2026-09-28.tex`（Construction 2/2′/3、Definition 2⁺／SO-cloning／K1／K2、Theorem 3–6、Proposition 1 與證明；PDF 見 `Paper drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf`）。拍板後併入主稿。
3. **修訂三的延伸**（QPT 提升）：確認所選 LWE IBE 的 adaptive 證明為 straight-line（ABB10 adaptive 版、CHKP10 的 partitioning／artificial abort 不回捲對手）；若改走 QROM 的 GPV08＋Zha12，要另外處理 cloning 遊戲中 A、B、C 的量子隨機預言機存取與 wrapper 兩側的一致模擬。KdmIBE 三步（IND-ID-CPA、GC、KDM-SKE）皆為 straight-line，列一段附錄。
4. **定義章整理**：把「使用者金鑰層級 RNC」寫成介面定義，讓 Construction 1、2 都成為實例。
5. **新穎性複查**：unclonable × {IBE, KDM, selective opening, non-committing} 的 ePrint 全文檢索；追蹤引用 KT18、GKK25、HKNY24 的新論文。
6. **（可選）** 調研 Bartusek–Khurana（CRYPTO 2023）的 certified everlasting 技術能否繞過非承諾性，處理 Construction 2 的 msk 揭露版——這是 §3.5 第 1 點的唯一已知可能方向，尚未驗證。

---

## 8. 參考文獻

- **[KT18]** F. Kitagawa, K. Tanaka. *Key Dependent Message Security and Receiver Selective Opening Security for Identity-Based Encryption*. PKC 2018.（§4 KdmIBE／Thm 5；§5 RsoIBE／Thm 6；§1.3 lifting 與「只揭露 secret key」的討論；Remark 1 尺寸要求；Thm 4 從 IND-CPA PKE 得 SIM-RSO PKE）
- **[GKK25]** R. Goyal, F. Kitagawa, V. Koppula, R. Nishimaki, M. S. Rajasree, T. Yamakawa. *Non-Committing Identity Based Encryption: Constructions and Applications*. PKC 2025.（Def 8；Thm 3/14；§1 對 regular／strong incompressible IBE 的區分）
- **[GKRV25]** R. Goyal, V. Koppula, M. S. Rajasree, A. Verma. *Incompressible Functional Encryption*. ITCS 2025（ePrint 2024/798）。
- **[AK21]** P. Ananth, F. Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.
- **[HKNY24]** T. Hiroka, F. Kitagawa, R. Nishimaki, T. Yamakawa. *Robust Combiners and Universal Constructions for Quantum Cryptography*. TCC 2024, App. E.
- **[BL20]** A. Broadbent, S. Lord. *Uncloneable Quantum Encryption via Oracles*. TQC 2020.
- **[HPW15]** C. Hazay, A. Patra, B. Warinschi. *Selective Opening Security for Receivers*. ASIACRYPT 2015.
- **[CHK05]** R. Canetti, S. Halevi, J. Katz. *Adaptively-Secure, Non-Interactive Public-Key Encryption*. TCC 2005.
- **[ABB10]** S. Agrawal, D. Boneh, X. Boyen. *Efficient Lattice (H)IBE in the Standard Model*. EUROCRYPT 2010（摘要明言含 adaptively-secure IBE 的延伸）。
- **[CHKP10]** D. Cash, D. Hofheinz, E. Kiltz, C. Peikert. *Bonsai Trees, or How to Delegate a Lattice Basis*. EUROCRYPT 2010.
- **[Yam17]** S. Yamada. *Asymptotically Compact Adaptively Secure Lattice IBEs and Verifiable Random Functions via Generalized Partitioning Techniques*. CRYPTO 2017.
- **[GPV08]** C. Gentry, C. Peikert, V. Vaikuntanathan. *Trapdoors for Hard Lattices and New Cryptographic Constructions*. STOC 2008.
- **[Zha12]** M. Zhandry. *Secure Identity-Based Encryption in the Quantum Random Oracle Model*. CRYPTO 2012.
- **[ACPS09]** B. Applebaum, D. Cash, C. Peikert, A. Sahai. *Fast Cryptographic Primitives and Circular-Secure Encryption Based on Hard Learning Problems*. CRYPTO 2009.
- **[App11]** B. Applebaum. *Key-Dependent Message Security: Generic Amplification and Completeness*. EUROCRYPT 2011.
- **[BK23]** J. Bartusek, D. Khurana. *Cryptography with Certified Deletion*. CRYPTO 2023.（§7 第 6 點，僅列為待調研方向）

> **相關報告**：[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)｜[延後引理報告](./Deferred_Query_Lemma_Unnecessary_Who_Answers_Queries_Two_Reveals.md)（「誰在回答查詢」的分析方式，本文 §3.4 沿用）｜[古典／量子盤點報告](./Classical_vs_Quantum_Inventory_and_Challenge_Identity_Query_Rule_Evaluation.md)（分裂後禁查 id\* 的討論，本文 §3.5 第 2 點）｜[W1 路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)（§4 甲案／乙案，本文 §3.6 更新其前提）｜[Reports 導覽](./README.md)
