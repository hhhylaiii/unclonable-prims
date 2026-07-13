# AK21 精讀與 PQC 槽替換品總盤點：「最適合的替換品」已有答案，方向需要轉向

> **日期**：2026-07-12
> **任務**：逐頁精讀《Unclonable Encryption, Revisited》（AK21，TCC 2021）全文，抽出「PQC 槽」在證明中真正消耗的介面，並 survey 所有候選替換品，回答「換成什麼最適合」。
> **一句話結論**：**「最適合」的替換品是 RNCE（接收者非承諾加密）——這個答案是對的，對到 Hiroka-Kitagawa-Nishimaki-Yamakawa 已於 2023 年 12 月把它寫進論文附錄（[HKNY24] arXiv:2311.09487, TCC 2024, Appendix E），連 unclonable-IND 的保持都一併證了。** 白板線（重構 UE）的低垂果實已被摘走，repo 既有三份報告均未察覺此事。本文記錄精讀結果、查證證據、剩餘真空格，並給出方向轉向建議。

---

## 0. TL;DR（五點）

1. **AK21 的兩個構造完整拆解完畢**（§1）。私鑰版 = otUE + PRF/OTP 偽金鑰加密；公鑰版 = otUE + 偽隨機密文 SKE + 單金鑰 FE（trojan）。兩個證明的核心技巧相同：把古典密文換成與 otUE 金鑰無關的版本，reveal 階段交出「事後綁定」的偽金鑰，偽金鑰由 B̃/C̃ 用**附在 register 上的共享古典秘密**（隨機性 r／FE.MSK 等）本地重建。
2. **證明真正消耗的介面比 Def 16 弱**（§2）：Def 16 的 FakeGen 無 trapdoor、作用在誠實密文上；但歸約方自己生成所有古典金鑰、可以把任何秘密附給 B̃/C̃——所以**有 trapdoor 的可模糊化（= RNCE 的 Fake/Reveal 形狀）就足夠**。這是白板「換槽」問題的正解。
3. **但這個正解是先行技術**（§3）：[HKNY24] Appendix E 標題就是 *Unclonable PKE from One-Time Unclonable SKE and PKE with Quantum Ciphertexts*，用量子密文 RNCE 填槽、基於 HMNY21（憑證刪除）的技術，Lemma E.2 證明 **unclonable IND-CPA 保持**。repo 的偽金鑰報告（「所有公鑰 UE 都走六種範式、無人不經 FE」）與 7 月 survey（只記這篇的明文擴展）都漏了這個附錄。
4. **剩餘真空格只有三個**（§4）：V1 統一 compiler 的基礎化＋「可模糊化是否必要」的分離問題（可做、規模偏小）；V2 無 trapdoor 的公鑰偽金鑰（Bendlin 牆、高風險）；V3 其他安全目標（CCA／everlasting，價值未驗證）。
5. **建議**（§5）：白板線降級為備援；主軸回到路線甲（unclonable IPFE）——而且今天的發現對路線甲是**大禮**：HKNY24 App E 的 Fake/Reveal 歸約管線正是 P1 構造 UE 歸約段需要的證明模板。
6. **（07-13 追加，§4+）第二輪盤點復活了白板線**：槽位吃得下任何「all-or-nothing」加密的 NC 版。查證後：UE+憑證刪除已被 AKL23 §8.3 做掉、registered×刪除已有 2026-03 論文；但 **RNC-IBE（PKC 2025，應用清單零量子）→ unclonable IBE 這格全空**——「unclonable IBE」一詞文獻中不存在，證明可套 HKNY24 模板，實質工作在 pq 版 RNC-IBE（現有 pq 版僅 poly-ID relaxed）與定義。這是與路線甲並列的第二條可行主軸（W1）。

---

## 1. AK21 精讀筆記

### 1.1 定義域（§3）

