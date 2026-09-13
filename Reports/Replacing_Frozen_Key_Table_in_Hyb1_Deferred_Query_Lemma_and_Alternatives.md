# 換掉 Hyb₁ 的凍結表 K：分裂後查詢的「延後引理」、四條替代路線的評估與文獻對照

> **日期**：2026-09-12
> **議題來源**：老師對主定理討論稿（`papers/Proof_of_Main_Theory_in_UIBE.pdf`，Hyb₁ 第 3 步「先做凍結」處的畫線）的意見：在 Hyb₁ 裡維護整張金鑰表 K 不可行，會碰到指數大的東西。
> **本文回答的問題**：表 K 到底在證明裡扮演什麼角色、「指數」精確地出在哪、可以換成什麼、各方案代價為何、別人怎麼處理同類問題。
> **一句話結論**：表 K 可以**整個刪掉**，而且不需要改動 GKK25 的假設，也不需要改動老師已核可的定義——只要在證明開頭加一條「分裂後查詢可延後到揭露之後」的 WLOG 引理。刪掉之後，主定理不再依賴 T = poly(λ)，身分空間可以是指數大。

---

## 0. TL;DR

1. **表 K 的兩個功能**。它不只是「讓 B、C 分裂後能查到金鑰」；它還把所有 Sim₂ 的呼叫**壓進 KEM 遊戲的 pre-challenge 階段**——因為 GKK25 Def 8 的遊戲**沒有 post-challenge 查詢階段**（對手在挑戰階段拿到 msk 之後就不再有預言機）。任何「保留分裂後查詢、由挑戰者親自回答」的方案，都得同時處理這兩件事。
2. **「指數」出在哪**。凍結要對所有 id ∉ dom(K) ∪ {id\*} 逐一呼叫 Sim₂，共 |ID| − 1 次；表 K 也有 |ID| − 1 項。GKK25 的主構造（SXDH，Thm 12）與任何實用 IBE 的身分空間是 {0,1}ⁿ，|ID| = 2ⁿ——凍結不可行。即使退到 T = poly 的 relaxed 設定，也等於把主定理的成立條件綁在一個與不可複製性無關的參數上。
3. **拍板建議：延後引理（Lemma D）**。分裂後、揭露前的金鑰查詢，對手可以**自己**延後到拿到 msk 之後再做——揭露之後它有 msk，自己跑 KeyGen 就好，根本不需要挑戰者回答。這是一個對手端的 WLOG 改寫，**完全模擬**（不是計算不可區分），所以：(i) 定義不必改，Definition 2 原封不動；(ii) 證明只需處理「分裂後不查詢」的對手；(iii) Hyb₁ 不再有凍結、不再有表 K、不再有字典序；(iv) Claim 1 的歸約不再凍結；(v) Claim 2 的 cls 只剩 (st₃, r, c₂)；(vi) 主定理對**任意身分空間**成立，Setup 改為 GKK25 的 Setup(1^λ, 1ⁿ) 語法；(vii) 定義本身建議直接寫成 G₀（分裂前查詢＋揭露 msk，分裂後無互動），G₁ 與「只揭露 sk_{id\*}」的變體降為一則「被蘊含」的備註——形狀與 GKK25 Def 8 完全一致（§2.7）。
4. **定義稿 §3 對「先揭露再查詢」的判斷（「定義較弱」）需要更正**。延後引理證明它跟「揭露前查詢」**等價**：每個在揭露前查詢的對手，都有一個獲勝機率完全相同、只在揭露後自己算金鑰的對手。路線圖的 D-W1 出路 (ii)「reveal 給 msk ⇒ 分裂後查詢自足」**成立**，但成立的形式是「對手自足」而非「挑戰者用 m̃sk 回答」——後者在 Claim 2 的 wrapper 裡做不到（揭露前算不出 m̃sk），這正是先前繞去凍結的原因。
5. **修訂一（memoized 預言機）可以撤回**。它當初是為了讓 Claim 1 從表 K 回答重複查詢；表沒了，它的存在理由也沒了。改回標準的「每次查詢新鮮隨機性」預言機，延後引理是**完美**模擬；若堅持保留 memoized，延後引理仍成立，代價是多一個 PRF hybrid（§2.3）。
6. **其他方案都被支配**：無狀態模擬器介面（§3.B）救不了 Claim 1，且要改 GKK25 的定義並重證 Thm 12；PRF 懶惰表（§3.C）以 B 為前提；保留 T = poly（§3.D）正是老師反對的；分裂前承諾查詢集合（§3.E）與「只做分裂前查詢」（§3.F）現在都知道與現行定義等價或更弱，沒有必要。
7. **文獻上沒有人處理過「分裂後對帶狀態模擬器做適應性查詢」**：AKL23 的 cloning game 框架分裂後只有一問一答；MM24 的 unclonable FE 把函數金鑰的選擇放在分裂前、adaptive 版只留在 remark；QROM／Haar-ROM 系列分裂後可查的預言機是無狀態的；certified deletion 系列是單一對手。我們也不需要處理它——它可以被 WLOG 掉（§4）。

