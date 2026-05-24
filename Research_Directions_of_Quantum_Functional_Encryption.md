# 量子函數加密（Quantum Functional Encryption）碩論方向 Survey

> 背景：報告完 Mehta & Müller (2024) "Unclonable Functional Encryption"（arXiv:2410.06029）後，老師給出兩個延伸方向的建議，並進一步討論「升級階梯」這個 compiler 式的研究方向。本文件整理 survey 結果與初步思考。

---

## 一、老師給的兩個延伸方向

### 第一點：往下挖 QFE 真正的底層

你報告的論文走的是一條「very general」的路線：用 [BY22] 的 Quantum Garbled Circuits 把整個 poly-sized 量子電路 garble 起來，然後用一個 universal circuit $U(C, \rho_m) = C(\rho_m)$ 把電路描述當輸入丟進 QFE。它的 poly-sized 構造其實是：

- **QGC 的 decomposable encoding**
- **+ 對每一個 bit 用 classical TwoFE（[GVW12]）**
- **+ 兩個 OneQFE 處理 quantum input 與 offline 部分**

Mehta-Müller 把 underlying classical FE 當黑盒，但古典 FE 本身又有非常豐富的階層（這是過去十年的研究主軸）：

> FE for **linear functions（inner product）** → FE for **quadratic functions** → FE for **polynomials of degree d** → FE for **all circuits（需要 iO）**

從低次到高次，假設強度與效率差非常多。老師希望你弄清楚 FE 並不是一個東西，而是一整個階梯。

### 第二點：簡化 decomposable 那塊

如果 underlying function class 不是 poly-sized 整個電路，而是更受限的 class（linear、quadratic、NC1 等），那麼 QGC 的 decomposable encoding 那一塊（也就是把 $\{U_1, \ldots, U_l, U_{in}, U_{off}\}$ 拆成獨立片段、然後每片獨立加密）可以大幅簡化甚至不必要。

換句話說，**對受限 function class 的 QFE，可能根本不需要走 QGC 這條路**，這就是一個值得研究的方向。

---

## 二、推薦的閱讀清單

### (A) 古典 FE 的階梯（必讀，由淺入深）

要做 quantum 版本之前，必須對 classical FE 的核心構造非常熟悉。建議依下列順序看：

1. **Abdalla, Bourse, De Caro, Pointcheval (PKC 2015)** — *"Simple Functional Encryption Schemes for Inner Products"*
   - IPFE 的開山之作，從 DDH 構造出第一個 inner-product FE
   - 論文非常短、技巧非常乾淨，建議先把這篇逐行讀懂

2. **Agrawal, Libert, Stehlé (CRYPTO 2016)** — *"Fully Secure Functional Encryption for Inner Products, from Standard Assumptions"*（ALS16）
   - 把 IPFE 推到 adaptive security
   - 給出 LWE、DDH、DCR 三種版本
   - **LWE 版本對你最重要**，因為它是 post-quantum 的

3. **Baltico, Catalano, Fiore, Gay (CRYPTO 2017)** — *"Practical Functional Encryption for Quadratic Functions with Applications to Predicate Encryption"*
   - 從 bilinear pairing 出發的第一個 quadratic FE
   - 看完這篇就知道為什麼從 linear 跳到 quadratic 需要新的工具

4. **Lin (CRYPTO 2017)** — *"Indistinguishability Obfuscation from SXDH on 5-Linear Maps and Locality-5 PRGs"*
   - 與 **Ananth, Sahai (EUROCRYPT 2017)**
   - FE for degree-$D$ polynomials，是 quadratic 之上的延伸，從 $D$-linear maps 構造

5. **Gorbunov, Vaikuntanathan, Wee (CRYPTO 2012)**
   - 這就是 Mehta-Müller 用的那個 TwoFE 的源頭
   - 順便看一下定義與 simulation security 的證法

### (B) 量子端的底層（必讀）

