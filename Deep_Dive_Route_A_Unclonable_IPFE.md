# 路線甲深度分析：從 IPFE 到 Unclonable IPFE——技術路徑、難點地圖與知識補完清單

> **日期**：2026-07-10
> **前提**：已決定主攻路線甲（把 MM24 升級階梯的底層換成 IPFE，最終 lift 成 unclonable 版本）。
> **本文目的**：(1) 把「Unclonable IPFE for quantum messages」這個目標的語義釘死；(2) 逐條盤點 MM24 三個定理對底層元件的介面需求，找出換成 IPFE 之後哪裡會斷；(3) 給出三條具體技術路徑與各自的難點；(4) 列出需要補足的知識與驗收標準。
> **依據**：MM24 全文精讀（含 Section 5 逐行）、BBC26 v2 原文、2026-07 survey 報告的文獻盤點。

---

## 0. TL;DR：三個最重要的發現

1. **MM24 的 lifting 定理（Thm 7）消耗 universality，對受限函數類「不適用 as stated」。** 它要求底層 QFE 支援的不是 C 本身，而是一個包裝電路 U_{(C,a,b)}——內含 Pauli 修正、測量、條件分支、以及執行「作為輸入傳進來的解密電路 C_Dec」。就算你只想要 inner product 的 C，包裝後的電路類也遠超出 IPFE。**「換底層」的真正工作量在重建 lifting，而不是把 TwoFE 換成 IPFE。**
2. **存在一條繞過整座階梯的直接合成路徑**（本文路徑 P1）：`ct = (IPFE.Enc(x + r), UE.Enc(k, r))`，把不可複製性externalize到一個一次性 UE 實例上。歸約出奇地乾淨（見 §3.1），有機會做成一個模組化 compiler 定理：「任何 unclonable-IND 安全的 UE × 任何 IND 安全的 IPFE ⇒ secret-key Unclonable IPFE」。這是建議的第一攻擊點。
3. **Clifford 直構（之前討論的「QOTP + IPFE 發 pad 修正」想法）存在一個致命的可逆性洩漏攻擊**：Clifford 的 pad 更新映射 L_C 可逆，解密者可以先解古典部分、逆推原始 pad、直接拿回 ρ 本身。這個攻擊解釋了為什麼 QHE 能做 Clifford-free 而 QFE 不能，本身就是一個可寫進論文的 observation（§3.3）。

---

## 1. 先釘死目標語義：三種「Unclonable IPFE」

「Unclonable IPFE for quantum messages」這個短語其實混著三個不同的目標，必須先選定：

### (I) 古典向量訊息、量子密文、古典輸出 ⟨x, y⟩ —— 建議主目標

- 訊息 x ∈ Z_q^n 是古典向量；函數金鑰對應 y ∈ Z_q^n；解密輸出 ⟨x, y⟩
- 「量子」體現在**密文**：密文含量子態，使兩個分裂的對手無法同時解密——這正是 UE 的精神（BL20/AK21 的訊息也是古典的，量子性在密文）
- 正確名稱是 **Unclonable IPFE**（不需要 "for quantum messages" 後綴）
- 安全遊戲：MM24 Def 26 的直接特化——對手選 x₀, x₁ 與向量 y_B, y_C，splitting 後 B、C 各拿獨立生成的 sk_{y_B}, sk_{y_C}，兩人**同時**猜對 b 的機率 ≤ 1/2 + negl。注意 Def 26 **不要求 admissible queries**（MM24 Lemma 5 特別指出這點）：允許 ⟨x₀,y⟩ ≠ ⟨x₁,y⟩ 的金鑰，單人可以贏，但兩人同贏被 no-cloning 擋住

### (II) 量子訊息、函數類為受限酉類（Clifford），輸出量子態 —— 第二章目標

- 訊息是量子態 ρ_m，函數類是 Clifford 電路（量子世界裡「線性映射」的正統類比：Heisenberg 圖像下 Clifford 把 Pauli 線性映到 Pauli），輸出 C(ρ_m)
- 這才是字面意義的「QFE for a restricted class on quantum messages」
- 有實質的定義性貢獻空間，但有 §3.3 的陷阱要繞