---

## 1. 問題的精確形狀

### 1.1 表 K 的來歷：D-W1

主定理的第二步（Claim 2）把整場 Hyb₁ 包成 otUE cloning 遊戲的一個對手 (Ã, B̃, C̃)。分裂線右邊的 B̃、C̃ 互不通訊，但各自要替內芯 B、C 扮演 Hyb₁ 的挑戰者——包括回答查詢階段 II 的金鑰查詢。Hyb₁ 裡回答查詢的是 Sim₂，而 GKK25 把 Sim₂ 定義成**帶狀態**的演算法（更新 st₂，st₂ 再流進 Sim₃、Sim₄）。兩條互不通訊的分支各跑一份 Sim₂，狀態就分岔；分岔之後 Sim₄ 在兩側算出的 m̃sk 也未必相同，而 Hyb₁ 要求兩側拿到**同一把** m̃sk。這是 D-W1。

2026-08-28 版的解法是**凍結**：A 宣告 id\* 之後、分裂之前，挑戰者依字典序對所有 id ∉ dom(K) ∪ {id\*} 呼叫 Sim₂ 補滿表 K；分裂後的查詢一律回表，Sim₂ 不再被呼叫，狀態在分裂前定案。

### 1.2 「指數」出在哪

| 面向 | 凍結的代價 | 後果 |
|---|---|---|
| 呼叫次數 | Sim₂ 被呼叫 \|ID\| − 1 次 | 身分空間為 {0,1}ⁿ 時是 2ⁿ 次；GKK25 的主構造（SXDH，Thm 12，id ∈ ℤ_p）與所有實用 IBE 都是這種情形 |
| 表的大小 | \|ID\| − 1 項，且要複製進 B̃、C̃ 兩側的 cls | 同上，指數大 |
| 定理陳述 | 證明用掉 T = poly(λ)（Hyb₁ 第 3 步明寫「此處用掉 T = poly(λ)」） | 主定理只對 relaxed 設定成立；不可複製性本身跟 T 無關，這個依賴是證明技術的產物 |
| 後量子實例 | GKK25 目前唯一的 pq 實例（Thm 3/14）本來就只有 poly-ID | 表面上「剛好夠用」，但把定理綁在零件的暫時性限制上，之後換零件（乙案：自造指數身分空間的 LWE 版 RNC-IBE）定理就得重證 |

### 1.3 一個先前沒有寫明的事實：凍結還有第二個功能

回頭看 Claim 1 的歸約 R：它是 GKK25 Def 8 遊戲的對手。**Def 8 的遊戲只有 pre-challenge 查詢階段**——挑戰階段回傳 (msk, ct\*, seskey) 之後就沒有預言機了（對手手上有 msk，不需要）。所以 R 在挑戰之後**根本沒有 Sim₂ 可以呼叫**；它能給 B、C 的金鑰只有兩種：挑戰前查到的（表 K），或用收到的 m̄sk 自己算的 KeyGen(m̄sk, ·)。

凍結把所有 Sim₂ 呼叫壓進 pre-challenge 階段，Claim 1 才能「挑戰後只回表」。這意味著：

> **任何保留「分裂後、揭露前由挑戰者回答查詢」的替代方案，都必須同時解決 (a) B̃、C̃ 兩側一致、(b) Claim 1 的 R 在挑戰後拿得到答案。** 只解決 (a) 的方案（例如「要求 Sim₂ 無狀態」）不夠，見 §3.B。

而「用 KeyGen(m̃sk, ·) 回答」在 Claim 1 沒問題（R 有 m̄sk），在 Claim 2 卻不行：wrapper 裡 m̃sk = Sim₄(st₃, c₂ ⊕ k; r) 要等 otUE 挑戰者亮出 k 才算得出來，而查詢階段 II 在揭露之前。這兩個約束一夾，「揭露前的查詢」在證明裡就沒有任何自然的回答來源——除非它根本不需要被回答。

---

## 2. 拍板建議：分裂後查詢的延後引理

### 2.1 兩個遊戲

- **G₁**：Definition 2 現行的實驗——查詢階段 II 在分裂與揭露之間，B、C 各自獨立查詢（≠ id\*），之後揭露整把 msk。
- **G₀**：刪去查詢階段 II 的實驗——分裂之後直接揭露 msk。

金鑰預言機採**標準約定**：每次查詢以新鮮隨機性執行 KeyGen(msk, id)（即撤回修訂一；memoized 的情形見 §2.3）。

**Lemma D（分裂後查詢可延後）.** 對任意 QPT 對手 (A, B, C)，存在 QPT 對手 (A, B′, C′)，使得

