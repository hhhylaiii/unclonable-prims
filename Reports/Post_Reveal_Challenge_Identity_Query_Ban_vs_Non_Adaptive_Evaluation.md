# 揭露後禁查 id\* 與 non-adaptive 選項：評估與建議修改

> **日期**：2026-09-28
> **議題來源**：老師：「安全性實驗中，揭露後的查詢階段不能查詢 id\* 好像怪怪的？或是用 non-adaptive 就好？」
> **依據文件**：[主定理稿 v3.1](../Paper%20drafts/Proof_of_Main_Theory_in_UIBE.pdf)（Definition 2/3 的查詢階段 II／III、Remark 1–3、11、Hyb₁、Claim 1/2）。
> **對照文件**：[定義稿](../Paper%20drafts/Draft_of_Security_Definition_of_Unclonable_IBE.pdf)（2026-08-14；Definition 8/9＝Definition A/B，Remark 5–9；尚未同步修訂四）；[KT18 報告](./KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md)與 [KT18 LaTeX 稿](../Paper%20drafts/KT18_x_UE_Constructions_Theorems_Proofs.pdf)（Construction 2/3、Definition 2⁺、Remark「分裂後禁查 id\* 的角色」、Q5）；[09-13 古典／量子盤點報告](./Classical_vs_Quantum_Inventory_and_Challenge_Identity_Query_Rule_Evaluation.md) §2。
> **09-28 追記**：依本報告產出主定理稿 v3.2（修訂六）。v3.2 保留查詢階段 II 的禁令，只解禁揭露後的查詢階段 III；依 §3.1 引理 A，這與 §7.1 的單段寫法（G_free）等價。
> **一句話結論**：老師的直覺是對的。分裂之後（不論揭露前或揭露後）的 id\* 禁令不擋任何平凡攻擊，唯一作用是不讓 B、C 拿到「第二把、獨立產生的 sk_{id\*}」。建議改成「**分裂前禁查 id\*；分裂後先揭露同一把 sk_{id\*}，接著一段不設限的適應性查詢**」。主構造（Construction 1）零成本；KT18 路線的 Construction 2 要把 KeyGen 以 PRF 去隨機化（證明已有草圖，尚待寫成正式證明），Construction 3 另需核對 KDM 查詢的順序。**non-adaptive 不需要**：真正的分水嶺是「分裂後能不能拿到第二把 sk_{id\*}」，與 adaptive 與否無關；改成 non-adaptive 只會得到較弱的定義。

---

## 0. TL;DR

| 問題 | 結論 |
|---|---|
| 定義稿裡有這條禁令嗎？ | **「揭露後」的禁令沒有**。定義稿（08-14）揭露的是整把 msk，揭露之後沒有查詢階段；Remark 7 明寫「揭露階段之後不設限」。但它在分裂後、揭露前的查詢階段 II 仍禁查 id\*（在揭露 msk 的版本裡，這段查詢整個是冗餘的，§1）。老師指的「揭露後」禁令，是 09-13 修訂四（改揭露 sk_{id\*}）之後的寫法，也就是 v3.1 Definition 2 的查詢階段 III「仍不得查詢 id\*」。**定義稿還沒同步修訂四**，所以不論這題結論如何，定義稿都要改。 |
| 這條禁令怪嗎？ | **怪**。分裂前的禁令擋的是平凡攻擊（A 先解密再複製古典的 m），是不可複製性本身的內容；分裂後 B、C 本來就會拿到 sk_{id\*}，禁令擋不到任何**平凡**攻擊，只是不讓他們多拿幾把**獨立產生**的 sk_{id\*}。對 KeyGen 隨機化的方案，這一點可能有差（§3.4），所以拿掉禁令是實質的加強，不只是改寫。 |
| 「揭露前」與「揭露後」有差嗎？ | **沒有**（§3.1，即第三版的延後引理）。分裂後的查詢放在揭露前或揭露後，定義強度完全相同，所以問題其實是「分裂後要不要禁查 id\*」。 |
| 是否需要修改？ | **建議修改**。現行寫法不是錯的（KT18 的 SIM-RSO 定義也有同樣的禁令，見 §6），但可以改成更強、也更自然的版本。主構造的定理直接撐得住；KT18 路線需要補寫證明。 |
| 代價 | **Construction 1**：零。v3.1 Remark 3（揭露 msk 的強化版）已經蘊含新定義（§3.2），證明只要刪掉「id ≠ id\*」。**Construction 2**：KeyGen 以後量子 PRF 去隨機化（KT18 footnote 6）。證明要多一步 PRF hybrid，並在歸約與 wrapper 裡維持「同一身分同一把金鑰」的一致性（查表、Q-wise independent 函數）；KT18 稿已有草圖，尚待寫成正式證明。去隨機化之後，分裂後再查 id\* 只會拿回同一把金鑰（§3.3）。**Construction 3**：預期同法可行，但要另外核對分裂前的 KDM 查詢是否用到 s\*。 |
| 改用 non-adaptive？ | **不需要**。「拿掉分裂後查詢」確實讓禁令消失，但得到的是較弱的定義，也不再模擬「複製之後其他使用者被腐化」；MM24 的 non-adaptive 版本本身也不限制分裂後拿哪把金鑰。「分裂後不給預言機」的版本（§5 的 N1）可以當成新定義的推論，寫一句備註即可。 |