### (III) 量子訊息、輸出 Tr(Mρ) 型的「內積」 —— 排除

- 若把「內積」理解為觀測量的期望值 Tr(Mρ)，**單份密文根本無法解出**——期望值需要多拷貝（shadow tomography 的領域），單次解密只能給一個測量樣本
- 除非把 correctness 改成抽樣語義（解密輸出一個按 Born rule 分布的樣本），否則定義自身就崩了。抽樣語義是可以做的，但 admissibility（MM24 Def 25 的 trace distance 條件）會變得很難刻畫，不建議當主目標

**建議**：論文主體做 (I)，第二部分（時間允許）做 (II)。(I) 對外的賣點是「首個 unclonable 的受限類 FE、具體、基於標準工具」；(II) 的賣點是「quantum message 上受限類 QFE 的首個正式處理」。

---

## 2. MM24 介面盤點：每一層定理到底消耗什麼

換元件之前，先把每個定理的「輸入規格」列清楚。以下每一項都是精讀原文後的盤點，標注了換成 IPFE 後的狀態。

### Thm 5（OneQFE，§4.1）

| 需求 | 內容 | 換 IPFE 後 |
|------|------|-----------|
| 古典元件 | **adaptive SIM-secure** 的 IdFE（恆等函數 FE，GVW12 實例化） | 見難點 D5：IPFE 的 SIM-security 有天花板 |
| 量子技巧 | QOTP + 模擬器用 EPR pairs 延遲teleport | 與函數類無關，可保留 |
| 限制 | 電路 C 在 **Enc 時固定**（加密者自己算 C(ρ_m)） | 這正是 FE 該解決的；OneQFE 只是墊腳石 |

### Thm 6（poly-sized QFE，§4.2）

| 需求 | 內容 | 換 IPFE 後 |
|------|------|-----------|
| 古典元件 | adaptive SIM-secure 的 TwoFE × l 個 instance（每個管 universal circuit 輸入的一個 bit） | 若函數類受限，universal circuit 不必要，整段可重新設計（老師第二點）|
| 量子元件 | OneQFE × 2（管量子輸入與 offline 部分）+ **DQRE**（BY22 QGC，decomposability 是靈魂） | 受限類需要多少 decomposability 是核心研究問題 |
| 結構前提 | **universal circuit** U(C, ρ_m) = C(ρ_m)，C 的描述逐 bit 餵進 TwoFE | IPFE 的 y 有代數結構，逐 bit 選擇是浪費；但拆掉它就要重寫全部 hybrid |

### Thm 7（UFE lifting，§5.2）——最關鍵、之前低估的一層

定理原文的前提是「any single-query QFE scheme for n-qubit messages **and universal circuits**」。逐條拆開它到底用了 universality 的哪些部分：

**(a) 包裝電路 U_{(C,a,b)} 的能力需求。** KeyGen 不是直接對 C 發金鑰，而是對包裝電路 U_{(C,a,b)} 發金鑰，其內部要做：
1. 讀 flag bit f 並**條件分支**（f=0 走誠實模式輸出 C(ρ_m0)；f=1 走證明模式）
2. 對輸入中的 |dk₀⟩, |dk₁⟩ 施 **Pauli 修正** X^a Z^b（teleportation 解碼；(a,b) 是 KeyGen 時 hardwire 進電路的隨機串——這就是「可變解密金鑰」機制的來源）
3. **計算基測量**前 λ 個 qubit 並檢查是否全零（teleport 成功驗證）
4. 把輸入中的古典串 **C_Dec 解讀為電路描述並執行它**：U(C_Dec, |dk*⟩, ρ_UE) = b
5. 最後才算 C(ρ_mb)

第 4 步是赤裸裸的 universal circuit 呼叫。**就算 C 是 inner product，U_{(C,a,b)} 也不是**——它屬於「Clifford 修正 + 測量 + 分支 + 通用電路模擬 + C」的複合類。

