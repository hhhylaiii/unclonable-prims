# 2026 年 7 月文獻總盤點：近期發展與可著手的研究路線

> **Survey 日期**：2026-07-10
> **方法**：以已精讀的四篇論文為錨點——[AK21]《Unclonable Encryption, Revisited》、[AKL23]《Cloning Games》、[MM24]《Unclonable Functional Encryption》、[ABDP15]《Simple FE for Inner Products》——向外檢索 2024 年 10 月（MM24 發表）之後的所有相關文獻，重點放在 2025 下半年至 2026 年 7 月的最新結果。
> **目的**：更新三份既有報告未涵蓋的進展，評估哪些研究路線仍然空白、哪些已被搶佔，並提出符合已讀論文基礎的新候選路線。

---

## 一、總覽：過去九個月的大事記

| 時間 | 事件 | 對本研究的意義 |
|------|------|----------------|
| 2025-10 | Broadbent-Culf-Rochette 給出**最優無條件 UTE** 並證明 UTE ≈ UE 漸近等價（arXiv:2510.00903） | UTE 正式成為通往 UE 的可用墊腳石 |
| 2025-10 | Çakan-Goyal-Kitagawa-Nishimaki-Yamakawa《Multi-Copy Security in Unclonable Cryptography》（ePrint 2025/1921）**上了 CRYPTO 2026** | 多複製安全從 open problem 變成已解決的基礎設施 |
| 2025-10 | Kitagawa-Nishimaki-Pappu《Collusion-Resistant SKL Beyond Decryption》（ePrint 2025/1842）**上了 EUROCRYPT 2026** | 金鑰租用流派持續高速推進，MLTT compiler 出現 |
| 2026-01 | **[BMMS26]** Barhoush-Mehta-Müller-Salvail 證明 **SIM-secure QFE 不可能**（arXiv:2601.17497） | 劃定 QFE 的死路，反向確立「受限類 + IND 安全」是活路 |
| 2026-02 | Bhattacharyya-Culf《Uncloneable Encryption from Decoupling》正式刊於 **Nature Physics** | 無條件 UE（逆多項式安全）進入頂級期刊視野 |
| 2026-03 | **[BBC26]** Bhattacharyya-Broadbent-Culf《The Uncloneable Bit Exists》（arXiv:2603.08916）——無條件、指數小優勢的不可複製位元 | 領域中心 open problem 被解決（注意：v1 有瑕疵，v2 於 2026-06 大幅修正分析） |
| 2026-03 | Bartusek-Goldin 在 **Haar 隨機預言機模型**中構造可重用、任意長度訊息的 UE（arXiv:2603.11437 / ePrint 2026/506） | 首個「可重用 UE 存在於 microcrypt」的證據 |
| 2026-05 | Bartusek-Goldin《Boosting Uncloneable Encryption in Microcrypt》（arXiv:2605.27647）：不可複製位元 + SKE ⇒ many-time 任意長度 UE（tight） | 「從 bit 放大到完整 UE」這條收尾路線已被做掉 |
| 2026-07-03 | Xu-Takeuchi 首個**容錯 secure key leasing**（arXiv:2607.02989） | SKL 流派開始走向實作面 |
| 2026-07-08 | Botteron-Broadbent-Culf-Nechita-Pellegrini-Rochette《Towards Unconditional Uncloneable Encryption》刊於 *Quantum* | 反交換 Pauli 候選方案的嚴格分析（K=2–7 證明、K≤17 數值） |

**一句話總結**：UE 的資訊論端（無條件構造）在 2026 上半年被 Broadbent/Culf 系與 Bartusek/Goldin 系兩組人馬高速收割完畢；**但 MM24 下游的 QFE 構造端至今無人動手**——這是對你最重要的策略情報。

---

## 二、各戰線詳細近況

### 2.1 QFE / UFE 戰線（錨點：MM24）

**[BMMS26] 不可能性結果的精確內容**（arXiv:2601.17497，2026-01）：

- **無界 challenge messages**：對手可查詢無界數量的挑戰訊息時，SIM-secure QFE **無條件不可能**（與古典屏障對齊）
- **多 function keys**：對手可拿多把函數金鑰時，不可能性只需假設 **pseudorandom quantum states（PRS）**——比古典所需的 PRF 更弱；另有一條獨立基於 PKE 的不可能性證明
- 技術貢獻：pseudorandom states 的**不可壓縮性（incompressibility）**性質，作者自認有獨立價值
- **留下的活路**：單一（或有界）query、selective security、受限 function class 下的 IND-secure 構造，以及 bounded storage model（[Barhoush-Salvail]，arXiv:2309.06702，v5 已改題為 *Simulation-Secure FE in the Bounded Storage Model*）