---

## 1. 定義稿的寫法，以及老師指的是哪一版

**定義稿（2026-08-14）Definition A 的實驗**：建置 → 查詢階段 I（A，適應性）→ 挑戰（id\* 未在查詢階段 I 查過）→ 分裂 → 查詢階段 II（B、C 各自查詢，**不得查詢 id\***）→ **揭露階段：B、C 皆收到整把 msk** → 猜測。配套的 Remark：

- Remark 5：查詢階段 II 必須排在揭露之前，否則冗餘；並稱它是「本定義相對於 unclonable 公鑰加密唯一多出來的自由度，也是定義層唯一有技術內容的地方」。
- Remark 7：「不得查詢 id\*」只約束查詢階段 I 與 II；揭露後不設限，因為整把 msk 都給了。
- Remark 8：揭露整把 msk 是更強的定義，由 RNC-IB-KEM 免費支撐。

**09-13 修訂四（老師要求）**把揭露物改成 sk_{id\*}。因為 B、C 此後拿不到其他身分的金鑰，v3 起的寫法把分裂後查詢拆成兩段：查詢階段 II（揭露前，禁查 id\*）→ 揭露 sk_{id\*} → 查詢階段 III（揭露後，**仍不得查詢 id\***）。老師覺得怪的是最後這條。定義稿至今沒有同步這次修訂（README 的工作順位第 2 項「定義章同步」）。

**附帶發現：定義稿 Remark 5 不精確。** 在揭露 msk 的版本裡，查詢階段 II 即使放在揭露之前也完全冗餘：把 B 換成「先收下 msk、再在內部執行 B，並以 KeyGen(msk, ·) 自己回答 B 在查詢階段 II 的每個查詢」即可，B 的視野分布完全相同。所以定義稿的 Definition A 實質上等於「分裂後沒有任何查詢」的版本，Remark 5 說的「唯一有技術內容的地方」並不存在。直到修訂四把揭露物降成 sk_{id\*}，分裂後查詢才可能增加對手的能力，例如在看過挑戰之後才選的身分、兩人各自的獨立金鑰；這些都無法由 sk_{id\*} 算出。

---

## 2. 每一段禁令在擋什麼

| 時段 | 若允許查詢 id\* 會發生什麼 | 禁令必要嗎？ |
|---|---|---|
| 分裂前（查詢階段 I；若有挑戰後、分裂前的查詢亦同） | A 取得 sk_{id\*}，對 ρ 解密得到**古典**的 m，把 m 複製進兩個暫存器，以壓倒性機率獲勝（解密正確性）。任何允許這件事的定義都無法被滿足。 | **必要**。這就是不可複製性的內容，對應 AK21 中「k 只在分裂後揭露」。 |
| 分裂後、揭露前（查詢階段 II） | B（或 C）多拿到一把**獨立產生**的 sk_{id\*}（不是揭露物那一把）。依 §3.1，這等同於揭露後再查一次，與下一列是同一件事。 | 不必要（不擋平凡攻擊） |
| 揭露後（查詢階段 III） | B、C 手上已有 sk_{id\*}；再查 id\* 只會多拿到**獨立產生**的金鑰（KeyGen 隨機化時），或同一把（KeyGen 確定性時）。 | 不必要（不擋平凡攻擊） |

表中的「不必要」指不擋平凡攻擊。禁令確實限制了對手的能力：對 KeyGen 隨機化的方案，可以人工構造在現行定義下安全、拿掉禁令後被攻破的例子（§3.4）。

**與標準 IBE 的差別。** IND-ID-CPA 整場遊戲都禁查挑戰身分，是因為對手自始至終握有挑戰密文，一把 sk_{id\*} 就能平凡解密。cloning game 的設計則是**刻意在分裂之後把 sk_{id\*} 交出去**，不可複製性的主張正是「即使兩人都持有 sk_{id\*}，也無法同時解密」。把標準 IBE 的禁令原封不動延伸到分裂之後，才造成「揭露後還禁查」這個看起來矛盾的組合。