**(b) 訊息空間需求。** 誠實加密把訊息 pad 成 (ρ_m ⊗ |0…0⟩ ⊗ |0⟩)，證明中的 Enc* 則要加密 (ρ_m0 ⊗ ρ_m1 ⊗ σ₀^A ⊗ σ₁^A ⊗ ρ_UE ⊗ C_Dec ⊗ |1⟩)——包含 **EPR halves 與 UE 密文這些真量子態**。所以底層方案必須能加密量子態，純古典 IPFE 不能直接 slot in，即使目標語義是 (I)。

**(c) 安全概念需求。** 歸約用的是 **2-player single-query IND-security**（Def 30，由 single-query IND 蘊含，Appendix A.2）＋ Def 25 admissibility 的「單一量子態 σ 定義兩個 challenge messages」特例＋「歸約後 A* 不再與訊息糾纏」的論證。換底層後這些 plumbing 全部要重走。

**(d) UEQ 元件的介面需求**（lifting 只要求其存在，不顯式使用——universal 性質）：
- one-time、單 bit 訊息、**unclonable-indistinguishable**（Def 18）
- **量子解密金鑰** |dk⟩，且 KeyGen 用同一 randomness 跑兩次可得兩份 |dk⟩
- |dk⟩ 可被 **teleport**（純態、尺寸 l(λ) 有界）
- **解密電路有 poly 大小的古典描述** s(λ)（因為 C_Dec 要塞進訊息、被 universal circuit 執行）
- [AKY24] 滿足全部條件

**結論**：對 IPFE 類，Thm 7 不適用 as stated。要嘛為「包裝後的結構化類」直接造 QFE（路徑 P2），要嘛換一個不吃 universality 的 lifting（路徑 P1）。

---

## 3. 三條技術路徑

### 3.1 路徑 P1（建議首攻）：直接合成——IPFE(x+r) ⊗ UE(r)

**構造 sketch**（secret-key 版）：

```
Setup:    (ipfe.mpk, ipfe.msk) ← IPFE.Setup;  K ← PRF key
KeyGen(y): sk_y = (IPFE.KeyGen(ipfe.msk, y), K)
Enc(x):   取新鮮 id；k_id = PRF_K(id)；r ← Z_q^n 均勻
          輸出 ct = ( id, IPFE.Enc(ipfe.mpk, x + r), UE.Enc(k_id, r) )
Dec(sk_y, ct): 由 K 導出 k_id；r = UE.Dec(k_id, ρ_UE)
          輸出 IPFE.Dec(ipfe.sk_y, ct_cl) − ⟨r, y⟩ = ⟨x, y⟩
```

想法：把 x 用一次性 pad r 蓋住丟給古典 IPFE（古典部分可以隨便複製，不影響安全），把「不可複製性」完全externalize到 r 的 UE 加密上。任何人要算 ⟨x,y⟩ 都必須先從量子部分解出 r——兩個分裂的對手無法同時做到，這正是 UE 的保證。

**歸約 shape（為什麼我認為它乾淨）**：Unclonable-IND 遊戲中對手選 x₀, x₁。取 r₀ 均勻、令 **r₁ = r₀ + x₀ − x₁**，則 x₀ + r₀ = x₁ + r₁，古典 IPFE 部分在兩個世界**完全同分布**；於是「加密 x₀ vs x₁」恰好等價於「UE 加密 r₀ vs r₁」。歸約把 (r₀, r₁) 當作 UE unclonable-IND 遊戲的挑戰訊息對（AK21 型定義允許對手選訊息、且金鑰在 splitting 後交給 B 和 C——與我們 sk_y 含 UE 金鑰的設計完全對齊），B*/C* 拿到 UE 金鑰後即可在本地補齊 sk_y 的古典半邊。整個歸約不需要任何關於「線性後處理」的新引理。

**三個坑（也是論文要處理的實質內容）**：

