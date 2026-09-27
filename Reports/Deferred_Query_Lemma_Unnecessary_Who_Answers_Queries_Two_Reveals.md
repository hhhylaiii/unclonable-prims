# Lemma 1（分裂後查詢延後）不需要：誰在回答查詢、兩個揭露事件、兩階段遊戲的文獻做法與真正的時間邊界

> **日期**：2026-09-27
> **議題**：v3 主定理稿的 Lemma 1（分裂後查詢可延後）感覺奇怪，但似乎又需要它——因為分裂時 B、C 拿到的是 sk_{id\*} 而不是 msk，不能自己跑 KeyGen(msk, ·)。本文評估有沒有更好的證明方式，並對照文獻中結構相同的兩階段遊戲怎麼處理查詢。
> **一句話結論**：Lemma 1 可以整個拿掉。B、C 確實沒有 msk，但它們從來不需要——**回答查詢的不是 B、C，而是扮演挑戰者的一方**，而在證明的每一個遊戲裡，這一方在分裂之後都持有主私鑰。先前覺得需要 Lemma 1，是把「UIBE 遊戲的揭露」（B 收到 sk_{id\*}）與「otUE 遊戲的揭露」（B̃ 收到 k）混為一談；後者發生在 B̃ 開始模擬 B 之前。修改後的完整稿（v3.1）已附。

---

## 0. TL;DR

1. **B、C 不需要 msk。** 定義裡分裂後的查詢由挑戰者回答；證明裡，分裂後回答查詢的一方分別是：Hyb₀ 的挑戰者（持 msk）、Hyb₁ 的挑戰者（持 m̃sk，挑戰階段就算好）、Claim 1 的 R（持 m̄sk，IBKEM 挑戰階段、也就是分裂之前就收到）、Claim 2 的 B̃／C̃（持 k，**一開始執行就收到**，從而算出 m̃sk）。四者都能用 KeyGen(·, ·) 直接回答查詢階段 II、揭露、查詢階段 III。
2. **Lemma 1 在 v3 裡什麼都沒買到。** v3 回答查詢階段 III 時用到的唯一性質是「回答的一方持有 m̃sk」，而這在查詢階段 II 同樣成立。把查詢階段 II 先搬到 III 再回答，跟直接在 II 回答，是同一件事。
3. **混淆的來源是兩個不同的「揭露」。** otUE 的 cloning 遊戲在分裂與揭露之間沒有任何互動——BL20 Def 7 甚至把分裂後的一方形式化為 D(H_K ⊗ H_B) → D(H_M)，金鑰與暫存器同時作為輸入。所以 B̃ 從「出生」就有 k；B 的整段分裂後生涯（含揭露前的查詢）都是 B̃ 在拿到 k 之後才模擬的。Lemma 1 想做的「延後」，在 otUE 那一層是**自動**發生的。
4. **真正的時間邊界是「分裂」，不是「揭露」。** 證明無法供應的只有一種金鑰：**挑戰之後、分裂之前** 由 A 查詢的金鑰（Ã 在分裂前不知道 k，而 GKK25 Def 8 的模擬器在挑戰後不回答查詢）。我們的定義本來就不給 A 這種查詢——建議把這條邊界與理由明寫成一則 Remark。
5. **文獻印證**：結構最接近的是 GKK25 自己的 Thm 15（RNC-IB-KEM ⇒ incompressible IBE）：第二階段的歸約 B₂ 一開始就收到 SKE 金鑰、跑 Sim₄ 得 msk——與我們的 B̃ 完全同構。反過來，它的 Def 4 允許第一階段對手在挑戰後查詢，而其證明概要沒有交代這一步——正是第 4 點的障礙。HMNY21 的 certified-deletion ABE 也允許挑戰後、刪除前的查詢，而它的 NC-ABE 語法有一個不吃訊息的 FakeSK，正是處理這種查詢所需的額外槽位性質。
6. **建議**：採「直接模擬」；刪除 Lemma 1；新增兩則 Remark（兩個揭露事件、時間邊界）。順帶修正 v3 中「SXDH／DDH 是合法實例」的錯誤。完整稿見 `papers/Proof_of_Main_Theory_in_UIBE_2026-09-27_v3_1.tex`。

---

## 1. 疑慮的精確位置

v3 的 Lemma 1：對任意對手 (A, B, C)，存在 (A, B′, C′)，其在查詢階段 II **不查詢**、且獲勝機率完全相同。B′ 的做法是等到揭露後，才把 B 在查詢階段 II 的查詢**向挑戰者**提出（在查詢階段 III）並轉回。證明第 0 步引用它，之後 Hyb₁、R、B̃ 都只需回答查詢階段 III。

兩點觀察：