**與 certified deletion 的差別。** HMNY21 的 ABE 刪除安全（Def 4.3）在有效刪除證書之後把 msk 交給對手，卻仍限制之後的金鑰查詢；其 footnote 8 說明這樣寫的原因：「Such queries are useless if A obtains msk in the previous item, but may be useful if the challenger returns ⊥ there」。也就是說，限制只為了證書無效（⊥）那一支，因為那時對手仍握有密文。cloning game 沒有這一支：分裂一定發生、揭露一定給出，所以這個理由不適用。

---

## 3. 形式化：幾個版本之間的關係

記號（除 G_msk 外，揭露物皆為**同一把** sk_{id\*}；預言機採 v3.1 的標準約定：每次呼叫以新鮮隨機性執行 KeyGen）：

- **G_ban**：分裂後可查詢，但不得查詢 id\*（v3.1 現行）。
- **G_free**：分裂後可查詢任何身分，含 id\*、可重複（**建議版**）。
- **G_none**：分裂後沒有任何查詢（non-adaptive 的一種讀法，即 §5 的 N1）。
- **G_msk**：揭露整把 msk（v3.1 Remark 3 的強化版；分裂後查詢冗餘）。

### 3.1 引理 A（時序無關；即第三版的延後引理）

> 設分裂後的兩段查詢（揭露前、揭露後）允許的身分集合相同。則「查詢 → 揭露 → 查詢」與「揭露 → 查詢」兩種遊戲等價：任一方的對手都可轉換成另一方的對手，獲勝機率不變。

**證明**：「揭露 → 查詢」的對手 B′ 先收下揭露物，在內部執行原本的 B：B 在揭露前的查詢，B′ 原樣轉送給（揭露後的）預言機；B 進入揭露時，B′ 把手上的揭露物交給它；之後的查詢照轉。預言機無狀態、答案分布與時點無關，所以 B 的視野分布不變。反方向是平凡的（揭露前不查詢即可）。C 相同。∎

**推論**：老師說的「揭露後」與「揭露前」沒有差別，只要讓分裂後的查詢不設限，禁令就整個消失；把揭露放在分裂後的第一步、只寫一段查詢，定義最乾淨。這條引理 v3.1 已刪出證明，但修訂五保留它作為「揭露前後的查詢沒有差別」的說明，正好用在這裡。

### 3.2 引理 B（蘊含鏈）

> 若方案在 G_msk 下安全，則在 G_free 下安全；G_free ⇒ G_ban ⇒ G_none。

**證明**：G_free ⇒ G_ban：先用引理 A 把 G_ban 寫成「揭露在前」的形式，它就是 G_free 限制了查詢集合的特例。G_ban ⇒ G_none 是平凡的（分裂後不查詢即可）。G_msk ⇒ G_free：給定 G_free 的對手 (A, B, C)，構造 G_msk 的對手 (A′, B′, C′)。A′ 照跑 A，另外均勻抽一段 KeyGen 的隨機帶 ω，連同 id\* 複製進兩個暫存器（古典字串，可複製）。B′ 收到 msk 後計算 sk := KeyGen(msk, id\*; ω)，把它當揭露物交給 B，並以 KeyGen(msk, ·) 加新鮮隨機性回答 B 的所有查詢（含 id\*）。C′ 用同一段 ω，所以兩側拿到同一把 sk。ω 與其他一切獨立，因此 sk 的分布與挑戰者新鮮產生的 sk_{id\*} 相同。∎（這與 v3.1 Claim 2 預抽 r_sk、隨 cls 複製的手法完全相同。）

**推論**：v3.1 Remark 3 已經說明 Construction 1 在 G_msk 下成立，所以 **Construction 1 在 G_free 下自動成立**，不需要重新證明。

### 3.3 引理 C（確定性 KeyGen 下，禁令是空的）

> 若 KeyGen 為確定性（例如以 PRF 去隨機化），則 G_ban ⇒ G_free，兩者等價。

**證明**：依引理 A 先把揭露移到分裂後的第一步。A′ 把 id\* 複製進兩個暫存器，讓 B′、C′ 認得對 id\* 的查詢。此時對 id\* 的查詢答案就是 KeyGen(msk, id\*)，也就是揭露物本身；B′ 直接用手上的揭露物回答，其他查詢照轉。∎

### 3.4 真正的分水嶺

三條引理合起來，問題的核心不在「適應性」，也不在「揭露前或後」，而在：

> **分裂後，B、C 能不能拿到第二把、獨立產生的 sk_{id\*}？**