Pr[G₀(A, B′, C′) = 1] = Pr[G₁(A, B, C) = 1]。

不可區分型（Definition 3）同樣成立，證明逐字相同。

**證明.** B′ 收到暫存器 B 之後什麼都不做，等到揭露階段收到 msk。然後在內部執行 B：餵入暫存器 B；B 發出的每個查詢 id（≠ id\*，因為 G₁ 禁止）由 B′ 自己以新鮮隨機性計算 KeyGen(msk, id) 回答；B 進入揭露階段時交給它 msk；輸出 B 的輸出。C′ 對稱。

視野比對。在 G₁ 中，條件於 msk 與分裂後的聯合態，B 的查詢答案是 KeyGen(msk, id) 的獨立新鮮樣本，且與 C 那一側的答案獨立（挑戰者對每次呼叫都抽新鮮隨機性）；B′ 內部給出的答案分布相同、與 C′ 側同樣獨立。其餘所有輸入——mpk、查詢階段 I 的答案、挑戰密文、A 的分裂映射、揭露的 msk——兩個遊戲逐位元相同（G₀ 與 G₁ 的挑戰者在分裂之前的行為完全一樣）。勝利事件同為 m_B = m_C = m。∎

**推論.** G₀-安全 ⇒ G₁-安全（同一個界）；反向平凡（G₀ 的對手就是不查詢的 G₁ 對手）。**兩個定義等價。**

引理成立的關鍵只有一件事：**查詢階段 II 與揭露階段之間沒有任何其他互動**——B 在那段時間唯一能從外界拿到的東西就是金鑰，而金鑰的分布不依賴它何時被問。所以「先問再拿 msk」和「先拿 msk 再自己算」對 B 來說是同一件事。

### 2.2 為什麼這不是弱化定義

定義稿 §3 把「先揭露、再查詢」列為出路 2，但註記「定義較弱」；路線圖把 D-W1 出路 (ii) 寫成「用 m̃sk 本地自足產生」。兩處的直覺都是：把查詢挪到揭露之後，對手能做的事變少了。Lemma D 說明這個直覺不對：**揭露之前能問到的東西，揭露之後自己都算得出來，而中間沒有任何事件會因為「早知道」而改變**。等價是精確的（完美模擬），不是「安全性大致相當」。

同時它也修正了出路 (ii) 的形式：不是挑戰者用 m̃sk 回答分裂後的查詢（wrapper 在揭露前算不出 m̃sk），而是**對手自己**把查詢延後——延後之後連 m̃sk 都不必用來回答查詢，因為 G₀ 裡沒有查詢。

Definition 2 可以原封不動保留（老師已核可）；證明第 0 步引用 Lemma D 之後，接下來只處理 G₀ 的對手。若之後想簡化定義，也可以直接把查詢階段 II 刪掉，兩者等價。

### 2.3 與修訂一（memoized）的交互作用

修訂一的唯一動機是 Claim 1 的 R 從表 K 回答分裂後的重複查詢。表 K 沒了，動機也沒了。建議**撤回修訂一**，改回標準預言機（每次呼叫新鮮隨機性），理由有三：

1. Lemma D 在標準預言機下是完美模擬；在 memoized 預言機下，B 與 C 查同一個 id 會拿到**同一把**金鑰（一種分裂後的共享隨機性），而 B′、C′ 各自算 KeyGen 無法複製這個相關性。要保留等價得多走一步：A′ 在分裂前抽一把 PRF 種子 σ 複製進兩個暫存器，B′、C′ 用 KeyGen(msk, id; PRF_σ(id)) 回答——這樣兩側一致，但金鑰的隨機性從真隨機換成 PRF 輸出，需要一個 PRF hybrid（分辨者 = 整場 G₁，效率沒問題；需後量子 PRF，LWE 給得起）。可行但沒必要。
2. 查詢階段 I 由 R 直接轉送給 GKK25 的挑戰者。**GKK25 Def 8 允許多次查詢、對重複身分不設限、真實世界每次以新鮮隨機性回答**——標準預言機正好對齊，memoized 反而要 R 自己維護一張（懶惰、多項式大小的）表來擋重複查詢。
3. 對 KeyGen 為確定性的方案（GKK25 poly-ID 構造，KeyGen 是 msk 的投影）兩個約定本來就相同；對 KeyGen 隨機化的方案（SXDH 構造，s_i ← ℤ_p）標準約定是 IBE 文獻的預設。

### 2.4 改寫後的證明骨架