- **坑 1：金鑰分發時序失配。** UE 金鑰 k_id 是 per-ciphertext 的，但 FE 的函數金鑰在加密前就發出。上面用 PRF 解掉（AK21 可重用構造的同款技巧——他們的 otp′ = θ ⊕ PRF_k(r) 就是這個模式），代價是 **KeyGen 要把 PRF 主鑰 K 放進每把 sk_y** → 任何單一金鑰持有者可以解出所有密文的 r，於是他學到的是 x+r 與 r，即 ⟨x,y⟩ 之外還學到…注意：他學到 x + r 與 r ⇒ 學到 x！**這是天真版本的漏洞**：K 必須換成「只夠導出 UE 解密能力、不夠重建 r 全向量」的東西——但 UE.Dec 本來就輸出整個 r。修正方向（三選一，都是可做的研究決策）：
  (i) 接受它：把功能性定義成「金鑰持有者本來就可學 x」？不行，那 FE 就沒意義。
  (ii) 讓 IPFE 那邊不是加密 x+r 而是加密 x，把 r 融進 **IPFE 金鑰**：sk_y 對應向量 y 但帶偏移 ⟨r,·⟩……r 是 per-ct 的，金鑰做不到。
  (iii) **正解候選**：UE 加密的不是 r 而是一個短種子 s，r = G(s)（PRG）；且 UE.Dec 的輸出不直接給 r，而是——不行，拿到 s 一樣重建 r。**根本張力：任何讓合法解密者「算出 ⟨r,y⟩」的機制，若給出的資訊超過 ⟨r,y⟩ 本身，就洩漏 x 的額外資訊。** 真正的正解是讓量子部分輸出**恰好 ⟨r,y⟩**——即量子部分本身要有「函數式解密」結構，例如：UE 加密的物件改為 r，但 sk_y 內含的不是完整 UE 金鑰、而是「只能解出 ⟨r,y⟩ 的受限金鑰」——這就把問題遞迴成「r 上的（一次性、資訊論）IPFE」！好消息：**一次性、對稱金鑰、資訊論安全的 IPFE 是存在且簡單的**（pad-based：金鑰方持有 r 的線性函數值即可；或用 ⟨r,y⟩ = 對 r 的線性雜湊）。具體地：Enc 時對每個「將來可能的 y」……y 空間指數大，不能枚舉——需要用結構：令量子部分為 UE.Enc(k_id, s)、r = G(s)，sk_y 給 (ipfe.sk_y, K)，解密者解出 s 後重建 r 並算 ⟨r,y⟩——又回到洩漏 x。**誠實結論：坑 1 是 P1 最硬的一塊**，目前最乾淨的出路是**把功能性弱化為「單金鑰或有界金鑰 per 密文」，或接受解密者學到 r（等價於：這是『unclonable one-time-programmable IPFE』——每份密文只保護到第一次解密前）**；或者第四個方向：用 **FE 本身遞迴**（量子部分放 IPFE₂.Enc(r) 而 sk_y 含 IPFE₂.KeyGen(y)，UE 只包 IPFE₂ 的解密能力）——這開始長得像 MM24 的 trojan 結構，值得跟老師討論。⚠️ 把此坑寫清楚本身就是論文的 technical core。
- **坑 2：UE 選型（長訊息、unclonable-IND、可實例化）。** 需要對 n·⌈log q⌉ bit 訊息的 unclonable-IND UE。選項：AKLL22（QROM、BB84 態——與 AKL23 cloning games 銜接最順）；BBC26 不可複製位元 + HKNY24 明文擴展 / BG26b boosting（假設最弱，但見 §3.2 的效率警告）；AKY24（標準模型但量子金鑰 → sk_y 變量子態）。建議第一個定理用 QROM 版（故事最乾淨），把「換 UE 實例化」做成表格化的 corollary。
- **坑 3：q-ary 對齊。** UE 文獻是 bit 訊息；r ∈ Z_q^n 需要逐符號編碼與 hybrid，unclonable-IND 的 multi-bit hybrid 論證在 UE 這邊**不是**標準的（cloning 遊戲下 hybrid argument 出了名的麻煩——survey 裡 CGKNY26 整篇就在處理這類問題），需要檢查或引用現成的 multi-bit 擴展（HKNY24）。

**P1 的產出形態**：一個 compiler 定理 +2–3 個實例化 corollary + 與 MM24 的效率對照表（密文從 garbled universal circuit 縮到 O(n log q) 古典 + O(n log q) qubit）。**即使坑 1 最後只做出單金鑰版本，作為「首個 unclonable 受限類 FE」仍然成立。**