- KeyGen 確定性時，「第二把」就是同一把，放寬沒有內容（引理 C）。
- KeyGen 隨機化時，放寬才有內容，而且可能是嚴格的加強。人工反例：讓 id 的每把金鑰依新鮮硬幣附上 t_id 或 t_id ⊕ d_id（t_id 為偽隨機值，d_id 是另一個 IBE′ 對 id 的金鑰），並在密文裡多放一份 IBE′.Enc(id, m)。一把金鑰看不出 d_id，所以現行定義下這份古典密文無用；拿掉禁令後，B、C 各自多查幾次 id\*，就能以高機率 XOR 出 d_{id\*}，兩人都讀出 m。所以拿掉禁令是實質的加強。
- 放寬對 Construction 1 無害，因為 Sim₄ 交出的是整把 m̃sk，要幾把一致的金鑰都有；對 Construction 2 目前的證明則有害（§4）。
- 附帶好處：G_free 同時涵蓋 AKL23 所說的兩種挑戰分布。揭露句給兩人**同一把**（identical，同 AK21）；不設限的查詢讓兩人也能各自查到**獨立**的金鑰（independent，即 MM24 所稱的 variable decryption keys）。

---

## 4. 對三個構造的影響

| | G_none | G_ban（現行） | G_free（建議） | G_msk |
|---|---|---|---|---|
| **Construction 1**（v3.1，GKK25 RNC-IB-KEM ＋ OTP） | ✓ | ✓ | **✓（由 G_msk 蘊含；證明只刪「id ≠ id\*」）** | ✓（Remark 3） |
| **Construction 2**（KT18 雙重加密，KeyGen 隨機化） | ✓ | ✓（Theorem 3） | **現行證明不成立**（見下）；無已知攻擊 | 無證明 |
| **Construction 2**（KeyGen 以 PRF 去隨機化） | ✓ | 預期 ✓：Theorem 3 的骨架＋PRF hybrid＋一致性維護（KT18 稿 Remark (a)–(d) 的草圖，待寫成正式證明） | **與 G_ban 等價（引理 C）** | 無證明 |
| **Construction 3**（KDM 雙槽） | ✓ | ✓（Theorem 6 無 KDM 查詢的特例） | 預期同法可行：槽 1 的選擇字串 s 同樣去隨機化（sk⁽²⁾ 已是確定性）；另需核對分裂前的 KDM 查詢是否用到 s\*；未逐步寫出 | 無證明 |

**Construction 1 的直接改法**（若不想只靠引理 B）：Hyb₁、Claim 1 的 R、Claim 2 的 B̃／C̃ 本來就以 KeyGen(m̃sk, ·)（或 KeyGen(m̄sk, ·)）加新鮮隨機性回答分裂後的查詢，直接把 id\* 也這樣回答即可。揭露物仍用預抽的 r_sk 算出，兩側一致。Claim 1 的合法性檢查不受影響，因為分裂後的查詢都由 R 自己回答、不轉送給 IBKEM 挑戰者。

**Construction 2 為什麼卡住**（KT18 稿 Remark「分裂後禁查 id\* 的角色」）：Hyb₂ 裡挑戰密文的兩半加密 r_j ⊕ α，揭露物的選擇字串是 s\* = r ⊕ k。若 B 再查一次 id\*，拿到新的選擇字串 s′ ≠ s\*，它在 s′_j ≠ s\*_j 的位置會解出 1 − k_j，與真實世界可分辨；Claim R1 的 𝒟_i 也拿不到 IBE 挑戰身分 (id\*, i, 1 − s\*_i) 的金鑰。**這是證明的缺口，目前沒有已知攻擊**：真實方案裡兩半加密同一個 k_j，第二把金鑰只會再給一次 k。這只說明這個分辨方法在真實方案裡無效，並不構成安全證明。

**修法**（KT18 footnote 6：「we can always make a key generation algorithm of IBE deterministic by using pseudorandom functions」）：令 (s, KG 的隨機帶) := PRF(K, id)，K 放進 msk。證明依 KT18 稿該 Remark 的 (a)–(d)：(a) 開頭加一步「PRF → 真隨機函數」；(b) RF(id\*) 在分裂前從不被用到（A 不得查 id\*），可在挑戰時抽出當 s\*，並在 Hyb₂ 程式化成 r ⊕ k；(c) 𝒟_i 以查表維持一致；(d) wrapper 對 id\* 用程式化的 s\* 與預產生的候選金鑰，對其他身分用放在 cls 裡的 Q-wise independent 函數（古典查詢下與 RF 完全同分布，兩側一致）。**不能把 PRF 金鑰本身放進 cls**，否則 s\* 在分裂前就被決定。代價是一個後量子 PRF（由後量子單向函數即得，不增加假設；只需對古典查詢安全）、一步 hybrid，以及 (c)(d) 的一致性維護。去隨機化之後，同一身分在 A、B、C 之間都是同一把金鑰，這已不是 Theorem 3 原本證明的遊戲，所以 (c)(d) 要逐步寫出，不能只引用 Theorem 3。Construction 3 另需注意：K2 的 KDM 查詢可以依賴 sk_{id\*}（含槽 1 的 s\*），因此 (b)「RF(id\*) 在分裂前從不被用到」要到 Theorem 6 的 Step 1（先中和輔助密文）之後才成立，順序要重新核對。

