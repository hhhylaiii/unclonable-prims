# Reports 導覽：報告清單與狀態、建議閱讀順序與關係圖、關鍵事實速查表、縮寫編號索引

> 最後整理：2026-08-14（檔名與標題全面改為描述大綱；新增定義章草稿）。報告間存在「後文更正前文」的關係；讀任何一份之前，先看本頁確認它的狀態與被更正處。各報告文首均有更正/狀態註記，與本頁同步。
>
> **只想知道現在在做什麼** → 直接讀 [Unclonable IBE 主軸路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)，其餘全部是通往它的探索紀錄；定義章的細節見[定義草稿](./Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md)。

---

## 1. 現況摘要（截至 2026-08-07）

- **主軸已定案：W1 — Unclonable IBE**。構造 `ct = (RNCIBE.Enc(mpk, id, k), otUE.Enc(k, m))`；「unclonable IBE」一詞文獻中不存在，證明套 HKNY24 App E 模板。實質工作在 **cloning game 定義**（A1–A5 五個維度）與**後量子 RNC-IBE 零件**（兩案並陳，待拍板）。唯一被識別的技術硬點是 **D-W1**（split 後金鑰查詢 vs stateful 模擬器）。全文見[路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)。
- **定義章已有草稿（2026-08-07）**：上層原語用 game-based（cloning game）、下層槽位用 simulation-based，兩者在 split 線接合；草稿給出 Definition A（搜尋型）與 Definition B（unclonable-IND）、D-W1 的三個解（凍結 Sim₂ 狀態／先 reveal 再查詢／只做 split 前查詢），並主張主定理以 **RNC-IB-KEM（Def 8）**為介面陳述。見[定義草稿](./Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md)。
- **路線甲（Unclonable IPFE）已降為備援／延伸章**——引用地圖確認零競爭者，但技術風險較高（D2：合法解密者經 r 洩漏 x）。Deep Dive 報告內容仍有效，作為 W1 撞牆時的退路。
- **白板線（重構 UE）原形已關閉**——正解 RNCE 已被 HKNY24（TCC 2024）Appendix E 先行做掉（含 unclonable-IND 保持）。剩餘 V1（compiler 基礎化）可當論文理論章、與 W1 共用工具；V2 高風險、V3 價值未驗證。
- **已否定的構造想法**：Clifford-QOTP 直構（可逆性洩漏攻擊，轉為 no-go observation，本身可交付）。

---

## 2. 報告清單與狀態

檔名即大綱；下表的「內容」欄是同一份大綱的展開，「暱稱」欄是各報告內文互相引用時使用的簡稱。