- **Lemma 1 本身並不依賴 B 自己跑 KeyGen。** 延後的查詢仍由挑戰者回答——所以「B 只有 sk_{id\*}、不能自己查」這件事，對 Lemma 1 成不成立沒有影響；它影響的是另一個版本（揭露 msk 時的「對手自足」寫法），那個版本在 v3 已經不用了。
- **Lemma 1 對證明沒有貢獻。** 證明回答查詢階段 III 的三個地方（Hyb₁ 挑戰者、R、B̃）用的都是「手上有 m̃sk／m̄sk」。這個條件在查詢階段 II 期間同樣成立（見 §2 的表），所以查詢階段 II 可以原地用同一個方式回答，不必先搬家。

所以問題不是「有沒有比 Lemma 1 更好的引理」，而是「根本不需要引理」。

---

## 2. 誰在回答查詢：四個遊戲逐一核對

| 遊戲 | 分裂後回答查詢（含揭露物）的一方 | 手上持有 | 何時到手 | 分裂後能否回答查詢階段 II？ |
|---|---|---|---|---|
| Hyb₀（真實遊戲） | 挑戰者 | msk | 建置階段 | 能：KeyGen(msk, ·) |
| Hyb₁ | 挑戰者 | m̃sk = Sim₄(st₃, k̄; r) | 挑戰階段（分裂之前） | 能：KeyGen(m̃sk, ·) |
| Claim 1 的 R | R（單一機器，扮演 UIBE 挑戰者） | m̄sk（真實世界 = msk；模擬世界 = m̃sk） | IBKEM 挑戰階段（分裂之前） | 能：KeyGen(m̄sk, ·) |
| Claim 2 的 B̃、C̃ | B̃、C̃ 各自（扮演 B、C 眼中的挑戰者） | k → k̄ = c₂ ⊕ k → m̃sk | **otUE Phase 2 一開始**（B̃ 開始模擬 B 之前） | 能：KeyGen(m̃sk, ·) |

對手 B、C 本身在整個過程中只有 sk_{id\*} 與查到的金鑰——它們不需要更多，因為回答查詢從來不是它們的工作。

Claim 2 裡兩側的一致性：m̃sk 兩側相同（同一個 k、同一份 cls = (st₃, r, r_sk, c₂, id\*)、Sim₄ 給定 r 後確定性）；s̃k_{id\*} 兩側相同（r_sk 預抽並附掛）；查詢答案兩側各自新鮮抽樣——在標準（非 memoized）預言機下，這正是 Hyb₁ 中單一挑戰者的分布。

---

## 3. 兩個揭露事件

```
UIBE 遊戲（B 的視角）
  分裂 ──► 查詢階段 II ──► 【UIBE 揭露：收到 sk_id*】──► 查詢階段 III ──► 輸出 m_B

otUE 遊戲（B̃ 的視角）
  分裂 ──►【otUE 揭露：收到 k】──► B̃ 開始執行：
                                     k̄ := c₂ ⊕ k；m̃sk := Sim₄(st₃, k̄; r)；s̃k_id* := KeyGen(m̃sk, id*; r_sk)
                                     然後才模擬 B 的整段生涯（查詢階段 II → UIBE 揭露 → 查詢階段 III）
                                     全部用 KeyGen(m̃sk, ·) 回答
                                 ──► 輸出 m_B
```

- **形式依據**：AK21 Def 11 的 Phase 2——"B and C are not allowed to communicate. The key k is revealed to both of them."；BL20 Def 7 把 cloning attack 的後兩方形式化為 B_λ : D(H_K ⊗ H_B) → D(H_M)，並令 B_k := ρ ↦ B(|k⟩⟨k| ⊗ ρ)。也就是說，**分裂後的一方是一個「以金鑰為輸入」的量子映射**，中間沒有「先收暫存器、之後才收金鑰」的互動時段。B̃ 就是這樣一個映射：輸入 (k, 暫存器 B ⊗ cls)，輸出 m_B。
- **為什麼這讓 Lemma 1 多餘**：Lemma 1 想做的事是「把揭露前的動作延後到揭露後」。在 otUE 遊戲裡，因為分裂與揭露之間沒有互動，「B̃ 的一切動作都發生在收到 k 之後」是**自動**成立的（任何 B̃ 在收到 k 之前的計算都可以挪到之後，不改變任何東西）。UIBE 遊戲在分裂與揭露之間**有**互動（查詢階段 II），但那整段都是 B̃ 模擬給 B 看的內部劇情——B̃ 自己早就有 k 了。
- 一句話：**UIBE 的揭露是 B̃ 送給 B 的；otUE 的揭露是 B̃ 自己收的。** 前者時間點由 B̃ 在模擬中決定，後者是 B̃ 存在的起點。

---

## 4. 真正的時間邊界：挑戰之後、分裂之前的查詢

