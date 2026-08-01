# Reports 閱讀指南

> 最後整理：2026-08-01。報告間存在「後文更正前文」的關係；讀任何一份之前，先看本頁確認它的狀態與被更正處。各報告文首均有更正/狀態註記，與本頁同步。
>
> **只想知道現在在做什麼** → 直接讀 [路線 W1 技術路線圖](./Route_W1_Unclonable_IBE_Roadmap.md)，其餘全部是通往它的探索紀錄。

---

## 1. 現況摘要（截至 2026-08-01）

- **主軸已定案：W1 — Unclonable IBE**。構造 `ct = (RNCIBE.Enc(mpk, id, k), otUE.Enc(k, m))`；「unclonable IBE」一詞文獻中不存在，證明套 HKNY24 App E 模板。實質工作在 **cloning game 定義**（A1–A5 五個維度）與**後量子 RNC-IBE 零件**（兩案並陳，待拍板）。唯一被識別的技術硬點是 **D-W1**（split 後金鑰查詢 vs stateful 模擬器）。全文見[路線圖](./Route_W1_Unclonable_IBE_Roadmap.md)。
- **路線甲（Unclonable IPFE）已降為備援／延伸章**——引用地圖確認零競爭者，但技術風險較高（D2：合法解密者經 r 洩漏 x）。Deep Dive 報告內容仍有效，作為 W1 撞牆時的退路。
- **白板線（重構 UE）原形已關閉**——正解 RNCE 已被 HKNY24（TCC 2024）Appendix E 先行做掉（含 unclonable-IND 保持）。剩餘 V1（compiler 基礎化）可當論文理論章、與 W1 共用工具；V2 高風險、V3 價值未驗證。
- **已否定的構造想法**：Clifford-QOTP 直構（可逆性洩漏攻擊，轉為 no-go observation，本身可交付）。

---

## 2. 報告清單與狀態

| 暱稱 | 檔案 | 日期 | 狀態 |
|---|---|---|---|
| **W1 路線圖** | [Route_W1_Unclonable_IBE_Roadmap.md](./Route_W1_Unclonable_IBE_Roadmap.md) | 2026-08-01 | **現行主軸文件**（定義維度 A1–A5、主定理、證明骨架與 D-W1、RNC-IBE 精讀、pq 兩案、執行計畫。其 §6 更正了 HMNY21 的發表處與「零量子應用」的說法） |
| **2026-08 Meeting 筆記** | [Meeting_2026-08_W1_RNCIBE_Discussion.md](./Meeting_2026-08_W1_RNCIBE_Discussion.md) | 2026-08-01 | 現行（meeting 用：RNC-IBE 精讀的五個討論點、四個待拍板問題、論文閱讀範圍。「混淆電路的範圍」建議開場先問——它是唯一可能翻掉後量子路徑的變數） |
| **Meeting 討論報告** | [Meeting_2026-07-15_Two_Routes_Discussion.md](./Meeting_2026-07-15_Two_Routes_Discussion.md) | 2026-07-13 | 歷史紀錄：兩線並列的討論定稿。**§4 的三個骨架選項已由 W1 路線圖取代**（主軸定為 W1）；§1–§3 的背景與兩線描述仍有效 |
| QFE survey | [Research_Directions_of_Quantum_Functional_Encryption.md](./Research_Directions_of_Quantum_Functional_Encryption.md) | 2026-05 | 部分過時：方向 1 直構想法被 D4 攻擊否定；Project B/C 判斷已由深度分析更新 |
| 五方向報告 | [Research_Directions_of_Unclonable_Encryption.md](./Research_Directions_of_Unclonable_Encryption.md) | 2026-05 | 部分過時：方向一/三的新穎性判斷被 HKNY24 App E 推翻；方向五結果端被 CGKNY26 拿走 |
| 偽金鑰報告 | [public_key_encryption_with_fake-key_property_report.md](./public_key_encryption_with_fake-key_property_report.md) | 2026-05（07-12 更正） | 核心判斷已更正：「六種範式」應為七種（HKNY24 App E 的 RNCE 路線） |
| 7 月 survey | [Survey_Recent_Developments_and_New_Routes_2026-07.md](./Survey_Recent_Developments_and_New_Routes_2026-07.md) | 2026-07-10 | 兩處更正：漏掉 HKNY24 App E；路線甲的直構想法被否定。其餘文獻盤點與路線評估仍有效 |
| 路線甲深度分析 / Deep Dive | [Deep_Dive_Route_A_Unclonable_IPFE.md](./Deep_Dive_Route_A_Unclonable_IPFE.md) | 2026-07-10 | **降為備援**：技術內容仍有效（P1–P3、D1–D10、K1–K12），但路線甲已非主軸。用途＝W1 撞牆時的退路、論文延伸章、Clifford no-go 短文的來源 |
| 白板報告（v3） | [AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md](./AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md) | 2026-07-12 | R1/R3 新穎性被 v3 更正推翻；R2（V1）存活；§4 兩線對照與 §1 白板解碼仍有效 |
| AK21 精讀報告 | [AK21_Close_Reading_and_PQC_Slot_Survey.md](./AK21_Close_Reading_and_PQC_Slot_Survey.md) | 2026-07-12（07-13 追加 §4+） | 現行（**W1 的出處**：E1–E4 介面、HKNY24 查證、V1–V3、W1–W6。注意：§6 的 HMNY21 發表處與 §3.3「同年同會」已被路線圖 §6 更正；§5.3 的「主軸轉回路線甲」建議已被推翻） |

---

## 3. 建議閱讀順序

**只要現況／要動手做事** → **[W1 路線圖](./Route_W1_Unclonable_IBE_Roadmap.md)（單獨自足，其餘不必讀）**。

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

```
QFE survey ──────────┐
                     ├─→ 7月survey ─→ Deep Dive（路線甲）──（備援）
五方向 ─→ 偽金鑰報告 ─┘         └────→ 白板報告 v3 ─→ 精讀報告 §4+
                                                              │ W1 誕生
                                       Meeting 報告（兩線並列）─┤
                                                              ↓
                                                W1 路線圖（現行主軸）
（─→ = 承接；精讀報告回頭更正了 五方向/偽金鑰/7月survey 三份；
  W1 路線圖回頭更正了 精讀報告 的兩處引用與一處建議）
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