**MM24 引用地圖（截至 2026-07，Semantic Scholar）**——這是本次 survey 最有價值的發現：

引用 MM24 的論文**只有約五篇**，且全部不是構造端的跟進：
1. [BMMS26]（impossibility，作者群自己的續作）
2. Broadbent-Culf-Rochette 最優 UTE（只是引用比較）
3. Bhattacharyya-Culf decoupling（只是引用比較）
4. Botteron 等 Towards Unconditional UE（只是引用比較）
5. 一篇 Multi-Party FE survey（背景引用）

**結論：沒有任何人開始做「把 MM24 底層換成受限 function class」這件事。** 升級階梯 compiler 的窗口完全開著。MM24 本身仍是 ePrint 預印本（2024/1683，最後修訂 2025-03-14），尚未落地重要會議。唯一要提防的競爭者就是 Mehta-Müller 團隊本身——他們 2026 年 1 月的產出是 impossibility 方向，顯示其當前重心在劃界而非構造，但這個判斷需要每季重新檢查。

### 2.2 UE 本體戰線（錨點：AK21、AKL23）

**無條件 UE 已大勢底定**，收尾動作在 2026 上半年密集完成：

- **[BBC26]《The Uncloneable Bit Exists》**（arXiv:2603.08916）：兩個不通訊對手即使**都拿到金鑰**也無法同時解密，對手優勢指數小。技術核心是酉不變態的近似性質 + 對稱化論證壓縮對手策略空間。注意 v1（2026-03）的分析有瑕疵，v2（2026-06）重寫了大部分證明——引用時務必用 v2。
- **Botteron 等（Quantum, 2026-07-08）**：反交換 Pauli 運算子構造的候選方案，K=2 到 7 嚴格證明對手成功率為 1/2 + 1/(2√K)，NPA 層級數值驗證到 K=17，漸近上界 5/8。這篇與 BBC26 是同一個問題的兩條互補進路。
- **Bartusek-Goldin（HROM）**（arXiv:2603.11437）：在所有人都能查詢 Haar 隨機酉 U（含 U†、U*、U^T）的模型中，構造**可重用、任意長度訊息、unclonable-IND 安全**的 UE。技術貢獻是基於 path recording 框架的「unitary reprogramming lemma」。這是 miniCrypt/microcrypt 中存在可重用 UE 的首個證據。
- **Bartusek-Goldin（Boosting）**（arXiv:2605.27647）：假設不可複製位元存在，(a) many-time SKE ⇒ many-time t→t′ 任意長度 UE（且此結果 tight）；(b) PRU ⇒ identical-copy 安全的 many-time UE。**這意味著「從 BBC26 的 bit 放大成完整 UE」這個自然的碩論題目已經被做掉了**。

**結構理論端**：

- **Kitagawa-Yamakawa《Foundations of Single-Decryptor Encryption》**（ePrint 2025/1219，TCC 2025）：系統化 SDE 各安全概念間的蘊含與分離。與 AKL23 cloning games 的抽象化精神一脈相承，讀它可以校準「安全概念關係圖」這類貢獻的寫法。
- **UTE 支線**：Champion-Kitagawa-Nishimaki-Yamakawa 的 UTE 原始論文（arXiv:2410.24189）刊於 TCC 2025；Broadbent-Culf-Rochette（arXiv:2510.00903）給出**無條件 UT-IND 安全構造 + 多密文擴展 + 有界抗合謀擴展**，並在 PRU 假設下推到 unbounded／everlasting，同時證明 UTE 與 UE 隨對手數增長漸近等價。
- **[AKL23] 框架的直接延伸**：《The Curious Case of "XOR Repetition" of Monogamy-of-Entanglement Games》（arXiv:2509.01831，ITCS 2026）研究 MoE 遊戲的平行重複與 XOR 重複——這是 cloning games 證明機器的底層工具，若你走任何需要新 MoE bound 的構造，這篇是必讀。
- 其他擴展：連續變量 UE（arXiv:2603.06393）、裝置無關 UE（*Quantum*, 2025-01）。

**仍然開放的核心問題**（與 2025 年時相同）：標準模型、古典解密金鑰、IND 安全的計算性 UE。這是大魔王，不建議碩論正面進攻，但所有路線的敘事都應該指向它。