證明唯一無法供應的，是**在 Claim 2 裡由 Ã 回答、而且必須在分裂前給出**的金鑰：

- Ã 在分裂前不知道 k（k 由 otUE 挑戰者持有，Phase 2 才揭露），因此算不出 m̃sk = Sim₄(st₃, c₂ ⊕ k; r)，也就無法以 KeyGen(m̃sk, ·) 回答。
- GKK25 Def 8 的模擬世界在挑戰之後**沒有**查詢階段（對手已拿到 msk），所以也不能改用 Sim₂ 在挑戰後回答——Def 8 不保證這樣產生的金鑰與事後的 Sim₄ 輸出一致。

因此：**若 A 可以在收到挑戰密文之後、分裂之前查詢金鑰**，證明在 Def 8 之下走不通。我們的定義（08-14 老師核可版、v3、v3.1）只給 A 查詢階段 I（挑戰之前），沒有這個問題；建議在定義後明寫這條邊界與理由（v3.1 的 Remark「時間邊界是分裂，不是揭露」）。

若日後想允許 A 在挑戰後查詢，選項有三：

| 選項 | 做法 | 代價 |
|---|---|---|
| poly-ID 凍結 | Ã 在挑戰前以 Sim₂ 對所有身分預先作答，挑戰後查表 | 只在 2^d = poly(λ) 成立——這其實才是「凍結」唯一真正的用處 |
| 更強的槽位介面 | 要求模擬器在 session key 揭曉前就能產生金鑰（FakeSK 型），且與事後的 Sim₄ 一致 | 要改 Def 8，並重證零件（GKK25 的構造未證此性質） |
| WLOG 搬到挑戰前 | — | 一般不成立：A 選哪個身分可能依賴挑戰密文 |

---

## 5. 文獻對照：兩階段遊戲如何處理查詢

| 文獻 | 兩階段結構 | 第二階段的一方何時拿到「揭露物」 | 第一階段對手能否在挑戰後查詢 | 與我們的關係 |
|---|---|---|---|---|
| **AK21 Def 11／BL20 Def 7**（otUE cloning game） | 分裂 → 揭露 k | Phase 2 一開始；BL20 把 B 寫成以 k 為輸入的映射 | 無預言機 | §3 的形式依據：B̃ 從出生就有 k |
| **AKL23 Def 6**（cloning game 框架） | 分裂 → Ref 送 ch_B、ch_C → 回答 | 第二階段一開始 | 無預言機 | 框架同形 |
| **HKNY24 App E**（RNCE ⇒ unclonable PKE） | 分裂 → 揭露 sk | B̃、C̃ 收到 otUE 金鑰後各自跑 Reveal | 無金鑰查詢 | 「收到底層金鑰後本地重算揭露物」的模板 |
| **GKK25 Def 4＋Thm 15**（RNC-IB-KEM ⇒ strongly incompressible IBE） | A₁ 壓縮 → A₂ 收到 (mpk, msk, aux, st) | 歸約 B₂ 在第二階段一開始收到 inc.sk，跑 Sim₄ 得 ibkem.msk | **能**（Def 4 有 post-challenge query phase） | 與我們結構最接近：B₂ ≅ B̃。其證明概要只說 B「用前兩個模擬器模擬到挑戰階段」，**沒有交代 A₁ 挑戰後的查詢如何回答**——在 Def 8 之下正是 §4 的障礙 |
| **HMNY21 Def 4.3**（ABE with certified deletion） | 挑戰 → 刪除證書 → 若有效則收到 msk | 刪除之後 | **能**（挑戰後、刪除前、以及收到 msk 後都可查未授權屬性） | 其 NC-ABE 語法有獨立的 FakeSK（不吃訊息），即 §4 表中「更強的槽位介面」那一類；論文正文只以非正式方式陳述 NC-ABE 安全性 |
| **MM24**（unclonable FE） | 分裂 → B、C 收到 sk_{C_B}、sk_{C_C} | 分裂後 | A 不拿金鑰；C_B、C_C 在分裂前就固定 | adaptive 版（B、C 有 KeyGen 預言機）只在 Remark 提及、未構造 |
| **KN23**（one-out-of-many unclonable PE） | 量子密文；simulation-based（Gorbunov 等人 PE 定義的推廣） | — | 待讀其 §6 的正式實驗 | 本來就在「與 KN23 分離」的待辦中 |

兩個可以直接用在論文裡的句子：

- 「後挑戰金鑰由揭露出來的主私鑰產生」這個做法，與 GKK25 自己在 Thm 15 對第二階段對手的處理一致（B₂ 拿到 inc.sk 後以 Sim₄ 產生 msk）。
- 我們的定義刻意不給第一階段對手挑戰後的查詢，理由正是 GKK25 Def 8 不提供挑戰後的模擬金鑰；相對地，GKK25 Def 4 給了這種查詢，而 Thm 15 的證明概要沒有處理它。