| 位置 | 2026-08-28／09-12 版 | 延後引理版 |
|---|---|---|
| 證明第 0 步 | 無 | 引用 Lemma D：WLOG (A, B, C) 在分裂後不查詢 |
| Hyb₁ 建置與查詢階段 I | Sim₁；Sim₂（帶狀態、單一挑戰者依序呼叫） | 不變——**Sim₂ 的帶狀態性完全不是問題**，因為所有呼叫都在分裂前、由同一台機器依序執行 |
| Hyb₁ 挑戰 | 凍結（字典序、T − 1 次 Sim₂）；Sim₃；k̄、r、Sim₄；OTP；給 (ct₁\*, c₂, ρ) | 刪凍結；其餘不變 |
| Hyb₁ 查詢階段 II | 一律回表 K | **刪除** |
| Hyb₁ 揭露 | 給 m̃sk | 不變 |
| 三個設計註記 | 凍結寫進定義、真實世界不需對稱凍結、r 預抽 | 前兩則刪除；只剩「r、c₂ 預抽＋Sim₄ 確定性」 |
| Claim 1 的 R | 凍結；分裂後回表 | 刪凍結；分裂後無查詢；其餘逐字相同 |
| Claim 2 的 Ã | 凍結補滿 K；cls := (st₃, r, c₂, K) | 無凍結；**cls := (st₃, r, c₂)** |
| Claim 2 的 B̃ | 用 K 回答查詢階段 II | 無查詢要回答；亮出 k 後算 k̄ := c₂ ⊕ k、m̃sk := Sim₄(st₃, k̄; r) |
| 「T = poly(λ) 用在哪」 | Hyb₁ 第 3 步 | **不再用到** |

歸約的 straight-line 性質、Lemma 1（OTP 重參數化）、Remark（Sim₄ 確定性）全部不受影響。

### 2.5 主定理敘述的變化

- 語法改為 GKK25 的寫法：Setup(1^λ, 1ⁿ)，n 為身分長度（relaxed 設定時為 Setup(1^λ, 1^T)）。Definition 1、Definition 8（RNC-IB-KEM 語法）同步。
- 主定理對**任意身分空間**成立。實例化清單因此擴大：GKK25 Thm 12（SXDH，指數身分空間，非後量子）與 Thm 3/14（DDH／LWE，poly-ID，後量子）都是合法實例；先前只有後者。
- 修訂三（QPT 提升檢查）不變。

### 2.7 定義怎麼寫：直接寫成 G₀，把 G₁ 降為等價備註

Lemma D 的內容就是「分裂後的查詢不增加對手能力」——對手在 G₁ 與 G₀ 能做到的事完全一樣。既然如此，定義沒有理由保留一個沒有內容的階段。建議：

- **Definition 2 改寫為 G₀**：查詢階段 I → 挑戰 → 分裂 → 揭露 msk → 猜測。分裂之後除了揭露沒有任何互動。
- **加一則備註（或命題）**：「允許 B、C 在分裂後（揭露前或揭露後）對 KeyGen(msk, ·) 做適應性查詢的變體，與本定義等價（Lemma D）；揭露階段只給 sk_{id\*} 的變體被本定義蘊含（見下）。」

這樣寫有三個好處。第一，形狀與 GKK25 Def 8 **完全一致**——他們的遊戲同樣沒有 post-challenge 查詢階段，理由相同：挑戰階段已把 msk 交出。上層遊戲與槽位遊戲同形，Claim 1 的歸約就是純粹的轉送。第二，落在 AKL23 的 cloning game 框架內（分裂後一問一答）。第三，定義稿 Remark 5 所謂「定義層唯一有技術內容的自由度」（查詢階段 II 與揭露的先後）其實沒有內容，把它拿掉，定義就沒有需要辯護的地方。

**「這不就是禁止分裂後查詢嗎？」** 對手的**能力**確實一樣——這正是引理證明的事。差別在於**定理的結論**：若只是在定義裡禁止分裂後查詢，定理只對不查詢的對手成立，審稿人可以問「那會查詢的對手呢」；有了 Lemma D，答案是「被蘊含」。這跟 IBE 證明裡「WLOG 對手不重複查詢同一身分」是同一種句型：不是限制對手，而是把任意對手改寫成一個同樣成功、但形狀更簡單的對手，再對後者證明。引理便宜，但它是「可以放心把定義寫成 G₀」的憑證。

**msk 揭露蘊含「sk_{id\*} 揭露＋分裂後查詢」**：設 (A, B, C) 是「揭露只給 sk_{id\*}、分裂後任意時點可查 id ≠ id\*」這個遊戲的對手。構造 G₀ 的對手：A′ 在分裂前抽一條隨機帶 ω 複製進兩個暫存器；B′ 收到 msk 後令 sk_{id\*} := KeyGen(msk, id\*; ω)（兩側相同，因為 ω 共享且 id\* 在分裂前已知），並以新鮮隨機性回答 B 對其他身分的查詢；C′ 對稱。完美模擬。所以 msk 版是嚴格較強的定義，PDF 第 2 頁「(sk_id\*)」那個變體不需要另外處理，也不建議改過去（§2.6 只是備案）。