### 2.3 金鑰生命週期流派：SKL／Certified Deletion／Copy Protection

這個流派（「給古典原語加上量子金鑰管理」）在 2025–2026 進展極快，且**與 IPFE 的交集存在明確空白**：

| 原語 | SKL 已知結果 | 假設 |
|------|--------------|------|
| PKE | Kitagawa-Morimae-Yamakawa simple framework（arXiv:2410.03413，EUROCRYPT 2025）：古典撤銷 | 僅需 IND-CPA PKE |
| PRF | 同上 | OWF |
| 簽章 | 同上（靜態簽章金鑰） | SIS |
| PKE/ABE 抗合謀 | ePrint 2025/262（ASIACRYPT 2025） | 標準假設 |
| 解密以外的功能、抗合謀 | Kitagawa-Nishimaki-Pappu，ePrint 2025/1842（**EUROCRYPT 2026**）：multi-level traitor tracing（MLTT）compiler | 標準假設 |
| 容錯版 PKE-SKL | Xu-Takeuchi，arXiv:2607.02989（2026-07） | — |
| FE（secret-key、bounded） | [KN22]（ASIACRYPT 2022） | 標準 SKFE |
| **公鑰 IPFE** | **檢索範圍內未發現任何工作** | — |

Certified deletion 端：certified everlasting FE 的抗合謀版本（arXiv:2302.10354）依賴 iO 做一般電路；**受限 function class（如 inner product）+ 標準假設**的組合同樣未被佔據。

Copy protection 端持續活躍：UPO Revisited（ePrint 2025/1880，TCC 2025）、entropic inputs 上的複製保護（ePrint 2025/1264）、任意挑戰分布下的 malleable-puncturable 功能複製保護（arXiv:2507.19032）。這條線假設都很重（iO），對碩論參考價值大於進攻價值。

### 2.4 古典 IPFE 端近況（錨點：ABDP15）

古典 IPFE 在 2025 仍有紮實的增量進展，可作為你構造的即插即用元件庫：

- **Tightly Secure IPFE Revisited: Compact, Lattice-based, and More**（ePrint 2025/1613，ASIACRYPT 2025）：首個 almost-tight 的 lattice IPFE——如果你的構造 reduce 到 IPFE，用它可以拿到更好的具體安全參數
- **Toward Practical Lattice-based Unbounded IPFE**（ePrint 2025/2232）：向量長度不需在 setup 時固定，附實作
- Identity-based IPFE from RLWE（tight、QROM）、去中心化多授權 AB-IPFE from lattices（arXiv:2505.11744）
- 註：另有一篇「IPFE with fine-grained revocation」（arXiv:2509.07804）——這裡的 revocation 是古典存取控制意義，與量子 SKL 的 revocation 無關，檢索時勿混淆

### 2.5 可模糊性／偽金鑰周邊（錨點：主線二）

- **Non-Committing Registered FE**（Sarkar，ePrint 2026/1293，AFRICACRYPT 2026）：把 NCE 推廣到 registered FE 設定，lattice 實例化基於 **plain LWE + equivocal LWE**。兩個看點：(1) 「**equivocal LWE**」已成為一個被命名、被使用的假設——你若要正式化偽金鑰性質，這是現成的代數基礎；(2) 證明「NCE × FE」的組合是學界認可的定義性貢獻題型。
- **RNCE with quantum ciphertexts**：《Robust Combiners and Universal Constructions for Quantum Cryptography》（arXiv:2311.09487）中出現「量子密文的接收者 NCE 可從量子密文 PKE 構造」；certified everlasting RNCE 亦已在 arXiv:2207.13878 中被定義。**但「偽金鑰性質的公鑰類比」本身仍無人正式定義**——主線二的 open problem 原封不動。

---

## 三、策略情報：空白區 vs 擁擠區

```
擁擠區（避免正面進入）                    空白區（可著手）
─────────────────────────              ─────────────────────────
無條件 UE / uncloneable bit             QFE for restricted classes（零篇）
  （Broadbent/Culf 系 × Bartusek/        UFE/UT-FE for inner products（零篇）
   Goldin 系，半年七篇）                  公鑰 IPFE-SKL / certified deletion
UE boosting / 放大定理（已收尾）           IPFE（檢索未見）
多複製安全（CRYPTO 2026 已解決）           公鑰偽金鑰性質正式定義（仍無人做）
copy protection from iO                  IPFE 的偽金鑰／可模糊性
PKE/ABE/PRF/簽章的 SKL                   cloning game for IPFE functionality
```