### 3.2 路徑 P2（正宗路線甲）：為包裝類 W(IPFE) 重建階梯

目標：直接構造 QFE for 結構化類 W = {U_{(y,a,b)} : y ∈ Z_q^n, a,b ∈ {0,1}^*}，其中 U_{(y,a,b)} 是 §2(a) 的包裝電路、payload 換成 inner product。然後 Thm 7 的證明對 W 重跑。

**可以簡化的部分（老師第二點的精確化）**：
- 包裝電路的骨架是固定的：`Pauli 修正（Clifford）→ 計算基測量 → 分支 → C_Dec → 內積`。除了 C_Dec 那步，全部落在 **Clifford + measurement** 模型內——BY22 的 QGC 對 C+M 電路有專門的輕量處理（不需要 T-gate gadget），garbling 這一段的成本可以大幅壓低
- **C_Dec 可否 hardcode？** 原構造把 C_Dec 當訊息輸入，是因為 lifting 要對「任意 UEQ」universal。若我們**固定一個具體的 UEQ 方案**（如 AKY24），C_Dec 就是一個已知的固定電路，可以直接編進包裝電路，不再需要 universal circuit 模擬。檢查點：Hybrid 1 的歸約中 C_Dec 由歸約自己放入訊息——hardcode 後歸約改為選用對應的包裝電路，admissibility 等式 U(ρ*_m0) = C(ρ_mb) = U(ρ*_m1) 形式不變，**初步判斷成立**，但要逐行驗證（這是一個具體、可在兩週內完成的驗證任務，適合當 Project A 的第一個里程碑）
- universal circuit 的逐 bit TwoFE 選擇機制：y 是 KeyGen 時才定的，但在 W 類中 y 直接參數化包裝電路——是否還需要「把 y 逐 bit 餵進密文」取決於重建後的 garbling 怎麼切。**這裡藏著本路徑最大的設計自由度，也是最可能出「新洞察」的地方**

**必須面對的難點**：
- **SIM vs IND 的全面改寫**（難點 D5）：MM24 的 Thm 5/6 hybrid 全程用古典元件的 adaptive SIM-security。古典事實：unbounded-collusion SIM-secure IPFE 不可能（AGVW13 一系）；ALS16 只有 IND。救命稻草是 **ALMT20（Agrawal-Libert-Maitra-Titiu, PKC 2020, ePrint 2020/209）：ALS 型 IPFE 的單金鑰 adaptive SIM-security**——恰好卡進 single-query 框架。兩個選擇：(a) 用 ALMT20 保持 SIM 路線（單金鑰限制與 MM24 的 single-query 一致，無損失）；(b) 把證明全面 IND 化（工作量大，但 BMMS26 已證量子端 SIM 反正到頂，IND 化有獨立價值）。建議 (a) 先行
- **AKY24 slot 的替換問題（Project C 的升級與警告）**：BBC26 給了無條件、金鑰古典（「even when both are given the key」）、one-time 單 bit 的理想 UE——正是 MM24 Remark 2 邀請的 ideal-UE reduction 對象。**但精讀 BBC26 v2 發現關鍵障礙：其方案用 Haar 隨機酉做加密，作者明言「the scheme no longer admits an efficient construction」**——金鑰（U 的描述）沒有 poly 表示，解密電路沒有 poly 描述，直接違反 lifting 對 s(λ) 的需求。橋接方向：把 Haar 酉換成有效近似（酉 t-design / PRU——BG26b 用 PRU 的先例；2603.06393 研究 approximate 2-design 的 UE），但 design 夠不夠、要幾階、安全損失多少，全是 open——**高風險高回報，適合當論文最後一章的 conditional result，不適合當關鍵路徑**
- 訊息空間含 EPR halves（§2(b)）：即使 payload 是古典內積，W-QFE 仍須支援量子訊息——OneQFE 那層（QOTP + IdFE/ALMT20-IPFE）天然支援，保留即可