6. **Brakerski, Yuen (STOC 2022)** — *"Quantum Garbled Circuits"*（[BY22]）
   - 這是你正在讀的論文最核心的依賴
   - 應該完全理解 QRE、decomposability、classical encoding for classical inputs 這幾個概念
   - 以及為什麼 garbling 要把電路一個 gate 一個 gate 來處理

7. **Bartusek, Khurana (2021)** — *"Indistinguishability Obfuscation of Null Quantum Circuits and Applications"*（[BK21]）
   - qiO 的基礎，跟你論文最後一節的 QMIFE → qiO 那塊有直接連結

8. **Ananth, Kaleoglu, Yuen (2024)** — *"Unclonable Encryption with Quantum Decryption Keys"*（[AKY24]）
   - 你論文 Section 5 構造 UFE 時用的關鍵原語

9. **Broadbent, Lord (TQC 2020)** — *"Uncloneable Encryption"*（[BL20]）
   - UE 的正式定義來源

### (C) 最重要的 follow-up（一定要看！）

10. **Barhoush, Mehta, Müller, Salvail (arXiv:2601.17497, Jan 2026)** — *"On the Impossibility of Simulation Security for Quantum Functional Encryption"*
    - **這是 Mehta & Müller 本人加上 Barhoush 與 Salvail 在 2026 年 1 月發的最新論文，直接接續你正在讀的這篇**
    - 證明了：
      - 當 adversary 可以查詢 unbounded 數量的 challenge messages 時，SIM-security 在 quantum 設定下也是 impossible 的（無條件成立）
      - 當可以查詢多個 function keys 時，impossibility 也成立（只需 pseudorandom state generators，比古典需要 PRF 還弱）
    - **這篇是你論文後續方向的「地圖」，強烈建議優先讀**

11. **Barhoush, Salvail (arXiv:2309.06702)** — *"Functional Encryption in the Bounded Storage Models"*
    - 上面那篇的姐妹篇
    - 繞過 impossibility 的方法是 bounded quantum storage model
    - 可以做為「另一條路」的思路來源

### (D) 與量子 FE 鄰近的近期研究

12. **Hiroka, Morimae, Nishimaki, Yamakawa** — *Certified Everlasting Functional Encryption*（quantum 的 certified deletion + FE）

13. **Kitagawa, Nishimaki** — *FE with Secure Key Leasing*（[KN22]）

14. **Çakan, Goyal (TCC 2023)** — *FE with copy-protected secret keys*

> 這些是「給古典 FE 加上量子特性」的另一個流派，跟你現在的「給量子訊息做 FE」是平行但相關的研究。

---

## 三、對碩論方向的初步構想（依老師兩點建議）

### 方向 1：QFE for restricted classes（最推薦，最契合老師的提示）

目前 Mehta-Müller 的 QFE 是 "QFE for all poly-sized quantum circuits"，用 QGC 黑盒。但是：

- 對 **linear functions over quantum messages**（例如對 quantum state 做某些 Pauli measurement 的線性組合，或者把 quantum state 部份視為 classical 後做 inner product），其實不需要 garble 整個電路。可能直接從 ALS16 的 IPFE + Quantum One-Time Pad 或 quantum teleportation 就可以拼出來，假設可以弱化到 LWE
- 對 **quadratic functions on quantum messages**，可能可以結合 [BCFG17] 的 pairing-based 技巧 + quantum 編碼
- 關鍵問題：當 message 是 quantum state 時，"linear function" 的定義要重新想，這本身就是個有趣的 definitional contribution

**可能的論文題目雛形**：
- 「Quantum Functional Encryption for Inner-Product Functionality from LWE」
- 「Efficient QFE for Low-Degree Functionalities without Quantum Garbled Circuits」

### 方向 2：簡化或改進 Mehta-Müller 的 poly-sized 構造

照老師的第二個提示：如果 underlying function class 不是 poly-sized 整個電路，decomposable encoding 可以簡化。具體可以做的：