三個結構性觀察：

1. **QFE 構造端是整張地圖上最反常的空白**。MM24 出版 21 個月、被 BMMS26 劃清死路之後，「受限類 + IND 安全」是官方認證的活路，卻沒有任何人（包括原作者）動手。合理解釋：會做 lattice FE 的人不熟量子編碼，會做量子的人不熟 IPFE 代數——你恰好兩邊都在讀，這就是你的比較優勢。
2. **SKL 流派的推進模式是「一個原語一篇論文」**（PKE → ABE → PRF → 簽章 → FHE → …），而 IPFE 這一格還空著，且所需工具（BB84-based framework + ALS16）全部現成。這種題目的風險輪廓非常適合碩論。
3. **UE 資訊論端的人力密度極高**，同一個問題半年內出現兩組人平行解決（BBC26 vs Botteron 等）。任何你能在三個月內想到的資訊論 UE 題目，大概率有人已經在做。

---

## 四、候選研究路線評估

以下路線均以「必須直接建立在四篇已讀論文之上」為篩選條件。路線甲即原路徑 1（維持），乙、丙、丁為本次 survey 新增或實質更新的選項。

### 路線甲：QFE/UFE for restricted classes（原路徑 1，地位強化）

- **內容**：Project A–D 照舊——把 MM24 第三層的 TwoFE/universal circuit 換成 IPFE 代數結構，目標「Unclonable IPFE for quantum messages」；含 Clifford-QOTP 金鑰更新線性化的直接構造想法（QOTP pad 更新對 Clifford 電路是線性映射，可由 IPFE 直接發放，砍掉整個 QGC 層）
- **Survey 後的更新**：(1) 引用地圖確認無人搶進；(2) BMMS26 給了最好的動機段落素材（「SIM 不可能 ⇒ 受限類是必然出路」）；(3) 古典元件可升級為 tightly-secure lattice IPFE（ePrint 2025/1613）
- **契合度**：MM24（骨架）+ ABDP15（替換元件）+ AK21/AKL23（第四層 lifting 的安全性語言）——四篇全用上
- **風險**：中。第三層拼裝邏輯需重新設計（已知）；競速壓力來自原作者團隊（目前判斷在做 impossibility，非構造）

### 路線乙（新增）：公鑰 IPFE with Secure Key Leasing / Certified Deletion

- **內容**：定義並構造「函數金鑰可租用、可驗證撤銷」的公鑰 IPFE。租出去的 sk_y 是量子態，撤銷後持有者無法再算 ⟨x, y⟩。構造素材：ALS16 的 LWE-IPFE × Kitagawa-Morimae-Yamakawa 的 BB84-based simple framework（EUROCRYPT 2025）
- **為什麼現在做**：SKL 流派「一原語一論文」的推進模式下 IPFE 格子還空著；KN22 只做了 secret-key bounded FE；PKE/ABE 抗合謀 SKL（ASIACRYPT 2025）與 MLTT compiler（EUROCRYPT 2026）提供了直接可模仿的證明模板
- **契合度**：ABDP15/ALS16（主體）+ AKL23（SKL 安全性歸約到 monogamy-of-entanglement 式論證，cloning games 語言直接適用）
- **風險**：低-中。工具全部標準假設、全部現成；主要工作是「IPFE 的函數金鑰結構（一個向量 y 的 lattice 陷門）如何與 BB84 編碼相容」這個技術問題。**注意事項**：動手前必須精讀 ePrint 2025/1842，確認其 MLTT compiler 是否已隱含涵蓋 IPFE（其宣稱範圍是 "beyond decryption"，以 PRF/簽章為例；IPFE 未被點名，但 compiler 的普適性需逐條核對）
- **與甲的關係**：完全共用文獻基礎，可作為平行保底項目——若甲的第三層重構卡關，乙保證有可發表產出

### 路線丙（新增變體）：Untelegraphable IPFE——unclonability 的輕量版下放

- **內容**：把 Champion 等（TCC 2025）的 UTE/UT-FE 概念實例化到 inner product：兩個只能傳古典訊息的對手不能同時算出正確的 ⟨x, y⟩。因為 UTE 已有無條件構造（Broadbent-Culf-Rochette 2025-10），UT-IPFE 有機會在**極弱假設（甚至無條件）**下達成，而完整的 unclonable IPFE 需要重得多的工具
- **契合度**：MM24（UFE 定義的弱化版）+ ABDP15 + AKL23（UTE 的安全遊戲是 cloning game 的古典輸出限制版）
- **風險**：低。定義 + 構造 + 與 UFE-IPFE 的分離／蘊含關係，工作量可控
- **定位**：不建議單獨成篇，而是作為路線甲的 fallback 階梯——「先做 UT-IPFE（弱、穩），再攻 unclonable IPFE（強、險）」，論文敘事上自然形成遞進