---

## 5. 老師的 non-adaptive 選項

「non-adaptive」有三種讀法，逐一看：

| 讀法 | 做法 | 禁令還在嗎？ | 評估 |
|---|---|---|---|
| **N1：分裂後不給預言機**（只剩查詢 I 與揭露） | 分裂後 B、C 只收到 sk_{id\*}，不能再查詢 | 消失（整段查詢都拿掉了） | 最簡單，與 AK21、HKNY24 的公鑰版同形；但它是 G_none，**比 G_ban、G_free 都弱**，而且不再模擬「複製之後其他使用者被腐化」。在揭露 msk 的定義稿裡，它其實與 Definition A 等價（§1 附帶發現）。對 Construction 1 沒有省到任何東西；對 Construction 2 只省下去隨機化。 |
| **N2：MM24 式的 non-adaptive**（分裂後拿哪些金鑰事先決定） | A 事先宣告 B、C 分裂後各拿哪些身分的金鑰，分裂後直接發給 | 看清單能不能含 id\*：MM24 Def 26 對 C_B、C_C **不設任何限制**（identity circuit 也行），照抄就等於允許 | non-adaptive 本身不解決問題：清單若含 id\*，Construction 2 遇到的是同一個「第二把金鑰」缺口；若不含，禁令只是換成靜態的。 |
| **N3：selective-ID**（A 開場就宣告 id\*） | Sim₁／歸約開場就知道 id\* | 仍在（靜態檢查） | 與這題無關（09-13 報告 §2.3）。只在未來某個 LWE 零件只有 selective 安全時當備案。 |

**結論**：non-adaptive 沒有碰到問題根源（§3.4），只會把定義變弱。建議主定義採 G_free；若想保留 N1 的簡潔敘述，可以在定義後加一句「拿掉分裂後的查詢階段即得較弱的版本，本文定理對較強版本成立，故亦蘊含之」。

---

## 6. 文獻怎麼處理

| 文獻 | 「事件」之後對手能拿到什麼 | 事件之後還限制查詢嗎？ | 對本題的意義 |
|---|---|---|---|
| **BF01**（標準 IBE，IND-ID-CPA） | 無揭露 | 整場禁查 id\* | 對手始終握有挑戰密文，禁令必要。cloning game 不是這個情形。 |
| **AK21** Def 11、**HKNY24** App. E（UE／unclonable PKE） | 分裂後兩人都拿到**同一把** k 或 sk | 沒有預言機 | 對應 N1 的形狀；揭露物＝能解密的金鑰本身。 |
| **AKL23**（cloning games） | 分裂後兩人都拿到 challenge（UE 中即金鑰） | — | 區分 identical 與 independent 兩種挑戰分布；G_free 兩者都涵蓋。 |
| **MM24**（Mehta–Müller，unclonable FE）Def 26 與 Remark 1 | 分裂後兩人各拿一把**獨立產生**的函數金鑰，電路 C_B、C_C 任選（identity circuit 亦可） | **不限制**；Def 26 標為「Non-adaptive」，Remark 1 的 adaptive 版給 B、C KeyGen 預言機 | 最接近「分裂後金鑰存取」的先例：**分裂後能解密的金鑰不受限**，non-adaptive 與否只是金鑰何時決定。 |
| **HMNY21** Def 4.3（ABE 刪除安全） | 有效刪除證書後給 msk | 仍寫「P(X\*) = ⊥」，footnote 8 說明只為了證書無效的 ⊥ 分支 | 事件後的限制只在「事件可能失敗」時有意義；分裂不會失敗。 |
| **KT18** Def 11（IBE 的 SIM-RSO） | 開啟時給被開啟身分的 sk | **挑戰後禁止對挑戰身分做 extraction**，金鑰只能經由開啟取得 | **與 G_ban 同形的先例**。KT18 禁的是所有挑戰身分：對未開啟的身分，它是標準 IBE 那種擋平凡攻擊的禁令；對已開啟的身分，它才對應我們分裂後的禁令（我們的 id\* 一定會被「開啟」）。同一篇 footnote 6 指出 KeyGen 永遠可以用 PRF 做成確定性；在那之下，對已開啟身分的禁令就是引理 C 說的空禁令。 |