- 找出哪些 function class 不需要 full QGC，可以直接用更輕量的 randomized encoding
- 探討當 underlying classical FE 換成 IPFE 或 quadratic FE 時，整個 hybrid construction 變成什麼樣子
- 量化 ciphertext size、key size、computational complexity 的改善

### 方向 3：繞過 Impossibility 的構造性結果

Barhoush 等人 2026 年的論文證了 SIM-secure QFE 是 impossible 的。那麼：

- 在 IND-security 下，能否得到 Mehta-Müller 沒給出的更具體構造（例如基於 LWE 而非黑盒 QGC）？
- 在 bounded quantum storage model（[Barhoush-Salvail 23]）下能做到什麼？
- 是否可以給出 single-query 或 selective security 下的更強構造？

### 方向 4：QFE 與其他量子原語的連結

- 你論文最後一章已經連結 QFE 與 qiO，反過來：能否用 IPFE 或 QFE 構造其他量子原語（quantum money、copy-protection、certified deletion）？
- 例如 Çakan-Goyal 的 copy-protected FE 是「古典訊息+量子 key」，你可以研究「量子訊息+古典 key」這條對偶路徑

### 建議的時間安排

- **第一階段（1～1.5 個月）**：把 (A) 的 5 篇古典 FE 論文吃透，特別是 ABDP15、ALS16、BCFG17。這是地基。同時把 BY22 也仔細啃一遍
- **第二階段（1 個月）**：讀 (C) 的 impossibility 論文（2601.17497），這會大幅改變你對「哪些方向值得做、哪些是死路」的判斷
- **第三階段**：開始進入研究選題

---

## 四、老師後續討論：「升級階梯」compiler 方向

### 老師提出的核心觀察

> 「這篇量子函數加密等同於是幫我們把升級的通道建好，一層一層地堆上去，到最後可以得到不可複製的性質，那我們就可以把底層的元件換成其他的 scheme，然後照樣的推到不可複製的性質」

### 升級階梯的四層結構

Mehta-Müller 的論文本質上是搭了一個四層的塔：

| 層級 | 名稱 | 建造方式 |
|------|------|----------|
| 第一層 | **QGC（[BY22]）** | 提供 decomposable quantum randomized encoding |
| 第二層 | **OneQFE**（QFE for a single circuit） | QGC + Quantum One-Time Pad + classical IdFE |
| 第三層 | **poly-sized QFE** | OneQFE + classical TwoFE（[GVW12]）+ universal quantum circuit 的 decomposable encoding |
| 第四層 | **UFE（Unclonable QFE）** | 任意 QFE + UE-with-quantum-decryption-keys（[AKY24]）+ flag bit / teleportation trick |

每一層都是「黑盒地用下面那層 + 一些古典或量子原語」拼出來的。**這個架構本身就是一個 generic compiler，而不只是一個 ad-hoc 構造**。

老師說的「換掉底層元件、照樣推到不可複製」這件事，就是把這個 compiler **解耦**，去問：每一層真正需要下層提供什麼性質？只要下層滿足這些性質，是不是用什麼具體 scheme 都可以塞進來？

### 為什麼這個方向是好的

**第一，它直接解決了原論文的開放問題。**
Mehta-Müller 在 Section 5 結尾就提到：與其 reduce 到「UE with variable decryption keys」，能不能 reduce 到 ideal UE（unclonable-indistinguishable + classical keys）？這就是在問「能不能換底層元件」。他們自己沒做，但 invite 別人做。

**第二，它讓 unclonability 從 "all poly-sized circuits" 下放到 restricted classes。**
目前的 UFE 結果是「任意 poly-sized 量子電路都可以 unclonable evaluation」，這在實務上其實太強了，密文 size 和效率都不好看。如果你把第三層的 underlying class 換成 inner product 或 quadratic，最後得到的就是「Unclonable Inner-Product FE on Quantum Messages」這種**具體、可實作、有應用場景**（隱私統計、聯邦學習）的東西。這比抽象的 UFE-for-all-circuits 更有 selling point。