---

## 6. 方案比較

| 方案 | 內容 | 正確性 | 評估 |
|---|---|---|---|
| A. 保留 Lemma 1 | 先把查詢階段 II 搬到 III，再只處理 III | 正確 | 多一步看似奇怪、實則無作用的 WLOG；審稿人會問「為什麼需要它」 |
| **B. 直接模擬（建議）** | 查詢階段 II、揭露、III 一律由持有 m̃sk／m̄sk 的一方以 KeyGen 回答 | 正確，所有步驟皆完美模擬 | 最短；讀者只需理解 §3 的兩個揭露事件 |
| C. 從定義刪掉查詢階段 II | 定義層面等價於 A | 正確 | 沒必要；老師要揭露 sk_{id\*}，揭露前的查詢有其語義 |
| D. 允許 A 在挑戰後查詢 | 更強的定義 | 在 Def 8 之下證不出（§4） | 不建議；若要做需 poly-ID 或更強槽位 |

---

## 7. v3.1 的修改清單（完整稿已附，可整份貼上 Overleaf）

- **刪除** Lemma 1（分裂後查詢可延後）與其後的 Remark；證明開頭的「第 0 步」刪除，改為「固定任意 QPT 對手」。
- **Remark「揭露物與查詢階段的設計」**：末句改為「兩段查詢都由挑戰者回答；B、C 從不需要 msk」。
- **新增 Remark「時間邊界是分裂，不是揭露」**（定義之後）：A 在挑戰後、分裂前不能查詢，以及理由與替代選項。
- **新增 Remark「兩個揭露事件」**（§3.1 對照表之後）：UIBE 揭露 vs otUE 揭露；BL20 Def 7 的形式依據。
- **Hyb₁**：查詢階段 II 改為「以新鮮隨機性計算 KeyGen(m̃sk, id) 回答」，查詢階段 III 同查詢階段 II。
- **Claim 1**：R 的查詢階段 II 以 KeyGen(m̄sk, ·) 回答；檢查點 2、3 涵蓋查詢階段 II。
- **Claim 2**：刪除「在查詢階段 II 不查詢」的前提；B̃ 以 KeyGen(m̃sk, ·) 回答 B 的查詢階段 II、III；時序核對改寫。
- **修訂五**：改題為「刪除凍結、金鑰表 K 與延後引理」，說明延後引理為何可以拿掉，以及真正的邊界。
- **順帶更正**：v3 把 GKK25 的 SXDH 構造與 DDH 版列為合法實例（Remark「前提的形狀」、修訂三、下一步）——已改為「目前唯一的實例是 Thm 3/14 取 LWE；SXDH／DDH 對 QPT 對手不安全，cloning 對手可在分裂前以 Shor 打開古典槽」。
- 定義本身（查詢階段 II／III 仍禁查 id\*、揭露 sk_{id\*}）**不動**；證明對「分裂後是否可查 id\*」不敏感，若日後採用 09-13 的「分裂後不設限」提案，同一份證明照用。

---

## 參考

- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021. Def 11（Phase 2 揭露 k）。
- **[BL20]** Broadbent, Lord. *Uncloneable Quantum Encryption via Oracles*. TQC 2020 / arXiv:1903.00130. Def 7（cloning attack：B_λ : D(H_K ⊗ H_B) → D(H_M)）、Def 8。
- **[AKL23]** Ananth, Kaleoglu, Liu. *Cloning Games*. CRYPTO 2023. Def 6。
- **[HKNY24]** Hiroka, Kitagawa, Nishimaki, Yamakawa. TCC 2024, arXiv:2311.09487, Appendix E。
- **[GKK25]** Goyal, Kitagawa, Koppula, Nishimaki, Rajasree, Yamakawa. *Non-Committing IBE*. PKC 2025. Def 4（strongly incompressible IBE，含 post-challenge query phase）、Def 8、§7 Theorem 15 與其證明概要。
- **[HMNY21]** Hiroka, Morimae, Nishimaki, Yamakawa. *Quantum Encryption with Certified Deletion, Revisited*. ASIACRYPT 2021, ePrint 2021/617 / arXiv:2105.05393. Def 4.3（ABE-CD）、NC-ABE 語法（FakeSetup／FakeCT／FakeSK／Reveal）。
- **[MM24]** Mehta, Müller. *Unclonable Functional Encryption*. ePrint 2024/1683. Remark 1、4。
- **[KN23]** Kitagawa, Nishimaki. *One-out-of-Many Unclonable Cryptography*. TCC 2023, ePrint 2023/229 / arXiv:2302.09836。