**Lemma D 的「內容」在哪**：證明只用了兩件事——(a) 查詢階段 II 與揭露之間沒有其他互動；(b) 查詢的答案只是 (msk, id, 新鮮隨機性) 的函數。任一條不成立，引理就失效：memoized 預言機違反 (b)（B、C 查同一身分拿到同一把金鑰，是一種分裂後才產生的共享隨機性，得靠 PRF 補），揭露只給 sk_{id\*} 則對手無法自足（得改成允許揭露後查詢、由挑戰者回答）。引理同時標出了哪些鄰近變體**不能**這樣做——這就是它不只是換個說法的原因。

### 2.6 若老師採「揭露階段只給 sk_{id\*}」（PDF 第 2 頁的另一處標記；備案）

這時揭露之後 B、C 拿不到其他身分的金鑰，分裂後查詢不再冗餘，定義應允許**揭露之後**也能查詢（記為 G₂′；同時允許揭露前後查詢的 G₃′ 與之等價）。Lemma D 的同款論證仍成立：揭露前的查詢可以延後到揭露後，由**挑戰者**回答（此時 memoized 與否都被挑戰者保持，不需要 PRF）。證明裡：

- Hyb₁ 用 KeyGen(m̃sk, ·) 回答揭露後的查詢；揭露物 sk_{id\*} := KeyGen(m̃sk, id\*)。
- Claim 1 的 R 用 KeyGen(m̄sk, ·) 回答（真實世界＝真 KeyGen，模擬世界＝Hyb₁）。
- Claim 2 的 B̃、C̃ 各自用手上的 m̃sk 回答；sk_{id\*} 的 KeyGen 隨機性 r′ 預抽並放進 cls，兩側算出同一把。標準預言機下兩側各自新鮮抽樣即為正確分布；memoized 下把 PRF 種子放進 cls 並在真實側多一個 PRF hybrid。

同樣**沒有表**、對任意身分空間成立。但要注意：給 msk 是更強的定義（RNC-IB-KEM 免費支撐，定義稿 Remark 8），改成只給 sk_{id\*} 是弱化，除非有別的理由（例如想與 KN23／HMNY21 的某個定義對齊），不建議因為這個議題而改。

---

## 3. 其他替代方案的評估

### A. 延後引理（本文建議）

上節。代價：零（一段一頁不到的引理）。收益：刪表、刪凍結、身分空間任意、修訂一可撤回、定義不動。

### B. 無狀態（history-free）模擬器介面

想法：在 RNC-IB-KEM 的定義裡加一條「Sim₂(st₁, id) 不更新狀態、Sim₃/Sim₄ 不讀取查詢歷史」，則 B̃、C̃ 各持一份 st₁ 就能在分裂後各自呼叫 Sim₂，並保持一致（隨機性由 st₁ 內的種子以 PRF_σ(id) 導出即可確定化）。

核對 GKK25 的兩個構造：

| 構造 | Sim₂ | Sim₃ / Sim₄ | 備註 |
|---|---|---|---|
| §5 SXDH（Thm 12，指數身分空間） | **無狀態**：輸出 sk_id 後 `st₂ := st₁`（其 Sim₂ 的最後一行） | st₃ := (st₂, u₁, u₂)；Sim₄ 用拒絕抽樣（隨機化，本文已用 r 顯式化） | Sim₂ 給的是 semi-functional 金鑰，**不等於** KeyGen(m̃sk, ·) 給的 normal 金鑰；兩者的不可區分性由其 dual-system hybrid 證明 |
| §6 批次加密（Thm 14 → Thm 3，poly-ID） | **有狀態**：維護 st⁽²⁾ = {(id, r_NCE.Setup^id)}，懶惰生成 | 其 Sim₃ 在挑戰時**對所有剩餘身分枚舉** NCE.Sim₂——也就是 GKK25 自己在模擬器內部做了凍結（T = 2^d = poly 才做得到） | KeyGen 確定性（msk 的投影） |

所以「無狀態」對指數身分空間的構造是成立的，對 poly-ID 構造不成立（但那裡枚舉本來就便宜）。然而 B 有兩個致命問題：

1. **救不了 Claim 1**（§1.3）：Def 8 的遊戲挑戰後沒有 Sim₂ 存取，R 拿不到 semi-functional 金鑰來回答分裂後的查詢。要走 B 就得把 Def 8 改成有 post-challenge 查詢階段（模擬世界由 Sim₂ 回答、真實世界由 KeyGen 回答），並**重證 Thm 12**——dual-system 的 hybrid 要能在挑戰密文已是 semi-functional 之後繼續把金鑰切成 semi-functional；預期可行（它們的 Lemma 序列本來就對 q 把金鑰逐一切換、與挑戰的先後順序無關），但這是一整段新證明，且對每個未來的零件都要重做。
2. **多出一條與不可複製性無關的介面假設**，跟 key-transparent 是同一類毛病（老師剛反對過）。