### 3.3 路徑 P3（教訓與可搶救部分）：Clifford 直構的可逆性洩漏攻擊

之前討論的直構想法：`Enc(ρ) = (X^a Z^b ρ, IPFE.Enc(a,b))`，KeyGen(C) 對 Clifford C 發「pad 更新線性映射 L_C」的 IPFE 金鑰（C·X^aZ^b = X^{a′}Z^{b′}·C，(a′,b′) = L_C(a,b) 是 F₂ 上的線性映射）。

**攻擊**：Clifford 是可逆的 ⇒ L_C 可逆。解密者拿到 sk_{L_C} 後**不必先把 C 作用在量子態上**：直接解古典部分得 (a′,b′) = L_C(a,b)，計算 (a,b) = L_C^{-1}(a′,b′)，對原封不動的量子密文拆 pad，得到 **ρ 本身**而非 C(ρ)。功能性洩漏 = 全訊息，方案作為 FE 完全不安全。

**為什麼 QHE 沒這個問題**：Broadbent-Jeffery（CRYPTO 2015）的 Clifford-QHE 也是「QOTP + 古典追蹤 pad 更新」，但 evaluator **沒有解密金鑰**，pad 資訊始終加密著流動；FE 的本質就是把解密能力交出去，攻擊隨之成立。

**可搶救的部分**：
- 把這個攻擊一般化：**對任何「可逆函數類 + 古典金鑰直接揭露修正資訊」的 QFE 設計，同型攻擊都成立**（可逆性 ⇒ 修正資訊 ≡ 完整 pad ⇒ 訊息全裸）。寫成一個 no-go observation / design principle：「量子訊息上的 QFE for 酉類，函數金鑰揭露的古典資訊必須與 pad 統計獨立」——這解釋了為什麼 MM24 的 garbled-circuit 路線（修正資訊被鎖在 garbling 內、必須物理上消耗量子態才能取出）是必要的，也給 (II) 的任何未來構造劃了紅線
- 修法方向（若堅持做 (II)）：函數金鑰帶量子 gadget（EPR halves），解密 = teleport-through-gadget，修正在 Bell 測量後才出現且與消耗態不可分——這本質上是「單 gate 版的 QGC」，說明 (II) 的量子金鑰大概率不可避免；或改函數類為 **measure-then-linear**（先計算基測量再算內積）——不可逆，攻擊失效，且與 (I) 銜接

**產出形態**：2–3 頁的 observation + 邊界刻畫，放進論文當 motivation section 或獨立 short note。這是目前手上最接近「已完成」的可交付成果。

---

## 4. 難點總表

| # | 難點 | 擊中路徑 | 嚴重度 | 建議動作 |
|---|------|----------|--------|----------|
| D1 | Thm 7 lifting 消耗 universality：包裝電路 ∉ IPFE 類 | P2 | ★★★（結構性） | C_Dec hardcode 驗證（兩週任務）；或改走 P1 |
| D2 | P1 坑 1：合法解密者經 r 學到超過 ⟨x,y⟩ 的資訊 | P1 | ★★★（目前 P1 最硬） | 四個修正方向見 §3.1；先做單金鑰版保底；與老師討論遞迴 FE 結構 |
| D3 | IPFE 的 SIM-security 天花板（AGVW13 不可能性；ALS16 僅 IND） | P2 | ★★ | 用 ALMT20 單金鑰 adaptive SIM 續命；長期 IND 化 |
| D4 | 可逆性洩漏攻擊 | P3、(II) | ★★★（對直構致命） | 轉化為 no-go observation（可交付） |
| D5 | UEQ 介面：poly 解密電路描述、同 randomness 雙生成、可 teleport 純態金鑰 | P1、P2 | ★★ | 精讀 AKY24 確認每一項；BBC26 不滿足效率項 |
| D6 | BBC26 無有效構造（Haar 酉），ideal-UE 替換不是 free | P2 加分項 | ★★ | 降級為 conditional result；追蹤 design/PRU 橋接文獻 |
| D7 | 量子端 admissibility（Def 25）與 2-player IND（Def 30）的 plumbing 重走 | P1、P2 | ★★ | 精讀 MM24 Appendix A.2 + Def 25；P1 版遊戲先寫死不含糾纏訊息的特例 |
| D8 | multi-bit / q-ary UE 的 hybrid 論證在 cloning 遊戲下非標準 | P1 | ★★ | 引 HKNY24 明文擴展或 AKLL22 multi-bit；不行就自己證（CGKNY26 有工具） |
| D9 | 抗合謀語義三分：多古典 FE 金鑰（IPFE 端免費）vs 多 UFE 金鑰對（Def 26 的 B/C）vs 多密文拷貝（CGKNY26） | 全部 | ★ | 論文裡開一節把三者定義切開，本身是貢獻 |
| D10 | 效率主張要能兌現（lattice 參數、密文尺寸帳） | P1、P2 | ★ | K9 補課；寫 comparison table 時再精算 |