**第三，它跟 Barhoush-Mehta-Müller-Salvail 2026 那篇 impossibility 完美互補。**
那篇證了 SIM-secure QFE 在許多 setting 下不可能。換言之，學界已經知道「無條件、最強形式的 QFE」做不到。剩下的路就是兩條：

1. 走 bounded storage model 等弱模型
2. 把 function class 限制小一點換取更強的構造性結果

後者就是這個方向。

**第四，碩論可以「層級式」進攻。**
你不需要一次解全部問題。可以先處理「第三層換成 IPFE 會怎樣」，做出來就是一篇結果；接著看「第四層換掉 AKY24 base 會怎樣」，又是一篇。一個碩論可以拆成 2～3 個 sub-contribution，逐步累積。

### 要警惕的「不那麼模組化」的地方

說實話，這個 compiler 並沒有它表面看起來那麼黑盒，有幾個地方的 proof 是與底層緊密耦合的，你要有心理準備可能要重證：

**1. 第三層那個 decomposable encoding 是耦合最深的地方。**
Mehta-Müller 用
$$\text{Encode}(U, C, \rho_m, R, e) = (U_1(C[1],R,e_1), \ldots, U_l(C[l],R,e_l), U_{in}(\rho_m,R,e_2), U_{off}(R,e_3))$$
這種拆解方式，**前提是 underlying class 是 universal circuit**。

如果你換成 IPFE，universal circuit 本身就不必要了——因為 inner product 已經有非常結構化的形式，不必再經過 "circuit description as input" 這一步。所以「換底層」不只是把 TwoFE 換成 IPFE 那麼簡單，整個第三層的拼裝邏輯可能要重新設計。但這也意味著結果可能更乾淨、更有效率。

**2. 第四層的 flag bit trick 用到了 QFE 可以 hardcode 量子 EPR pair 進 ciphertext、再用 function key 帶 teleportation correction 的能力。**
這個 trick 對任何「正常的」QFE 都應該 work，但如果你的 underlying QFE 是 secret-key、或不支援 quantum input、或 keyGen 不能 depend on quantum 資料，那這個 trick 可能要改寫。你要先驗證新的 underlying QFE 滿足這個 trick 所需的接口。

**3. [AKY24] 這塊不能輕易動。**
Section 5 整個構造的 security 是 reduce 到 UE-with-quantum-decryption-keys 的 unclonable-indistinguishability。如果你把這個底層換成 [BL20] 或 [KT22]，你不只是改一個假設，而是要重新設計整個 hybrid argument。這個工作量不小，但也是一個很有貢獻的 result——因為 [AKY24] 是 2024 才出來的，假設它存在等於是站在最尖端的構造上。從 [BL20] base 換上去其實是「弱化假設」，更值得做。

**4. Security notion 之間的 lifting 也不一定 free。**
例如你想做 IND-secure 而不是 SIM-secure（因為 2601.17497 已經證 SIM-secure 不可能），那 OneQFE 與 TwoFE 都要重新挑選有 IND-secure 版本的構造，hybrid argument 也要對應改寫。

### 建議的具體 sub-projects

依照「先易後難、先具體後抽象」的順序：

#### Project A（暖身、約 1～2 個月）：把整條 compiler 寫成抽象的介面規格

就是把每一層需要的「最小接口」明確寫出來：

- 第 N 層需要第 N-1 層提供什麼演算法？
- 什麼 correctness？
- 什麼 security？

這件事本身就是一個有貢獻的 conceptual contribution，而且做完之後你會清楚知道每個 slot 可以塞什麼。這也會幫你寫論文 introduction 與 related work，因為你會「看見整個地圖」。

#### Project B（核心 contribution，約 3～4 個月）：把第三層的 underlying class 換成 IPFE

目標：給出 **Unclonable IPFE for Quantum Messages** 的具體構造。