結論：被 A 支配。唯一保留價值：若日後有人堅持「分裂後、揭露前的查詢必須由挑戰者親自回答（不 WLOG 掉）」，B 是那條路，代價如上。

### C. PRF 懶惰表（「懶惰凍結」）

想法：不枚舉，改由 Ã 抽一把 PRF 種子放進 cls，兩側用 PRF_σ(id) 當 Sim₂ 的隨機性懶惰生成金鑰。這只有在 Sim₂ 無狀態、且 Sim₃/Sim₄ 不讀查詢歷史時才一致——即它是 B 的實作細節，不是獨立方案；Claim 1 的問題原封不動。

### D. 保留 T = poly 與凍結（現狀）

GKK25 poly-ID 模擬器內部就在枚舉，所以在 relaxed 設定下凍結「免費」。但這正是老師反對的東西：定理只對 relaxed 設定成立、證明依賴 T = poly、換零件就得重證。A 之下，relaxed 設定仍是主定理的一個實例，什麼都沒少。

### E. 分裂前承諾查詢集合 Q

想法：讓 A 在分裂前宣告一個多項式大小的集合 Q ⊆ ID \ {id\*}，B、C 分裂後只能查 Q 裡的身分；凍結只補 Q，表是多項式大小，身分空間任意。這是介於「無分裂後查詢」與「適應性分裂後查詢」之間的 selective 變體。A 已證明適應性版本本身就是免費的，E 沒有存在的理由。

### F. 只允許分裂前查詢（定義稿出路 3）

定義稿把它當「起步版、分裂後查詢列為 open problem」。Lemma D 說明 G₀ ≡ G₁：**open problem 不存在**。若採 F 作為定義，也只是把等價的兩個寫法選了短的那個。

### 對照表

| 方案 | 刪表？ | 身分空間 | 改 GKK25 假設？ | 改我們的定義？ | 額外證明 | 判斷 |
|---|---|---|---|---|---|---|
| A 延後引理 | 是 | 任意 | 否 | 否（可選：刪查詢階段 II） | Lemma D（一頁內） | **採用** |
| B 無狀態模擬器 | 是 | 任意 | **是**（Def 8 加 post-challenge 查詢） | 否 | 重證 Thm 12；每個零件重核 | 備案 |
| C PRF 懶惰表 | 是 | 任意 | 同 B | 否 | 同 B | 併入 B |
| D 保留凍結 | 否 | 僅 poly | 否 | 否 | 無 | 老師反對 |
| E 承諾集合 Q | 部分 | 任意 | 否 | **是**（弱化） | 無 | 被 A 支配 |
| F 只做分裂前查詢 | 是 | 任意 | 否 | **是**（但與現行等價） | Lemma D 作為備註 | **= A 的定義寫法（§2.7 建議採此形式）** |

---

## 4. 文獻對照：別人怎麼處理「分裂後的預言機」

| 文獻 | 分裂後 B、C 能拿到什麼 | 有無帶狀態的預言機 | 對我們的意義 |
|---|---|---|---|
| AKL23 *Cloning Games*（框架） | 一個古典 challenge、一個回答（Definition 6）；「stateful cloning game」指的是 GenC 的隨機性被 Ver 讀取，不是互動預言機 | 無 | 框架層級就沒有分裂後的互動查詢；我們的 G₀（揭露 msk 即為 challenge）恰好落在框架內，G₁ 落在框架外——這是採 A 的另一個理由 |
| MM24 *Unclonable Functional Encryption* | 函數 C_B、C_C 在分裂**前**就固定（unclonable FE 實驗對所有 C_B、C_C 量化；2-player 版由 A 在挑戰前輸出），B、C 分裂後才收到 sk_{C_B}、sk_{C_C}；「給 B、C 適應性的 KeyGen 預言機」只在 Remark 1、Remark 4 提到，未構造 | 無 | 文獻中唯一帶金鑰查詢的不可複製原語，選擇了「分裂前決定金鑰」的定義；適應性版被視為更強且未達成。我們用 Lemma D 證明 IBE 的適應性版是免費的——可以在論文裡對比 |
| AK21 *Unclonable Encryption, Revisited* | 揭露金鑰 k（一次性）／私鑰 sk（公鑰版） | 無 | 我們的模板；分裂後只有一次揭露 |
| HKNY24（TCC 2024，App E，公鑰 UE from RNCE） | 揭露 sk；歸約裡 B̃、C̃ 各自用預抽的隨機帶算假金鑰 | 無 | 「揭露後本地重算＋預抽隨機帶」的先例，即我們的 r 技巧 |
| AKLLZ22（CRYPTO 2022，QROM 的 UE）；Bartusek 等 2026（Haar random oracle 的 UE） | 分裂後 B、C 可查隨機預言機 H | 預言機**無狀態、可程式化**，用壓縮預言機／reprogramming 分析 | 分裂後有預言機的唯一系列，但性質與帶狀態模擬器完全不同，技術不可移植 |
| Poremba–Ragavan–Vaikuntanathan（ITCS 2026，*Cloning Games, Black Holes and Cryptography*） | 對手被限制為單次 oracle query 的模型 | 無狀態 | 同上；且顯示「限制查詢次數」是文獻中處理分裂後預言機的常見手段——我們不需要 |
| Bartusek–Khurana（CRYPTO 2023，certified deletion，含 ABE）；HMNY22（certified everlasting FE） | 單一對手；證書驗證後對手取得秘密金鑰材料，之後的階段相當於我們的「揭露後」 | 模擬器有狀態，但只有單一對手、無分裂，不產生一致性問題 | 「證書後 = 揭露後」的古典類比：揭露之後的查詢由揭露物本身支撐，跟 §2.6 一致；分裂才是我們多出來的東西 |