所以現行寫法（G_ban）有 KT18 這個先例，不能說錯。但它背後的語義是「每個身分只有一把金鑰，開啟就是那一把」。在我們採用的標準預言機約定（每次查詢新鮮抽樣）下，這個語義沒有被寫出來，讀者只看到「已經給了、卻不准再問」，這正是老師覺得怪的來源。改成 G_free，並在 KT18 路線把 KeyGen 去隨機化，就把這層語義明確化了。

---

## 7. 建議的修改

### 7.1 定義（可直接取代定義稿的 Definition A 實驗框；沿用定義稿巨集，v3.1 請把 `1^T` 換成 `1^d`）

```latex
\begin{expbox}{$\Expt^{\mathrm{cloning}}_{\UIBE,(\cA,\cB,\cC)}(1^\lambda, 1^T)$}
\begin{itemize}[itemsep=2pt,topsep=1pt]
  \item \textbf{建置階段}：挑戰者執行 $(\mpk,\msk) \leftarrow \Setup(1^\lambda,1^T)$，將 $\mpk$ 送給 $\cA$。
  \item \textbf{查詢階段 I}：$\cA$ 可對預言機 $\KeyGen(\msk,\cdot)$ 做多項式次（古典的、適應性的）金鑰查詢。
  \item \textbf{挑戰階段}：$\cA$ 輸出一個查詢階段 I 中未曾查詢過的 $\id^*$。
        挑戰者均勻抽樣 $m \leftarrow \{0,1\}^n$，計算 $\rho \leftarrow \Enc(\mpk, \id^*, m)$ 並送給 $\cA$。
  \item \textbf{分裂階段}：$\cA$ 對其持有的量子態施加 CPTP 映射
        $\Phi : \Dm(\cH_A) \to \Dm(\cH_B \otimes \cH_C)$，將 $B$ 暫存器交給 $\cB$、$C$ 暫存器交給 $\cC$。
        \textbf{此後 $\cB$ 與 $\cC$ 不得通訊。}
  \item \textbf{揭露階段}：挑戰者計算 $\sk_{\id^*} \leftarrow \KeyGen(\msk,\id^*)$，
        將\textbf{同一把} $\sk_{\id^*}$ 交給 $\cB$ 與 $\cC$。
  \item \textbf{查詢階段 II}：$\cB$ 與 $\cC$ \emph{各自獨立地}對預言機 $\KeyGen(\msk,\cdot)$
        做多項式次（古典的、適應性的）金鑰查詢，\textbf{不設任何限制}（可查詢 $\id^*$、可重複查詢）。
  \item \textbf{猜測階段}：$\cB$ 輸出 $m_B$，$\cC$ 輸出 $m_C$。
\end{itemize}
實驗輸出 $1$（對手獲勝）$\iff m_B = m_C = m$。
\end{expbox}

金鑰查詢預言機採標準約定：每次呼叫以新鮮隨機性執行 $\KeyGen(\msk,\id)$，
不同呼叫（含同一身分的重複查詢、跨 $\cB$／$\cC$ 的查詢）彼此獨立。
```

新定義的「查詢階段 II」是分裂後唯一的一段查詢，等於舊寫法的查詢階段 II＋III。Definition B（不可區分型）的「修改的兩個階段」文字不必改：它只引用查詢階段 I。

**取代定義稿 Remark 5–8 的四則 Remark**（Remark 9「金鑰查詢為古典查詢」保留）：