| 報告（點擊即為完整標題） | 暱稱 | 日期 | 內容與狀態 |
|---|---|---|---|
| [Unclonable IBE 主軸路線圖：定義五維度、構造與主定理、證明骨架與 D-W1、後量子兩案](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md) | W1 路線圖 | 2026-08-01 | **現行主軸文件**（定義維度 A1–A5、主定理、證明骨架與 D-W1、RNC-IBE 精讀、pq 兩案、執行計畫。其 §6 更正了 HMNY21 的發表處與「零量子應用」的說法） |
| [Unclonable IBE 定義章草稿：game vs simulation 的分層、四個模擬器對應、定義 A/B 與 KEM 版槽位介面](./Unclonable_IBE_Security_Definition_Draft_Game_and_Simulation.md) | 定義草稿 | 2026-08-07 | 現行，路線圖 §1–§2 的細化：釐清 fake-key 對應的是 Sim₄ 而非 Sim₂、A2 的三個解、E3 是「歸約者在第二步變成模擬器」的邏輯後果、主定理應以 RNC-IB-KEM（Def 8）為介面 |
| [Meeting 筆記（2026-08）：RNC-IBE 介面核對的五個發現與三個待拍板問題](./Meeting_2026-08_RNC_IBE_Interface_Check_and_Open_Decisions.md) | 2026-08 Meeting 筆記 | 2026-08-01 | 現行（meeting 用：E1–E4 逐條對上、Def 6 交出 msk、D-W1 的三條出路、後量子只有 Thm 3 可用、主定理改走 KEM 介面；待拍板＝混淆電路的範圍／pq 甲乙案／split 後查詢是否進定義。「混淆電路的範圍」建議開場先問——它是唯一可能翻掉後量子路徑的變數） |
| [Meeting 報告（2026-07）：兩條候選主軸的做法、可行性、對照表與三個論文骨架選項](./Meeting_2026-07_Two_Candidate_Routes_IPFE_vs_IBE.md) | Meeting 討論報告 | 2026-07-13 | 歷史紀錄：兩線並列的討論定稿。**§4 的三個骨架選項已由 W1 路線圖取代**（主軸定為 W1）；§1–§3 的背景與兩線描述仍有效 |
| [量子函數加密方向 Survey：兩個延伸方向、閱讀清單與「升級階梯 compiler」構想](./Quantum_FE_Directions_and_Upgrade_Ladder_Idea.md) | QFE survey | 2026-05 | 部分過時：方向 1 直構想法被 D4 攻擊否定；Project B/C 判斷已由深度分析更新 |
| [不可複製加密的五個研究前沿：公鑰偽金鑰、演進史、與 NCE 的三角關係、不可複製 FE、一對多推廣](./Unclonable_Encryption_Five_Research_Frontiers_Map.md) | 五方向報告 | 2026-05 | 部分過時：方向一/三的新穎性判斷被 HKNY24 App E 推翻；方向五結果端被 CGKNY26 拿走 |
| [偽金鑰性質為何至今沒有公鑰版：結構性障礙、公鑰 UE 的替代範式、可模糊性概念的角色](./Fake_Key_Property_Why_No_Public_Key_Version_Exists.md) | 偽金鑰報告 | 2026-05（07-12 更正） | 核心判斷已更正：「六種範式」應為七種（HKNY24 App E 的 RNCE 路線） |
| [文獻總盤點（2026-07）：MM24 之後的最新結果、空白區與擁擠區、五條候選路線優先序](./Literature_Survey_2026-07_Results_Gaps_and_Route_Ranking.md) | 7 月 survey | 2026-07-10 | 兩處更正：漏掉 HKNY24 App E；路線甲的直構想法被否定。其餘文獻盤點與路線評估仍有效 |
| [Unclonable IPFE 備援路線深潛：目標語義、MM24 介面需求、路徑 P1–P3、難點 D1–D10、補完清單 K1–K12](./Unclonable_IPFE_Backup_Route_Paths_Obstacles_Knowledge_Gaps.md) | 路線甲深度分析 / Deep Dive | 2026-07-10 | **降為備援**：技術內容仍有效（P1–P3、D1–D10、K1–K12），但路線甲已非主軸。用途＝W1 撞牆時的退路、論文延伸章、Clifford no-go 短文的來源 |
| [白板配方解碼：otUE + PQC 的出處與真實介面、四個換槽方向 R1–R4、與路線甲的分界](./Whiteboard_Recipe_Decoded_and_PQC_Slot_Swap_Directions.md) | 白板報告（v3） | 2026-07-12 | R1/R3 新穎性被 v3 更正推翻；R2（V1）存活；§4 兩線對照與 §1 白板解碼仍有效 |
| [AK21 逐頁精讀與槽位替換品盤點：介面 E1–E4、HKNY24 App E 查證、真空格 V1–V3、候選 W1–W6](./AK21_Close_Reading_Slot_Interfaces_and_Route_Candidates.md) | AK21 精讀報告 | 2026-07-12（07-13 追加 §4+） | 現行（**W1 的出處**：E1–E4 介面、HKNY24 查證、V1–V3、W1–W6。注意：§6 的 HMNY21 發表處與 §3.3「同年同會」已被路線圖 §6 更正；§5.3 的「主軸轉回路線甲」建議已被推翻） |

---

## 3. 建議閱讀順序