---

## 5. 知識補完清單

按「動手前必備 → 動手中精讀 → 加分」排序。每項附驗收標準（能做到什麼才算讀完）與對應難點。

### 必備（未補齊前不要開始寫構造）

| # | 主題 | 材料 | 驗收標準 | 對應 |
|---|------|------|----------|------|
| K1 | Pauli/Clifford/stabilizer 形式主義、quantum teleportation、gate teleportation | Nielsen-Chuang §4.2–4.3、§10.5；Gottesman 講義 | 能手推 C·X^aZ^b·C† 的 pad 更新規則（對 H、CNOT、S 逐一算）；能寫出 teleport 一個態經過 EPR pair 的修正傳播 | P3 攻擊、D1、D4 |
| K2 | QOTP 上的加密計算（Clifford 免費、T 要 gadget） | Broadbent-Jeffery (CRYPTO 2015)；Dulek-Schaffner-Speelman (CRYPTO 2016) | 能解釋 BJ15 的 CL scheme 為何安全、且為何同樣結構搬到 FE 就出現 §3.3 攻擊 | P3、(II) |
| K3 | AK21 的可重用性轉換與 unclonable-IND 定義 | 已讀 ✓，回頭精讀 reusability 那節與 Def 16/18 對照 | 能複述 PRF 技巧如何把 one-time UE 變 reusable；能寫出 unclonable-IND 遊戲的形式化 | P1 坑 1、坑 2 |
| K4 | MM24 的 Def 25（admissibility）、Def 30（2-player IND）、Appendix A/B | mm24 原文 | 能把 Def 26 特化到 (I) 語義並自己寫出 P1 的安全遊戲定義 | D7 |

### 動手中精讀

| # | 主題 | 材料 | 驗收標準 | 對應 |
|---|------|------|----------|------|
| K5 | ALS16 逐行 + ALMT20 單金鑰 adaptive SIM | ALS16（已在讀）；ALMT20 = ePrint 2020/209 | 能陳述 ALMT20 的確切定理（單金鑰、adaptive、SIM）並指出它卡進 MM24 Thm 5 的哪個位置 | D3 |
| K6 | AKY24 精讀：金鑰結構、效率、Def 18 對應物 | AKY24 / ITCS 2025 版 | 逐條核對 §2(d) 的五項介面需求；特別確認其金鑰生成是否有效率（Haar 態 vs PRS 實例化） | D5 |
| K7 | GVW12：IdFE/TwoFE/bounded-collusion 的定義與 SIM 證明 | GVW12（推薦清單原有） | 能複述 Thm 5/6 對古典元件的最小需求，判斷 IPFE 能否頂替各 slot | P2 |
| K8 | BY22 QGC：DQRE 介面、C+M 電路的輕量 garbling | BY22（推薦清單原有） | 能指出包裝電路 W 的哪些段落落在 C+M 內、garbling 成本如何估 | P2 |
| K9 | 古典 FE 定義動物園與不可能性 | BSW11、O'Neill 2010、AGVW13 | 能畫出 IND/SIM × adaptive/selective × 金鑰數 的關係圖並標出不可能區 | D3、論文 related work |

### 加分