| 定義 | 內容 | 備註 |
|---|---|---|
| Def 7 QECM | (Setup, Enc, Dec)，訊息與金鑰古典、密文可量子 | |
| Def 8 | one-time IND 安全 | |
| Def 9/10 | （私鑰/公鑰）語意安全＝many-time IND | 可重用性的正式內容 |
| **Def 11 unclonability** | t-unclonable：(A,B,C) 贏 cloning 實驗機率 ≤ 2^{−n+t} + negl。**挑戰訊息由 challenger 均勻抽樣**；phase 2 金鑰同時亮給 B、C | 搜尋型（弱）概念 |
| **Def 12 unclonable-IND** | A 自選 m₀,m₁；贏率 ≤ 1/2 + negl | 強概念；AK21 自己的構造**不**滿足（當時無人滿足；後來 AKLL22 在 QROM 達成） |
| Def 13 otUE | Def 8 + Def 11 | BL20 conjugate encryption 實例化，資訊論、0.85ⁿ |
| Def 14/15 | 私鑰/公鑰 UE = 語意安全 + unclonability | |
| Remark 4 | 只考慮 one-time cloning（一份密文複製成兩份）；多密文/多複製是後來 CGKNY26 的事 | |

### 1.2 私鑰構造（§4）

**Def 16（偽金鑰性質）**：∃ PT 演算法 FakeGen : CT × M → K，對任意 m：
`{(ct^m ← Enc(k,m), k)} ≈c {(ct⁰ ← Enc(k,0), fk ← FakeGen(ct⁰, m))}`。
注意 FakeGen **只吃 (ct⁰, m)**——無 trapdoor、無秘密狀態，作用在誠實密文上。

**Thm 6 實例化**：Setup 輸出 (k, otp)（otp 與訊息等長、均勻）；`Enc((k,otp), m) = (r, PRF_k(r) ⊕ m ⊕ otp)`；FakeGen 抽 k′、令 `otp′ = ct₂⁰ ⊕ PRF_{k′}(ct₁⁰) ⊕ m`，輸出 (k′, otp′)。**腳註 11：此實例達到完美（資訊論）偽金鑰性質**——關鍵是金鑰裡的 otp 提供了「事後解釋任何密文為任何訊息」的自由度。金鑰內建 OTP＝可模糊性的最便宜來源。

**構造（§4.2）**：`Enc(k_PKE, m)：k_UE ← UE.Setup；ct = (PKE.Enc(k_PKE, k_UE), UE.Enc(k_UE, m))`——KEM-DEM，古典側裝一次性金鑰。

**不可複製性證明（§4.2.1）逐步**：
- Hybrid 1 = 真實 cloning 實驗；Hybrid 2 = 古典側換成 ct⁰ ← Enc(k_PKE, 0)，phase 2 亮出 fk ← FakeGen(ct⁰, k_UE)。
- Claim 1（兩個 hybrid 接近）：歸約 Ã 拿偽金鑰挑戰 (ct*, k*)，自己抽 k_UE、m，包成 cloning 實驗餵 (A,B,C)——real/fake 恰好對應 Hybrid 1/2。
- 最終歸約到 otUE：Ã 收到 otUE 挑戰密文 ct^m；自己生成 k_PKE 與 ct⁰（**與 k_UE 無關，所以做得到**）；跑 A 得二分態；**抽 FakeGen 的隨機性 r，附到 B、C 兩個 register 上**；phase 2 otUE challenger 亮出 k_UE，B̃/C̃ 各自以同一 r 計算 fk = FakeGen(ct⁰, k_UE; r)——**兩人得到同一把 fk**，完美模擬 Hybrid 2。
- **Thm 7**：UE t-unclonable ⇒ 構造 t-unclonable（安全度無損傳遞）；Cor 2：n·log₂(1+1/√2)。

### 1.3 公鑰構造（§5）

**三個工具**：otUE；**偽隨機密文** pq-SKE（密文 ≈ 均勻串，由 OWF 得到，腳註 13 的 `(r, PRF(k,r)⊕x)`）；**單金鑰公鑰 FE**（§2.4 的 SIM 式定義：先挑戰訊息、後一次電路查詢；**可從任何 pq-PKE 構造 [SS10, GVW12]**——這就是 Thm 2 假設歸到 pq-PKE 的原因）。