### 路線丁（更新）：IPFE 的偽金鑰性質——主線二的 FE 化收縮

- **內容**：不直接進攻「公鑰 UE 的偽金鑰性質」這個大 open problem，而是收縮到 IPFE：定義 fake function key——給定 ct = Enc(x)，FakeGen 產生 sk′_y 使其解出指定值 z ≠ ⟨x,y⟩，且 (ct, sk′_y) 與真實對不可區分。然後研究它 (1) 能否從 equivocal LWE（ePrint 2026/1293 使用的新假設）構造；(2) 是否給出通往 unclonable IPFE 的替代技術路徑（即方向四報告中「偽金鑰技術能否替代 MM24 路徑」的具體化）
- **契合度**：AK21（偽金鑰性質原始定義）+ ABDP15（代數結構）+ MM24（對照組）
- **風險**：中-高。定義性工作保底（參照 NC-RFE 被 AFRICACRYPT 接受，證明此題型可發表），但構造能否成立是真正的研究賭注。ALS16 金鑰的線性結構（sk_y 本質上是 y 的一個 lattice 相關線性函數）讓「偽造一把解出指定值的金鑰」比一般 FE 有更多代數空間，這是可行性的正面信號
- **定位**：作為甲的概念深化備援；若甲進行中發現 IdFE≈equivocability 的觀察可以做實，丁自動併入甲

### 路線戊（僅供背景關注）：無條件 UE 的計算版收尾

BBC26 之後的自然題目（boosting、放大、reusability）已被 Bartusek-Goldin 兩篇連發做掉；標準模型古典金鑰 UE 是超出碩論範圍的大魔王。**此戰線只需追蹤、不建議投入。**

### 評估總表

| 路線 | 新穎性窗口 | 風險 | 競速壓力 | 與已讀論文契合 | 建議角色 |
|------|-----------|------|----------|----------------|----------|
| 甲：受限類 QFE/UFE | 大（零競爭者） | 中 | 中（原作者團隊） | ★★★★（四篇全用） | **主軸** |
| 乙：IPFE-SKL | 明確（格子空著） | 低-中 | 高（NTT 團隊產能極高） | ★★★ | **平行保底** |
| 丙：UT-IPFE | 明確 | 低 | 中 | ★★★ | 甲的 fallback 階梯 |
| 丁：IPFE 偽金鑰 | 大 | 中-高 | 低 | ★★★ | 概念備援／甲的深化 |
| 戊：無條件 UE | 已關閉 | — | 極高 | ★★ | 只追蹤 |

---

## 五、建議與下一步

**總體判斷：survey 結果強烈支持維持路徑 1（路線甲）為主軸，且應加快。** 理由：(1) 21 個月無人搶進是反常的空白，不會永遠持續；(2) BMMS26 把「為什麼做受限類」的動機寫進了文獻，introduction 的正當性白送；(3) 你的四篇精讀論文恰好覆蓋這條路線的全部依賴。

具體行動順序：

1. **立即（本月）**：Project A 照計畫寫升級階梯的抽象介面規格，交給老師確認。同時把 Clifford-QOTP-IPFE 直接構造的 sketch 附上——這是「換底層才看得到的新洞察」的第一個候選
2. **短期（1 個月內）**：精讀 ePrint 2025/1842（MLTT compiler），做兩件事：確認路線乙的新穎性防線、學習他們 compiler 式論文的寫法（與你的 Project A 同型）
3. **並行決策點**：Project B 進行約兩個月後評估——若第三層重構順利，丙丁自動退為論文中的 remark/section；若卡關，啟動乙作為保底，丙作為降級目標
4. **持續監控**（每 4–6 週）：ePrint 上 "quantum functional encryption"、"unclonable" 關鍵字；Mehta/Müller/Barhoush/Salvail 四人的新作；NTT 團隊（Kitagawa/Nishimaki/Yamakawa）的 SKL 產線是否碰到 FE

---

## 六、新增文獻速查表（2025-10 之後，按戰線分組）