**結論**：沒有任何先例處理「分裂後對帶狀態模擬器的適應性查詢」，多數定義在框架層級就避開它。我們的處理方式是證明它可以被無損地 WLOG 掉，這件事本身可以在論文的定義章寫成一個小結果（「adaptive post-split key queries are free for IBE-type cloning games when the reveal contains msk」）。

---

## 5. 對現有文件的修改清單

**定義章（2026-08-14 版）**
- §3 出路表：出路 2 的「定義較弱」改為「與出路 1 等價（Lemma D）」；三條出路改寫為「一條引理」。
- Remark 5「查詢階段 II 必須排在揭露階段之前，否則冗餘」：保留敘述，補一句「但由 Lemma D，排在之前也不增加對手能力；兩種寫法等價」。
- Definition 1／6 語法：Setup(1^λ, 1ⁿ)（relaxed 時 1^T）。

**主定理文件（2026-09-12 OTP 版）**
- 第 3 節開頭加「第 0 步：Lemma D」。
- Hyb₁：刪凍結、刪查詢階段 II、刪前兩則設計註記；Theorem 1/2 刪去對 T 的依賴、身分空間任意。
- Claim 1：表格刪「凍結」列與「查詢階段 II」列；檢查點 1 的合法性只剩「id\* 未被查詢」。
- Claim 2：cls := (st₃, r, c₂)；B̃ 第 1 步刪「用 K 回答」。
- 第 4 節：修訂一改為「撤回」（理由 §2.3）；新增「修訂四：分裂後查詢的延後引理（證明修訂，定義不動）」。
- 第 5 節下一步：實例化清單加入 SXDH（非 pq）。

**路線圖／README**
- 「D-W1 驗證」項目結案：命題成立，形式為對手端 WLOG；D-W1 從「唯一技術硬點」降為「已解」。

---

## 6. 可直接貼進 Overleaf 的 LaTeX 片段

沿用主定理文件的巨集（`\cA`、`\cB`、`\KeyGen`、`\msk`、`\Hyb`、`\stt` 等）。