**Trojan 結構**：
- Setup：抽**均勻隨機串** ct；發 FE.sk ← KeyGen(MSK, F[ct])，其中 `F[ct](b, K, m) = SKE.Dec(K, ct) if b=0；m otherwise`。pk = FE.mpk、sk = FE.sk。
- Enc(pk, m)：`(FE.Enc(mpk, (1, ⊥, k_UE)), UE.Enc(k_UE, m))`——正常模式走 b=1 直接吐 k_UE。
- 證明：Hybrid 2 把 Setup 裡的隨機串換成 `SKE.Enc(k_SKE, k_UE)`（偽隨機密文性質）；Hybrid 3 把 ct₁ 換成 `FE.Enc(mpk, (0, k_SKE, ⊥))`（FE 安全性；合法因為兩個訊息在 F[ct] 下同值 k_UE）。此後 ct₁ 與 k_UE 無關；最終歸約中 B̃/C̃ 拿 (FE.MSK, k_SKE, r) 共享秘密，reveal 後本地算 `ct = SKE.Enc(k_SKE, k_UE; r)`、`FE.sk ← KeyGen(MSK, F[ct])` 交給 B/C。
- **Thm 8**：UE t-unclonable ⇒ PBKUE t-unclonable。

### 1.4 其餘章節（與換槽無關，簡記）

§6：把 BL20 的分析推廣到 real-orthogonal monogamy games（otUE 槽的一般化——量子核心可換成任何此類 MoE 遊戲的編碼）；0.71ⁿ 通用複製攻擊與對 conjugate encryption 的 0.75ⁿ 攻擊。§7：unclonable-IND UE ⇒ 點函數複製保護（計算正確性版）。

---

## 2. 證明真正消耗的介面（本文歸納），與關鍵觀察

把兩個證明並排，古典槽 ε 需要的性質恰好四條：

- **E1（模糊化不可區分）**：`(aux_pub, ct_真, key_真) ≈c (aux_pub, ct_假, key_假)`，對 QPT 整體成立（aux_pub = 公鑰等公開資訊）。
- **E2（訊息無關生成）**：ct_假（連同全部古典秘密）可在**不知道 m**（= k_UE）時生成。
- **E3（事後綁定金鑰）**：key_假 可由確定性 PT 演算法從（生成時固定的共享古典狀態 σ, m）算出——B 與 C 各自算、得同一把。
- **E4（後量子）**：E1 對 QPT 成立即可；量子側的複製界完全由 otUE 承擔（Thm 7/8 是無損傳遞）。

**關鍵觀察：Def 16 比 E1–E3 強。** Def 16 要求 FakeGen 無 trapdoor、作用在誠實密文 ct⁰ 上；但 E2/E3 允許 ct_假 是**模擬密文**、key_假 用**秘密 trapdoor** 算——因為歸約方自己跑 Setup、想附什麼秘密給 B̃/C̃ 都可以。「有 trapdoor 的模擬式可模糊化」正是 **RNCE（receiver non-committing encryption）的 Fake/Reveal 介面**。推論：
1. 私鑰槽：AK21 的 PRF+OTP 其實是「無 trapdoor RNCE」的奢侈版（perfect 版）；
2. 公鑰槽：不需要 FE——公鑰 RNCE 就夠，而 RNCE 可從 IND-CPA PKE 構造（假設仍最優）；
3. 這同時解釋了偽金鑰報告裡的困惑：公鑰情形卡住的是「無 trapdoor + 誠實密文」這個 receiver-deniability 式的過強要求（Bendlin 牆），而證明根本不需要它。

---

## 3. 查證結果：觀察正確，但已是先行技術

### 3.1 [HKNY24] Appendix E（本次調查的核心發現）

Hiroka, Kitagawa, Nishimaki, Yamakawa，《Robust Combiners and Universal Constructions for Quantum Cryptography》，arXiv:2311.09487（2023-12），TCC 2024。**Appendix E：*Unclonable PKE from One-Time Unclonable SKE and PKE with Quantum Ciphertexts***：