```latex
\begin{remark}[時間邊界是分裂：只有分裂前禁查 $\id^*$]
若 $\cA$ 在分裂之前取得 $\sk_{\id^*}$，它可解密 $\rho$ 得到古典的 $m$，再把 $m$ 複製進兩個暫存器，
以壓倒性機率獲勝；任何允許這件事的定義都無法被滿足。因此「不得查詢 $\id^*$」只約束分裂之前——
這就是不可複製性的內容，與 AK21 一次性遊戲中「$k$ 在分裂後才揭露」是同一件事。
分裂之後不需要任何限制：$\cB$、$\cC$ 本來就會收到 $\sk_{\id^*}$，不可複製性的主張正是
「即使兩人都持有 $\sk_{\id^*}$，也無法同時解密」。這與標準 IBE 不同：IND-ID-CPA 整場禁查 $\id^*$，
是因為對手始終握有挑戰密文；cloning game 則是在分裂之後刻意把 $\sk_{\id^*}$ 交出去。
\end{remark}

\begin{remark}[揭露的位置不影響定義強度]
分裂之後，把查詢放在揭露之前或之後，得到的定義等價：面對「查詢、揭露、查詢」的對手，
可以改寫成「先收下揭露物，再依序代為轉送它的查詢，並在它預期的時點交出揭露物」
（預言機無狀態，答案分布與時點無關）。因此本定義只寫一段分裂後的查詢，並把揭露放在分裂後的第一步。
\end{remark}

\begin{remark}[同一把 $\sk_{\id^*}$ 與各自的 $\sk_{\id^*}$]
揭露交給兩人的是同一把金鑰，對應 AK21 中兩人收到同一把 $k$（AKL23 的 identical 挑戰分布）。
由於查詢階段 II 不設限，兩人也可以各自查詢 $\id^*$、取得各自新鮮產生的金鑰
（MM24 的 variable decryption keys 形式），所以本定義同時涵蓋兩種慣例。
若 $\KeyGen$ 為確定性（例如以 PRF 去隨機化），查詢 $\id^*$ 只會拿回同一把 $\sk_{\id^*}$。
若拿掉查詢階段 II（分裂後不再給預言機），得到較弱的版本；本文的定理對上述較強版本成立，故亦蘊含之。
\end{remark}

\begin{remark}[揭露整把 $\msk$ 的強化版]
把揭露物換成 $\msk$ 得到較強的定義（此時查詢階段 II 冗餘），且它蘊含本定義：
對手可在分裂前均勻抽一段 $\KeyGen$ 的隨機帶 $\omega$、連同 $\id^*$ 複製進兩個暫存器，
兩側以 $\KeyGen(\msk,\id^*;\,\omega)$ 算出同一把 $\sk_{\id^*}$，其餘查詢以 $\KeyGen(\msk,\cdot)$ 自行回答。
\end{remark}
```

定義稿 Remark 6（「B、C 各自持有對同一個挑戰者的獨立預言機存取」）的**結論保留、理由要換**：原文說「這正是帶狀態模擬器會出問題的地方」，但 09-13 起已確認分裂後的查詢在證明中一律由持有（模擬的）主私鑰的一方以 KeyGen 回答，Sim₂ 從不在分裂線右邊被呼叫（v3.1 Remark 11）。建議改成：「兩人各自對同一個預言機查詢，答案各自以新鮮隨機性產生、互不影響。」

### 7.2 主定理稿 v3.1 的連動修改

1. **Definition 2／3**：實驗框換成 §7.1（`1^d`）；Remark 1「揭露物與查詢階段的設計」改寫為 §7.1 的前兩則 Remark；Remark 3 的「此時查詢階段 III 冗餘」改為「此時查詢階段 II 冗餘」，並補上蘊含 G_free 的一句（引理 B）。
2. **Hyb₁**：步驟 5–7 改為「揭露：把 s̃k_{id\*} 給 B、C；查詢階段 II：B、C 的每個查詢 id（**含 id\***）以新鮮隨機性計算 KeyGen(m̃sk, id) 回答」，刪掉「查詢階段 III：同查詢階段 II」。
3. **Claim 1 的表**：「查詢階段 II／揭露／查詢階段 III」三列改成「揭露／查詢階段 II（每個查詢 id，含 id\*，以新鮮隨機性計算 KeyGen(m̄sk, id)）」；檢查點 2、3 的「查詢階段 II、III」改為「查詢階段 II」。合法性檢查不變。
4. **Claim 2 的 B̃ 第 2 步**：「內部執行 B：交給它 s̃k_{id\*}；B 的每個查詢 id（含 id\*）以新鮮隨機性計算 KeyGen(m̃sk, id) 回答」。視野比對與時序核對中的「查詢階段 II、揭露、查詢階段 III」改為「揭露、查詢階段 II」。
5. **總圖的比較表、Hyb₁ 後的「三個設計註記」、Remark 11** 中「查詢階段 II、III」一併改為「查詢階段 II」。
6. **第 4 節**新增**修訂六**：「分裂後查詢不設限、揭露移到分裂後第一步」，理由引用引理 A、B（本文 §3）。修訂一、四、五中描述「查詢階段 II／III」的段落是歷史紀錄，可保留，註明由修訂六取代。

### 7.3 KT18 稿的連動修改（若採 G_free）

1. **Construction 2、3**：KeyGen 改為以後量子 PRF 去隨機化，(s, 隨機帶) := PRF(K, id)，K 放進 msk。Construction 3 的槽 1 同樣處理（槽 2 已經是確定性）。
2. **Definition 2⁺**：「II、III 仍不得查詢 id\*」改為「分裂後不設限」（查詢階段 I′ 仍不得查詢 id\*，因為仍在分裂前）。Definition SO（目標身分 id^(i\*)）與 K2（id\*）的對應修改另行核對：SO 還有其他未開啟的挑戰身分，K2 對其他**登錄**身分的分裂後限制另有 KDM 證明上的角色（登錄身分的金鑰不能被揭露），都不在本評估範圍。
3. **Theorem 3**（及連帶的 Theorem 4、6）：前提加上「PRF 對 QPT 安全」；證明依 Remark「分裂後禁查 id\* 的角色」(a)–(d) 寫成正式步驟（Theorem 6 另需核對 KDM 查詢與 s\* 的順序，§4）。該 Remark 改名為「KeyGen 去隨機化與分裂後查詢」。
4. **Q5 結案**：以去隨機化處理，不保留禁令。