### QFE / UFE
- **[BMMS26]** Barhoush, Mehta, Müller, Salvail. *On the Impossibility of Simulation Security for Quantum Functional Encryption*. arXiv:2601.17497 (Jan 2026).
- **[BS-BSM]** Barhoush, Salvail. *Simulation-Secure Functional Encryption in the Bounded Storage Model*. arXiv:2309.06702 (v5).

### UE 本體
- **[BBC26]** Bhattacharyya, Broadbent, Culf. *The Uncloneable Bit Exists*. arXiv:2603.08916 (Mar 2026; **引用 v2**, Jun 2026).
- **[BBCNPR26]** Botteron, Broadbent, Culf, Nechita, Pellegrini, Rochette. *Towards Unconditional Uncloneable Encryption*. Quantum 10, 2157 (2026-07-08); arXiv:2410.23064.
- **[BC26]** Bhattacharyya, Culf. *Uncloneable Encryption from Decoupling*. Nature Physics 22(2) (2026); arXiv:2503.19125.
- **[BG26a]** Bartusek, Goldin. *Unclonable Encryption in the Haar Random Oracle Model*. arXiv:2603.11437; ePrint 2026/506 (Mar 2026).
- **[BG26b]** Bartusek, Goldin. *A Note on Boosting Uncloneable Encryption in Microcrypt*. arXiv:2605.27647 (May 2026).
- **[CGKNY26]** Çakan, Goyal, Kitagawa, Nishimaki, Yamakawa. *Multi-Copy Security in Unclonable Cryptography*. ePrint 2025/1921; arXiv:2510.12626. **CRYPTO 2026**.
- **[KY25]** Kitagawa, Yamakawa. *Foundations of Single-Decryptor Encryption*. ePrint 2025/1219. TCC 2025.
- **[BCR25]** Broadbent, Culf, Rochette. *Optimal Untelegraphable Encryption and Implications for Uncloneable Encryption*. arXiv:2510.00903 (Oct 2025).
- **[XOR-MoE]** *The Curious Case of "XOR Repetition" of Monogamy-of-Entanglement Games*. arXiv:2509.01831. ITCS 2026.
- 連續變量 UE：arXiv:2603.06393；裝置無關 UE：Quantum 9, 1582 (2025).

### 金鑰生命週期（SKL / CD / Copy Protection）
- **[KMY25]** Kitagawa, Morimae, Yamakawa. *A Simple Framework for Secure Key Leasing*. arXiv:2410.03413. EUROCRYPT 2025.（PKE：僅需 IND-CPA、古典撤銷；PRF：OWF；簽章：SIS）
- **[PKE/ABE-SKL]** *PKE and ABE with Collusion-Resistant Secure Key Leasing*. ePrint 2025/262; arXiv:2502.12491. ASIACRYPT 2025.
- **[KNP26]** Kitagawa, Nishimaki, Pappu. *Collusion-Resistant Quantum Secure Key Leasing Beyond Decryption*. ePrint 2025/1842; arXiv:2510.04754. **EUROCRYPT 2026**.（MLTT compiler——路線乙動手前必讀）
- **[XT26]** Xu, Takeuchi. *Error-Tolerant Secure Key Leasing*. arXiv:2607.02989 (Jul 2026).
- Copy protection：*Copy-Protection from UPO, Revisited*（ePrint 2025/1880，TCC 2025）；*Copy Protecting Cryptographic Functionalities over Entropic Inputs*（ePrint 2025/1264）；arXiv:2507.19032.

### 古典 IPFE
- *Tightly Secure IPFE Revisited: Compact, Lattice-based, and More*. ePrint 2025/1613. ASIACRYPT 2025.
- *Toward Practical Lattice-based Unbounded IPFE*. ePrint 2025/2232.
- Identity-based IPFE from RLWE（2026，tight，QROM）；去中心化多授權 AB-IPFE（arXiv:2505.11744）.

### 可模糊性 / 偽金鑰周邊
- **[NC-RFE]** Sarkar. *Post-quantum Secure Non-Committing Registered Functional Encryption*. ePrint 2026/1293. AFRICACRYPT 2026.（plain LWE + **equivocal LWE**）
- RNCE with quantum ciphertexts：arXiv:2311.09487；certified everlasting RNCE：arXiv:2207.13878.

---

*本報告由 2026-07-10 的網路檢索彙整而成；ePrint/arXiv 編號均已逐一核對來源頁面。「檢索範圍內未發現」類的新穎性判斷（特別是路線乙、丙）在正式動筆前應以 ePrint 全文檢索 + Google Scholar 引用追蹤再次確認。*