| # | 主題 | 材料 | 對應 |
|---|------|------|------|
| K10 | MoE 遊戲的平行/XOR 重複 | AKL23（已讀 ✓）+ arXiv:2509.01831（ITCS 2026） | D8 若需自證 multi-symbol bound |
| K11 | lattice 參數估計 | lattice-estimator 工具 + ALS16 參數節 | D10 |
| K12 | HKNY24 明文擴展、BG26b boosting、CGKNY26 multi-copy | survey 報告文獻表 | P1 實例化表、D8、D9 |

---

## 6. 建議執行順序（12–16 週）與決策點

**W1–2**：K1、K2 補課。同時把 §3.3 的攻擊寫成 2–3 頁正式 note（含一般化陳述與 QHE 對照）——**第一個可交付成果**，拿去跟老師確認方向。
**W3–4**：K3、K4。寫出 (I) 語義的正式定義（Def 26 特化）＋ P1 compiler 定理的正式陳述。此時 D2（坑 1）的修正方向要跟老師定案：單金鑰保底 vs 遞迴 FE 結構。
**W5–8**：P1 證明主體（IPFE 歸約段 + UE 歸約段）。並行精讀 K5、K6。
**W9–10**：AK21 式 PRF 可重用化 + 實例化表（QROM 版 / AKY24 版 / conditional BBC26 版）。
**W11–12**：整理成 technical report 給老師；決策點——(a) P1 順利 ⇒ 深化（public-key 化、多金鑰）；(b) P1 卡在 D2 ⇒ 收縮為「單金鑰 Unclonable IPFE + P3 no-go」雙貢獻包；(c) 行有餘力 ⇒ 開 P2 的 C_Dec hardcode 驗證
**W13–16**（彈性）：P2 的包裝類 QFE 設計，或 (II) 的定義章。

**退場條件**：若 W8 結束時 P1 的兩段歸約仍有任一段寫不攏，停損收縮到 (b) 包——P3 note + 定義 + 部分結果依然構成可答辯的碩論骨架。

**給老師的三個問題**（下次 meeting 用）：
1. P1 的坑 1（解密者經 r 學到 x）：您傾向單金鑰保底版，還是值得投入遞迴 FE 結構（趨近 MM24 trojan 式構造）？
2. P2 的 C_Dec hardcode 驗證我計畫做成兩週的獨立 milestone——這個「把 universal 需求釘死成固定 UEQ」的弱化，您認為對定理價值的影響有多大？
3. BBC26 的效率鴻溝（Haar 酉不可有效實作）讓「換成無條件底座」變成 conditional result——這樣的章節放碩論裡合適嗎，還是乾脆砍掉？

---

## 7. 本文引用的文獻（新增於既有清單之外）

- **[ALMT20]** Agrawal, Libert, Maitra, Titiu. *Adaptive Simulation Security for Inner Product Functional Encryption*. PKC 2020; ePrint 2020/209.（單金鑰 adaptive SIM-secure IPFE——D3 的救命稻草）
- **[BJ15]** Broadbent, Jeffery. *Quantum Homomorphic Encryption for Circuits of Low T-gate Complexity*. CRYPTO 2015.（Clifford-QHE；P3 攻擊的對照組）
- **[DSS16]** Dulek, Schaffner, Speelman. *Quantum Homomorphic Encryption for Polynomial-Sized Circuits*. CRYPTO 2016.（T-gate gadget）
- **[BBC26]** Bhattacharyya, Broadbent, Culf. *The Uncloneable Bit Exists*. arXiv:2603.08916v2（2026-06）。本文確認：Haar-measure 加密、金鑰古典、strong uncloneable security（exp(−q)）、**無有效構造**（作者原文：「the scheme no longer admits an efficient construction」）。
- **[AKLL22]** Ananth, Kaleoglu, Li, Liu, Zhandry. *On the Feasibility of Unclonable Encryption, and More*. CRYPTO 2022.（QROM unclonable-IND；P1 首選實例化）
- **[HKNY24]** Hiroka, Kitagawa, Nishimaki, Yamakawa. 明文擴展技術. TCC 2024.（bit → 任意長度；D8）
- 其餘見 `Survey_Recent_Developments_and_New_Routes_2026-07.md` 文獻表。