### 7.4 定義稿其他需要同步的地方（與本題無關，但都已過時）

- **構造**：仍是 k := otUE.Setup(1^λ; k̄)，要換成修訂二拍板的 OTP 墊層（ct = (ct₁, c₂ := k̄ ⊕ k, ρ)）；Remark 3（key-transparent 捷徑）刪除；Remark 4 換成 v3.1 的「古典側與量子側的分工」。
- **§3 表格的 Sim₂ 列與「分裂後查詢的三條出路」**：前提（分裂後查詢要靠 Sim₂ 回答、會分岔）已確認不存在（09-13 報告 §2.4、v3.1 Remark 11），整段改寫為「分裂後的金鑰一律由持有模擬主私鑰的一方以 KeyGen 回答」。
- **記號**：Setup(1^λ, 1^T) → Setup(1^λ, 1^d)（v3.1 起身分空間大小不進入定理）。

---

## 8. 待老師拍板

- **Q-a**：主定義採 G_free（分裂前禁查 id\*、分裂後先揭露同一把 sk_{id\*}、再不設限查詢）？或採 N1（分裂後不給預言機）當主定義？**建議 G_free**，N1 以一句 Remark 帶過。
- **Q-b**：若採 G_free，KT18 路線的 Construction 2／3 以 PRF 去隨機化 KeyGen（即 KT18 報告 Q5 的做法）？**建議同意**；成本是一個由後量子單向函數即得的 PRF、一步 hybrid，以及證明中的一致性維護（需逐步寫出）。
- **Q-c**：揭露物維持「同一把」sk_{id\*}（對齊 AK21）？**建議維持**；不設限的查詢已經同時涵蓋 MM24 的「各自一把」。

---

## 參考

- **[AK21]** P. Ananth, F. Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021.（Def 11：分裂後兩人收到同一把 k）
- **[AKL23]** P. Ananth, F. Kaleoglu, Q. Liu. *Cloning Games: A General Framework for Unclonable Primitives*. CRYPTO 2023.（identical 與 independent 挑戰分布）
- **[BF01]** D. Boneh, M. Franklin. *Identity-Based Encryption from the Weil Pairing*. CRYPTO 2001.（IND-ID-CPA：第二階段查詢須 ≠ 挑戰身分）
- **[GKK25]** R. Goyal, F. Kitagawa, V. Koppula, R. Nishimaki, M. S. Rajasree, T. Yamakawa. *Non-Committing Identity Based Encryption: Constructions and Applications*. PKC 2025.（Def 8）
- **[HKNY24]** T. Hiroka, F. Kitagawa, R. Nishimaki, T. Yamakawa. *Robust Combiners and Universal Constructions for Quantum Cryptography*. TCC 2024, App. E.
- **[HMNY21]** T. Hiroka, T. Morimae, R. Nishimaki, T. Yamakawa. *Quantum Encryption with Certified Deletion, Revisited: Public Key, Attribute-Based, and Classical Communication*. ASIACRYPT 2021（arXiv:2105.05393）。Def 4.3 與 footnote 8。
- **[KT18]** F. Kitagawa, K. Tanaka. *Key Dependent Message Security and Receiver Selective Opening Security for Identity-Based Encryption*. PKC 2018.（Def 11：挑戰後禁止對挑戰身分 extraction；footnote 6：IBE 的 KeyGen 永遠可以用 PRF 做成確定性）
- **[MM24]** A. Mehta, A. Müller. *Unclonable Functional Encryption*. ePrint 2024/1683（arXiv:2410.06029）。Def 26（Non-adaptive）與 Remark 1（adaptive）。

> **相關報告**：[09-13 古典／量子盤點報告](./Classical_vs_Quantum_Inventory_and_Challenge_Identity_Query_Rule_Evaluation.md)（§2.2 首次提出分裂後不設限；本文補上引理 A–C 與 KT18 路線的代價）｜[KT18 報告](./KT18_Mechanisms_x_Unclonable_Encryption_RSO_Slot_from_Any_IBE_and_KDM_Two_Slot_Design.md)（§3.5 第 2 點、Q5）｜[延後引理報告](./Deferred_Query_Lemma_Unnecessary_Who_Answers_Queries_Two_Reveals.md)（引理 A 的原型）