- **動機**（原文）：AK21 的 pk-UE 用了古典密文 PKE，「their security proof relies on the existence of OWFs, but it is an open problem whether PKE with quantum ciphertexts implies OWFs」——所以他們為量子密文 PKE 重做一個不經 OWF 的版本，「for the reader's convenience」。
- **Def E.1**：量子密文 RNCE，六演算法 (Setup, KeyGen, Enc, Dec, **Fake, Reveal**)；Fake(pk) → (假密文 CT̃, aux)；Reveal(pk, MSK, aux, m) → 假金鑰 s̃k；安全性 = `(CT, sk) ≈ (CT̃, s̃k)`——**正是 §2 的 E1–E3 介面**。並註明「in the same way as [KNTY19, HKM+23], we can construct RNCE with quantum ciphertexts from PKE with quantum ciphertexts」（RNCE ⇐ PKE 泛型成立；KNTY19 精確出處待核，HMNY21 用同一事實）。
- **構造**：`Enc(pk, m) = (NCE.Enc(nce.pk, ske.sk), SKE.Enc(ske.sk, m))`——AK21 的 KEM-DEM 骨架，古典槽換成 RNCE。
- **Lemma E.2：構造滿足 unclonable IND-CPA**（**強概念**，A 自選 m₀,m₁）。Hyb₁ 把 (nce.CT, nce.sk) 換成 (Fake, Reveal(…, ske.sk))；Prop E.3 歸約到 RNC 安全性；Prop E.4 歸約到底層 one-time unclonable IND-CPA SKE——歸約中 Ã 把 **aux 與 nce.MSK 附到 B/C register**，reveal 後 B̃/C̃ 各自跑 Reveal 得同一把假金鑰。與 §1.2 的 AK21 管線逐步同構。
- 意涵：(i) 「公鑰 UE 不經 FE、靠可模糊化」**已存在**；(ii) 「compiler 保 unclonable-IND」**已寫下**（模組化：給它 one-time unclonable-IND SKE 就吐 unclonable-IND PKE；one-time 端 QROM 有 AKLL22，標準模型仍開放——那是大魔王，不是槽的問題）；(iii) 連量子密文槽（microcrypt 相容版）都覆蓋了。

### 3.2 對 repo 既有報告的更正

| 報告 | 原判斷 | 更正 |
|---|---|---|
| 偽金鑰報告 | 「所有公鑰 UE 走六種範式；無人不經 FE 以可模糊化論證做公鑰 UE」 | **第七範式存在**：HKNY24 App E 的 RNCE 路線（2023-12）。「偽金鑰性質沒有公鑰形式化」字面上仍真，但其**實質內容**（trapdoored 版）已被做掉 |
| 7 月 survey | HKNY24 只記為「明文擴展」（§2.2）；2311.09487 只記為「量子密文 RNCE 可從 PKE 構造」（§2.5） | 兩條記錄指向**同一篇論文**，且其 App E 直接命中白板線 |
| 白板報告 v2 | R1（公鑰偽金鑰不經 FE）標為「5 年 open、主推」；R3（unclonable-IND 版 compiler）標為「檢查點」 | R1 實質關閉、R3 已寫下（均為 HKNY24 App E）。R2 的「統一刻畫」部分存活（見 V1） |

### 3.3 同配方族譜（為什麼這格被 NTT 團隊佔走是自然的）

`量子編碼 + 古典可模糊化元件` 這個配方在他們手上已連做三次：HMNY21（BB84 + RNCE ⇒ 憑證刪除 PKE，TCC 2021——與 AK21 同年同會）→ HKNY24 App E（otUE + RNCE ⇒ unclonable PKE）→ 其 SKL 系列（KMY25 等）。survey 報告「SKL 流派一原語一論文」的觀察在 UE 這格同樣成立，只是成果藏在附錄。

---

## 4. PQC 槽候選品總矩陣（最終版）

**問「什麼最適合」的三層答案：**

### 已佔格子（概念層）