**只要現況／要動手做事** → **[W1 路線圖](./Unclonable_IBE_Main_Roadmap_Definition_Construction_Proof.md)（單獨自足，其餘不必讀）**。

**要知道為什麼是 W1**（三份，約 40 分鐘）→ 精讀報告 §3（HKNY24 App E 的發現）→ 精讀報告 §4+（第二輪盤點與 W1 的誕生）→ W1 路線圖。

**完整脈絡（時間順）**：

1. **QFE survey**（起點：老師兩點建議 + 升級階梯構想）
2. **五方向報告**（UE 端五個前沿的地圖）
3. **偽金鑰報告**（方向一的深入調查）
4. **7 月 survey**（文獻總盤點：空白區/擁擠區 + 路線甲乙丙丁戊評估）
5. **Deep Dive**（路線甲技術深潛：P1 首攻、D1/D4 兩個關鍵發現）
6. **白板報告 v3**（白板線 = 重構 UE 的換槽方向 R1–R4；與路線甲的分界）
7. **精讀報告**（AK21 逐頁精讀 → HKNY24 App E 發現 → W1 的提出）
8. **Meeting 報告**（兩線並列的定稿，供老師討論）
9. **W1 路線圖**（主軸定案，RNC-IBE 精讀落地）
10. **定義草稿**（路線圖 §1–§2 的細化：定義風格分層、模擬器對應、定義 A/B）

```
QFE survey ──────────┐
                     ├─→ 7月survey ─→ Deep Dive（路線甲）──（備援）
五方向 ─→ 偽金鑰報告 ─┘         └────→ 白板報告 v3 ─→ 精讀報告 §4+
                                                              │ W1 誕生
                                       Meeting 報告（兩線並列）─┤
                                                              ↓
                                                W1 路線圖（現行主軸）
                                                              ↓
                                                定義草稿（定義章落地）
（─→ = 承接；精讀報告回頭更正了 五方向/偽金鑰/7月survey 三份；
  W1 路線圖回頭更正了 精讀報告 的兩處引用與一處建議；
  定義草稿更正了「fake-key 對應 Sim₂」這個容易搞混的對應）
```

---

## 4. 關鍵事實速查（各報告敘述以此為準）