步驟：
- 用 ALS16 的 IPFE（LWE-based，post-quantum 假設）取代 GVW12 的 TwoFE
- 重新設計 universal circuit 那層（可能可以省掉），直接用 IPFE 的代數結構
- 跑一次第四層的 lifting，得到 Unclonable IPFE
- 比較 ciphertext size、key size、computational cost 跟原 Mehta-Müller 構造的差別

#### Project C（如果時間夠，約 2 個月）：把第四層的 base 從 [AKY24] 換成 [BL20]

這是降低假設。Mehta-Müller 自己在 Section 5 末尾就點名這是 open question。做出來等於是把 UFE 的假設條件變弱。

#### Project D（高風險高回報，視時間決定）：把 quadratic FE 也跑一次

類似 Project B 但 underlying 是 [BCFG17] 的 quadratic FE。難度高很多，因為 pairing-based 的 FE 跟 lattice-based 的 IPFE 拼裝方式不一樣，但成果會更 impressive。

### 策略建議

這個方向是 **constructive cryptography** 的典型作法：你不在發明新原語，你在把現有原語**重新組裝**並嘗試弱化假設、擴展 functionality。

這類論文的好處：
1. 不需要證新的 hardness（碩士階段這是現實）
2. 結果非常具體，audit-able
3. Reviewers 容易理解你的貢獻
4. 跟學界已有的工作有清楚的連結

唯一要小心的是 **不要做成「換一個假設、proof 完全照抄」** 的論文。你要在過程中找到一些「只有換掉底層才看得到」的新洞察——例如：

- 某層原本以為需要的性質其實不必要
- 某個 lifting 可以用更簡單的方式做
- 某個假設可以被弱化

這些洞察就是你的 conceptual contribution。

### 整體判斷

**這個方向是好的，可行的，而且跟最新文獻（2601.17497）連結得很自然。**

建議接下來：
1. 先把 Project A 的 abstract framework 寫出來給老師看，確認方向
2. 再投入 Project B

---

## 五、參考文獻速查表

### 古典 FE 階梯
- **[ABDP15]** Abdalla, Bourse, De Caro, Pointcheval. *Simple Functional Encryption Schemes for Inner Products*. PKC 2015.
- **[ALS16]** Agrawal, Libert, Stehlé. *Fully Secure Functional Encryption for Inner Products, from Standard Assumptions*. CRYPTO 2016.
- **[BCFG17]** Baltico, Catalano, Fiore, Gay. *Practical Functional Encryption for Quadratic Functions with Applications to Predicate Encryption*. CRYPTO 2017.
- **[GVW12]** Gorbunov, Vaikuntanathan, Wee. *Functional Encryption with Bounded Collusions via Multi-Party Computation*. CRYPTO 2012.
- **[Lin17]** Lin. *Indistinguishability Obfuscation from SXDH on 5-Linear Maps and Locality-5 PRGs*. CRYPTO 2017.

### 量子端底層
- **[BY22]** Brakerski, Yuen. *Quantum Garbled Circuits*. STOC 2022. (arXiv:2006.01085)
- **[BK21]** Bartusek, Khurana. *Indistinguishability Obfuscation of Null Quantum Circuits and Applications*.
- **[BL20]** Broadbent, Lord. *Uncloneable Encryption*. TQC 2020.
- **[AKY24]** Ananth, Kaleoglu, Yuen. *Unclonable Encryption with Quantum Decryption Keys*. 2024.

### 直接相關（你的論文與其後續）
- **[MM24]** Mehta, Müller. *Unclonable Functional Encryption*. arXiv:2410.06029 (2024).
- **[BMMS26]** Barhoush, Mehta, Müller, Salvail. *On the Impossibility of Simulation Security for Quantum Functional Encryption*. arXiv:2601.17497 (Jan 2026).
- **[BS23]** Barhoush, Salvail. *Functional Encryption in the Bounded Storage Models*. arXiv:2309.06702 (2023).