| 槽內容物 | 得到 | 出處 | 備註 |
|---|---|---|---|
| PRF + OTP（無 trapdoor 完美偽金鑰） | 可重用私鑰 UE（搜尋型） | AK21 §4 | 假設最優（OWF） |
| 偽隨密文 SKE + 單金鑰 FE（trojan） | 公鑰 UE（搜尋型） | AK21 §5 | 假設最優（PKE） |
| **RNCE（古典或量子密文）** | **公鑰 UE，且保 unclonable-IND** | **HKNY24 App E** | **概念上的正解；本文原以為的新定理** |
| 最弱化（uncloneable bit + SKE） | many-time UE（tight） | BG26b | 假設軸完成式 |
| 抗合謀原語 compiler | 多複製安全 UE | CGKNY26 | CRYPTO 2026 |

### 實例化層（薄，只夠當附註或效率小節）

- **equivocal LWE**（ePrint 2026/1293）／lattice RNCE：代數直構、具體效率——「槽的更好實作」，非新概念。
- **Somewhere equivocal encryption**（HJO+16，OWF）：AK21 腳註 5 已點名其私鑰方案是特例；唯一附加值是金鑰長度可短於訊息（PRF+OTP 的 otp 與訊息等長）——效率註腳。

### 真空格子（僅剩這些）

| # | 內容 | 評估 |
|---|---|---|
| **V1** | **compiler 的基礎化**：正式陳述「otUE + X ⇒ 可重用 UE」、刻畫最小 X（Def 16 ⇐ RNCE ⇐ ? 的蘊含分離圖，含 Bendlin 牆的精確定位）＋**必要性問題：純 IND-CPA 的 ε 是否足夠？**（找出可模糊化的黑盒分離／反例方案，或證明更弱充分條件） | 真空、可做；定位類似 KY25《Foundations of SDE》的 systematization。單獨成碩論偏薄，適合當一章。必要性分離是其中唯一的硬技術題，難度未知 |
| V2 | 無 trapdoor、誠實密文的**公鑰**偽金鑰（Def 16 的字面公鑰化） | 正面撞 Bendlin 不可能性；可能需要量子金鑰/互動；高風險，且 V1 做完會順帶回答「為什麼不需要它」 |
| V3 | 其他安全目標過 compiler：CCA-unclonability、everlasting 變體 | 無人做，但價值與難度均未驗證；不建議當主軸 |

---

## 4+. 第二輪盤點（2026-07-13 追加）：除了 RNCE，還有什麼能放進槽位

> 問題：RNCE 這格被佔了，但槽位介面 E1–E4 容納的是一整族元件——還有哪些「放得進去、且有可能做出來」的？

### 4+.1 結構規則：槽位的邊界在哪裡

槽位承載的是 k_UE 的**全有全無**傳遞：合法金鑰持有者拿到完整 k_UE、其他人拿到零。所以：

- **放得進去**：任何 all-or-nothing 型加密——PKE、IBE、ABE、predicate（payload-hiding）、broadcast、registered 變體——只要它有 NC/可模糊化版本（E1–E3），compiler 照走，輸出「該存取結構的 unclonable 版」。KEM-DEM 結構存活，因為金鑰要嘛解出全部 k_UE 要嘛什麼都沒有。
- **放不進去**：細粒度 FE/IPFE——功能金鑰只能拿到 f(k_UE)，功能性斷裂（v2 報告 §4 的觀察）。那是路線甲的領土。

因此第二輪候選空間 = {NC 版的各種存取結構加密} ∪ {給輸出 UE 疊加額外特性的組合}。

### 4+.2 本輪查證：又三格已被佔

| 格子 | 佔據者 | 證據 |
|---|---|---|
| UE ＋ 憑證刪除（同一方案雙保證） | **[AKL23] §8.3** "Construction of Unclonable Encryption with Certified Deletion"（asymmetric cloning games / deletion games，§8.2） | 本地 PDF 目錄直接確認——已精讀論文中的未注意章節，meeting 前務必回讀 §8 |
| Registered × 量子刪除 | arXiv:2603.07646《Registered ABE with Publicly Verifiable Certified Deletion, Everlasting Security, and More》（2026-03） | registered+quantum 交叉區已有人開採；其 "and More" 是否含 unclonability **待讀** |
| IBE 類的弱不可複製性 | [KN23]（TCC 2023）one-out-of-many unclonable predicate encryption（LWE） | 弱化概念（一對多）；AK21 型完整 cloning game 的 IBE 版**未**被它覆蓋 |