| 事實 | 定稿版本 | 出處報告 |
|---|---|---|
| **RNC-IBE = GKKNRY, PKC 2025**。Def 6 adaptive／Def 7 selective；模擬器 Sim₁–Sim₄，**Sim₄ 輸出的是 msk 不是 sk_{id\*}** | 挑戰階段對手收到 (msk, ct\*)——W1 因此可做「msk 洩漏仍不可複製」的強版 | W1 路線圖 §3 |
| **後量子 RNC-IBE 的唯一現成選項 = 其 Thm 3 取 X = LWE**：adaptive 安全、mpk compact、**身分空間僅多項式大小 T**、密文 poly(T, \|m\|, λ)。主構造 Thm 1 是 SXDH（非 pq）；Thm 5/6 需 iO | 別誤記為 selective；relaxed 版是 adaptive 的 | W1 路線圖 §3.2、§4 |
| **HMNY21 = ASIACRYPT 2021**（非 TCC 2021），全名 *Quantum Encryption with Certified Deletion, Revisited: Public Key, Attribute-Based, and Classical Communication*；NC-ABE 出自此文（用 iO），動機是**憑證刪除**不是不可複製性 | 更正精讀報告 §6 的 TCC 2021 與 §3.3 的「同年同會」 | W1 路線圖 §6 (C1) |
| 「RNC-IBE 零量子應用」需精確化：**該篇論文**未做量子應用，但 NC-ABE 這個概念本來就是為量子應用（憑證刪除）而生 | 競速風險應上調 | W1 路線圖 §6 (C2) |
| HKNY24 = arXiv:2311.09487 = *Robust Combiners and Universal Constructions for Quantum Cryptography*（TCC 2024）。§7 UE combiner、§8 明文擴展、**App E：RNCE ⇒ unclonable PKE（Lemma E.2 保 unclonable-IND）** | 三個章節是同一篇論文，勿當三篇 | 精讀報告 §3 |
| AK21 證明真正消耗的槽位介面是 E1–E4（trapdoored 可模糊化即足夠），Def 16（無 trapdoor + 誠實密文）過強 | Bendlin 牆咬的是過強版，證明不需要它 | 精讀報告 §2 |
| AK21 假設已最優（私鑰 OWF／公鑰 PKE），「換槽」不可能是降假設 | — | 白板報告 §2 |
| Clifford-QOTP 直構有可逆性洩漏攻擊（L_C 可逆 ⇒ 取回 ρ 本身） | 已從構造候選轉為 no-go observation | Deep Dive §3.3（D4） |
| MM24 Thm 7 lifting 消耗 universality，對 IPFE 類不適用 as stated | 換底層的真正工作量在重建 lifting | Deep Dive §2（D1） |
| BBC26《The Uncloneable Bit Exists》：**引用 v2**（2026-06，v1 分析有瑕疵）；Haar 隨機酉、**無有效構造** | 不能直接當 lifting 的底座（違反 poly 描述需求） | 7 月 survey §2.2；Deep Dive §3.2（D6） |
| BC26（Bhattacharyya-Culf, decoupling）：arXiv 2025，Nature Physics 正式刊出為 **2026-02**，逆多項式安全 | 引用年份以 2026 為準 | 7 月 survey §2.2 |
| CGKNY26 多複製安全：**CRYPTO 2026**（ePrint 2025/1921） | 「結果」已關閉；「一對多偽金鑰」技術路徑仍空 | 7 月 survey；白板報告 R4 |
| MM24 仍是 ePrint 預印本（2024/1683），未落地會議；引用它的僅約五篇、無構造端跟進 | 路線甲零競爭的依據（需每季複查） | 7 月 survey §2.1 |
| AKY24 = ITCS 2025（2024 年流傳） | 統一寫 ITCS 2025 | — |
| Somewhere equivocal encryption = [HJO+16] Hemenway-Jafargholi-Ostrovsky-**Scafuro**-Wichs, CRYPTO 2016 | 作者含 Scafuro（非 Sahai） | — |
| unclonable-IND（Def 12，強概念）one-time 現況：QROM 有 AKLL22；標準模型仍開放（領域大魔王） | compiler 的 one-time 原料現況 | 精讀報告 §3.1 |

---

## 5. 常用縮寫/編號索引

**現行主軸（W1 路線圖）**

- **A1–A5**：unclonable IBE 定義設計的五個維度（§1.2）——不可複製性強度／查詢時機／reveal 給什麼／selective vs adaptive／挑戰身分規則
- **D-W1**：本路線唯一被識別的技術硬點——split 後金鑰查詢 vs RNC-IBE 的 stateful Sim₂（§2.3）
- **甲案／乙案**：後量子零件的兩條策略——poly-ID 誠實起步／自造 LWE selective RNC-IBE（§4.3）
- **C1–C4**：W1 路線圖對既有報告的四項更正（§6）

**歷史編號**

- **P1–P3**：路線甲的三條技術路徑（Deep Dive §3）；**D1–D10**：難點總表；**K1–K12**：知識補完清單（Deep Dive §4–5）
- **E1–E4**：AK21 證明真正消耗的槽位介面（精讀報告 §2）——W1 路線圖 §2.4 逐條核對過 RNC-IBE
- **R1–R4**：白板線的四個換槽方向（白板報告 §2–3；R1/R3 已關閉）
- **V1–V3**：白板線剩餘真空格（精讀報告 §4；V1 可當論文理論章）
- **W1–W6**：第二輪換槽候選（精讀報告 §4+）——**W1 = unclonable IBE 已升為主軸**
- **路線甲～戊**：7 月 survey §四的五條候選路線（路線甲 = Unclonable IPFE，現為備援）