```latex
% ---------- 放在 \section{主定理的證明} 開頭、\subsection{證明總圖} 之前 ----------
\subsection{第 0 步：分裂後的金鑰查詢可以延後}\label{sec:defer}

令 $G_1$ 為 Definition~\ref{def:uibe-unclonable} 的實驗，$G_0$ 為刪去查詢階段 II 的實驗
（分裂之後直接進入揭露階段）。金鑰預言機採標準約定：每次查詢以新鮮隨機性執行
$\KeyGen(\msk,\cdot)$。

\begin{lemma}[分裂後查詢可延後]\label{lem:defer}
對任意 QPT 對手 $(\cA,\cB,\cC)$，存在 QPT 對手 $(\cA,\cB',\cC')$ 使得
\[
\Pr\bigl[G_0(\cA,\cB',\cC') = 1\bigr] \;=\; \Pr\bigl[G_1(\cA,\cB,\cC) = 1\bigr].
\]
對 Definition~\ref{def:uibe-unclonable-ind} 的不可區分型實驗同樣成立。
\end{lemma}

\begin{proof}
$\cB'$ 收到暫存器 $B$ 後不做任何動作，等到揭露階段收到 $\msk$。然後在內部執行 $\cB$：
餵入暫存器 $B$；$\cB$ 發出的每個查詢 $\id \neq \id^*$，由 $\cB'$ 自己以新鮮隨機性計算
$\KeyGen(\msk,\id)$ 回答；$\cB$ 進入揭露階段時交給它 $\msk$；最後輸出 $\cB$ 的輸出。
$\cC'$ 對稱。

視野比對：在 $G_1$ 中，條件於 $\msk$ 與分裂後的聯合態，$\cB$ 的查詢答案是
$\KeyGen(\msk,\id)$ 的獨立新鮮樣本，且與 $\cC$ 一側的答案獨立；$\cB'$ 內部給出的答案
分布相同、與 $\cC'$ 一側同樣獨立。其餘所有輸入——$\mpk$、查詢階段 I 的答案、挑戰密文、
$\cA$ 的分裂映射、揭露的 $\msk$——在兩個實驗中逐位元相同，因為 $G_0$ 與 $G_1$ 的挑戰者
在分裂之前的行為完全一樣。勝利事件同為 $m_B = m_C = m$。
\end{proof}

\begin{remark}
引理只用到一件事：查詢階段 II 與揭露階段之間沒有其他互動，而金鑰的分布不依賴查詢的
時點。由引理，$G_0$-安全蘊含 $G_1$-安全（同一個界），反向平凡；兩個定義等價。
\textbf{以下證明只處理 $G_0$：對手在分裂後不做金鑰查詢。}
\end{remark}

% ---------- 取代原 Hyb_1 框的第 3、5 步 ----------
  \item \textbf{挑戰}：$\cA$ 輸出 $\id^*$（未查詢過）。
        $(\ct_1^*, \stt_3) \leftarrow \Sim_3(\stt_2, \id^*)$；均勻抽 $\bk \leftarrow \{0,1\}^{\ell}$；
        抽定 $\Sim_4$ 的隨機帶 $r$；$\tmsk := \Sim_4(\stt_3, \bk;\, r)$。
        接著 $k \leftarrow \otUE.\Setup(1^\lambda)$；$c_2 := \bk \oplus k$；
        抽 $m \leftarrow \{0,1\}^n$，$\rho \leftarrow \otUE.\Enc(k, m)$。
        把 $(\ct_1^*,\ c_2,\ \rho)$ 給 $\cA$。
  \item \textbf{分裂}：同 $\Hyb_0$。（由 Lemma~\ref{lem:defer}，分裂後無金鑰查詢。）
  \item \textbf{揭露}：把 $\tmsk$ 給 $\cB$、$\cC$。

% ---------- Claim 2 的 cls ----------
其中 $\mathsf{cls} := (\stt_3,\ r,\ c_2)$ 為古典字串、可自由複製兩份。
```

---

## 參考文獻

- **[GKK25]** Goyal, Kitagawa, Koppula, Nishimaki, Rajasree, Yamakawa. *Non-Committing Identity Based Encryption: Constructions and Applications*. PKC 2025. — Def 8（RNC-IB-KEM，只有 pre-challenge 查詢階段、允許多次查詢）；§5.1 SXDH 構造（Sim₂ 無狀態：`st₂ := st₁`；KeyGen 隨機化；Thm 12）；§6 批次加密構造（Sim₂ 有狀態、挑戰時枚舉；KeyGen 確定性；Thm 14 → Thm 3）；Setup 語法 $(1^\lambda, 1^n)$「in some cases $1^T$」；Thm 11（KEM＋OTP）。
- **[AKL23]** Ananth, Kaleoglu, Liu. *Cloning Games: A General Framework for Unclonable Primitives*. CRYPTO 2023. — Definition 6（分裂後一問一答）、Definition 12（stateful cloning game 指 GenC 隨機性）。
- **[MM24]** Mehta, Müller. *Unclonable Functional Encryption*. ePrint 2024/1683 ／ arXiv:2410.06029；IACR Communications in Cryptology vol. 3 no. 2. — unclonable FE 實驗（C_B、C_C 在分裂前固定）；Remark 1、Remark 4（adaptive 版僅提及）。
- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.
- **[HKNY24]** Hiroka, Kitagawa, Nishimaki, Yamakawa. *Robust Combiners and Universal Constructions for Quantum Cryptography*. TCC 2024, arXiv:2311.09487, Appendix E.
- **[AKLLZ22]** Ananth, Kaleoglu, Li, Liu, Zhandry. *On the Feasibility of Unclonable Encryption, and More*. CRYPTO 2022, ePrint 2022/884.
- **[BK23]** Bartusek, Khurana. *Cryptography with Certified Deletion*. CRYPTO 2023, ePrint 2022/1178.
- **[PRV26]** Poremba, Ragavan, Vaikuntanathan. *Cloning Games, Black Holes and Cryptography*. ITCS 2026.
- **[B+26]** Bartusek et al. *Unclonable Encryption in the Haar Random Oracle Model*. arXiv:2603.11437 (2026).
- 本儲存庫：`Reports/Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md`（D-W1 與三條出路）、`Reports/Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md`（§3 出路表、Remark 5/8）、`papers/Proof_of_Main_Theory_in_UIBE_2026-09-12_OTP.tex`（現行主定理文件）。