### 4+.3 仍開放的候選（按 可行性×新穎性 排序）

| # | 槽內容物 → 輸出 | 材料現況 | 評估 |
|---|---|---|---|
| **W1 ★** | **RNC-IBE → Unclonable IBE**：`ct = (RNCIBE.Enc(mpk, id, k_UE), otUE.Enc(k_UE, m))` | RNC-IBE 剛出爐（Goyal-Kitagawa-Koppula-Nishimaki-Rajasree-Yamakawa，**PKC 2025**）；查證其應用清單**只有 incompressible IBE、零量子應用**；「unclonable IBE」一詞文獻中不存在 | **首選**。證明 = HKNY24 App E 模板 + IBE 金鑰查詢 hybrid。實質工作（不是白撿）：(a) RNC-IBE 主構造用 bilinear（**非 pq**），pq 版是 relaxed（DDH/LWE、**多項式大小 identity space**）——第一定理接受 poly-ID，或自造 LWE 上 selective RNC-IBE（GPV/ABB＋可模糊化）當技術核心；(b) unclonable IBE 的 cloning game 定義（金鑰查詢在 split 前後的給法）；(c) 與 KN23 一對多概念的分離/比較。競速風險：NTT+Goyal 產線自己補量子應用 |
| W2 | NC-RFE / equivocal LWE → **Registered unclonable encryption** | Sarkar ePrint 2026/1293（AFRICACRYPT 2026）元件現成 | 去信任機構＋不可複製的組合尚空，但 2603.07646 顯示鄰域已有人；registered 機制本身複雜度高。次選 |
| W3 | SKL-PKE ＋ NC 性質 → **UE with secure key leasing** | KMY25 框架（EUROCRYPT 2025）現成；「SKL 方案的 NC 化」需自行驗證 | 密文不可複製 × 金鑰可租回，雙量子生命週期的乘積題；SKL 產線風格，故事清楚。中風險 |
| W4 | NC＋CCA 古典組合 → **CCA-secure UE** | 古典 NCE+CCA 已知 | 定義＋compiler，低風險低新穎；附章料 |
| W5 | 統計性接收者可模糊化 → **everlasting unclonability** | 存在性未知（統計 RNCE 的 PKE 版可能撞牆） | 探索級；一週試探再定 |
| W6 | equivocal LWE／HPS／SEE 直構 | 全部現成 | 純效率實例化；註腳料 |
| — | NC-ABE → unclonable ABE | **NC-ABE 尚不存在** | W1 的自然後續：先造 NC-ABE 再過 compiler，工作量大，當延伸章 |

### 4+.4 W1 的定理形狀（供 meeting 討論）

> **定理（候選）**：若存在 pq 的 RNC-IBE（selective-ID 即可起步）與 one-time unclonable encryption，則存在 unclonable IBE：語意安全承自 IBE、t-unclonability／unclonable-IND 無損承自 otUE，且對任意多的非挑戰身分金鑰查詢安全。
>
> 一般化版本（把 W1/W2 收進同一條）：「otUE ＋ 任何 NC 化的 all-or-nothing 加密 ⇒ 該原語的 unclonable 版」——這就把 V1 的 compiler 基礎化與 W 系列構造統一成一篇論文的骨架：一條主定理、W1 當主要實例、W2/W4 當 corollary、V1 的必要性分離當理論章。

---

## 5. 方向建議

### 5.1 對「換什麼最適合」的直接回答

工程正解是 **RNCE**——判斷依據不是品味而是事實：領域裡最強的構造團隊已經選了它並寫進 TCC 論文。這件事同時意味著：**白板線（重構 UE）作為碩論主軸已不成立**，剩餘空間（V1–V3）只夠支撐附章或高風險賭注。

### 5.2 這個發現對路線甲反而是禮物

路線甲 P1（`ct = (IPFE.Enc(x+r), UE.Enc(k,r))`）的 UE 歸約段，需要的正是「古典秘密附進 B/C register、reveal 後本地重建金鑰」的管線——**HKNY24 App E 的 Lemma E.2 就是現成的證明模板**（把「NCE 金鑰」換成「sk_y 的古典半邊」）。精讀 AK21 + HKNY24 的投資直接轉化為 P1 的證明工具箱；且 P1 若需要古典側可模糊化（公鑰化或強安全概念時），RNCE/equivocal-LWE 都是現成元件。

### 5.3 給老師的匯報結構（meeting 定稿版）

1. **情報**：白板配方（otUE + PQC）我精讀完了；「換槽」的正解是 RNCE，但 HKNY24 在 2023 年 12 月的附錄已經做了，連 unclonable-IND 保持都證了——我們三份 survey 都漏了這個附錄（它藏在 combiners 論文裡、標題不含 unclonable 關鍵詞）。
2. **白板線剩什麼**：V1（compiler 基礎化＋可模糊化必要性分離）可當一章；V2 高風險。
3. **建議**：主軸放路線甲（unclonable IPFE，經上次確認仍零競爭），白板線的精讀成果直接成為 P1 的證明模板；V1 若老師覺得有價值，可作為論文第二章（「UE compiler 的介面刻畫」），與 P1 共用全部定義與工具。
4. **問老師**：(a) V1 這種 systematization 型貢獻在您看來夠不夠碩論的一章？(b) 是否同意把主力轉回路線甲、以 HKNY24 模板起跑 P1 的正式定義與定理陳述？

### 5.4 誠實的風險備註

- 本文「V1 無人做」的判斷基於 2026-07-12 的檢索；動筆前需以 ePrint 全文檢索 +「引用 HKNY24 的論文清單」再確認一次。
- HKNY24 App E 我讀的是 arXiv v2（2023-12-05）；TCC 2024 正式版是否有增改（例如把 App E 擴成正文、加古典密文版陳述）需要核對會議版。
- V1 的必要性分離（「IND-CPA 是否足夠」）技術難度未知：直覺上分離存在（reveal 階段使 IND-CPA 失效、而 hybrid 證明繞不開可模糊化），但構造反例方案或 oracle 分離都不是模板化工作——這是 V1 從 SoK 升級為 research paper 的關鍵賭注，值得花一週試探再定。

---

## 6. 引用

- **[AK21]** Ananth, Kaleoglu. *Unclonable Encryption, Revisited*. TCC 2021. ePrint 2021/412 / arXiv:2103.15009.（全文精讀：Def 7–16、Thm 6–8、§4.2.1/§5.1.1 證明、§6 ROMG、§7 copy-protection）
- **[HKNY24]** Hiroka, Kitagawa, Nishimaki, Yamakawa. *Robust Combiners and Universal Constructions for Quantum Cryptography*. arXiv:2311.09487 (v2, 2023-12), TCC 2024. **Appendix E**（本次發現的關鍵）；另含 §7 UE combiner、§8 明文擴展（survey 原記錄）。
- **[HMNY21]** Hiroka, Morimae, Nishimaki, Yamakawa. *Quantum Encryption with Certified Deletion, Revisited*. TCC 2021.（RNCE+BB84 配方的先例；HKNY24 App E 自承基於其技術）
- **[AKLL22]** Ananth, Kaleoglu, Li, Liu, Zhandry. *On the Feasibility of Unclonable Encryption, and More*. CRYPTO 2022. arXiv:2207.06589.（unclonable-IND 僅 QROM——compiler 的 one-time 原料現況）
- **[SS10, GVW12]** 單金鑰 FE ⇐ PKE（AK21 §2.4 Instantiations）。
- **[HJO+16]** somewhere equivocal encryption（AK21 腳註 5）。
- **[BG26b] [CGKNY26] [KY25]** 見 7 月 survey 文獻表。
- KNTY19（RNCE ⇐ PKE 的出處，HKNY24 引用）：**待核**——讀 HKNY24 參考文獻列表即可解決。
