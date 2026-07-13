# Reports 閱讀指南

> 最後整理：2026-07-13。報告間存在「後文更正前文」的關係；讀任何一份之前，先看本頁確認它的狀態與被更正處。各報告文首均有 ⚠️ 更正/狀態註記，與本頁同步。

---

## 1. 現況摘要（截至 2026-07-13）

- **主軸候選一：路線甲（Unclonable IPFE）**——引用地圖確認零競爭者；首攻路徑為直接合成 P1（`IPFE.Enc(x+r) ⊗ UE.Enc(r)`），最硬技術點是 D2（合法解密者經 r 洩漏 x）；HKNY24 App E 提供 UE 歸約段的現成證明模板。
- **主軸候選二：W1（Unclonable IBE via RNC-IBE 換槽）**——「unclonable IBE」一詞文獻中不存在，證明可套 HKNY24 模板；實質工作在 pq 版 RNC-IBE 與 cloning game 定義。
- **白板線（重構 UE）原形已關閉**——正解 RNCE 已被 HKNY24（TCC 2024）Appendix E 先行做掉（含 unclonable-IND 保持）。剩餘：V1（compiler 基礎化，適合當一章）、V2（高風險）、V3（價值未驗證）。
- **已否定的構造想法**：Clifford-QOTP 直構（可逆性洩漏攻擊，轉為 no-go observation，本身可交付）。

---

## 2. 報告清單與狀態

| 暱稱 | 檔案 | 日期 | 狀態 |
|---|---|---|---|
| **Meeting 討論報告** | [Meeting_2026-07-15_Two_Routes_Discussion.md](./Meeting_2026-07-15_Two_Routes_Discussion.md) | 2026-07-13 | ✅ 現行（meeting 用 high-level 彙整：兩線「要做什麼／為什麼可以／大概怎麼做」＋待拍板問題；細節出處對照見其 §7） |
| QFE survey | [Research_Directions_of_Quantum_Functional_Encryption.md](./Research_Directions_of_Quantum_Functional_Encryption.md) | 2026-05 | ⚠️ 部分過時：方向 1 直構想法被 D4 攻擊否定；Project B/C 判斷已由深度分析更新 |
| 五方向報告 | [Research_Directions_of_Unclonable_Encryption.md](./Research_Directions_of_Unclonable_Encryption.md) | 2026-05 | ⚠️ 部分過時：方向一/三的新穎性判斷被 HKNY24 App E 推翻；方向五結果端被 CGKNY26 拿走 |
| 偽金鑰報告 | [public_key_encryption_with_fake-key_property_report.md](./public_key_encryption_with_fake-key_property_report.md) | 2026-05（07-12 更正） | ⚠️ 核心判斷已更正：「六種範式」應為七種（HKNY24 App E 的 RNCE 路線） |
| 7 月 survey | [Survey_Recent_Developments_and_New_Routes_2026-07.md](./Survey_Recent_Developments_and_New_Routes_2026-07.md) | 2026-07-10 | ⚠️ 兩處更正：漏掉 HKNY24 App E；路線甲的直構想法被否定。其餘文獻盤點與路線評估仍有效 |
| 路線甲深度分析 / Deep Dive | [Deep_Dive_Route_A_Unclonable_IPFE.md](./Deep_Dive_Route_A_Unclonable_IPFE.md) | 2026-07-10 | ✅ 現行（路線甲的技術地圖：P1–P3、D1–D10、K1–K12、12–16 週計畫） |
| 白板報告（v3） | [AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md](./AK21_Hybrid_Recipe_and_PQC_Slot_Replacement.md) | 2026-07-12 | ⚠️ R1/R3 新穎性被 v3 更正推翻；R2（V1）存活；§4 兩線對照與 §1 白板解碼仍有效 |
| AK21 精讀報告 | [AK21_Close_Reading_and_PQC_Slot_Survey.md](./AK21_Close_Reading_and_PQC_Slot_Survey.md) | 2026-07-12（07-13 追加 §4+） | ✅ 現行（**最新結論所在**：E1–E4 介面、HKNY24 查證、V1–V3、W1–W6） |

---

## 3. 建議閱讀順序

**去 meeting／只要結論** → Meeting 討論報告（兩線對照與問題清單）→ 精讀報告 §0/§5 → Deep Dive §0/§6。

**完整脈絡（時間順）**：

1. **QFE survey**（起點：老師兩點建議 + 升級階梯構想）
2. **五方向報告**（UE 端五個前沿的地圖）
3. **偽金鑰報告**（方向一的深入調查）
4. **7 月 survey**（文獻總盤點：空白區/擁擠區 + 路線甲乙丙丁戊評估）
5. **Deep Dive**（路線甲技術深潛：P1 首攻、D1/D4 兩個關鍵發現）
6. **白板報告 v3**（白板線 = 重構 UE 的換槽方向 R1–R4；與路線甲的分界）
7. **精讀報告**（AK21 逐頁精讀 → HKNY24 App E 發現 → 方向轉向建議 + W1）

```
QFE survey ──────────┐
                     ├─→ 7月survey ─→ Deep Dive（路線甲）←─ 證明模板 ─┐
五方向 ─→ 偽金鑰報告 ─┘         └────→ 白板報告 v3 ─→ 精讀報告 ────────┘
（─→ = 承接；精讀報告回頭更正了 五方向/偽金鑰/7月survey 三份）
```

---

## 4. 關鍵事實速查（各報告敘述以此為準）

| 事實 | 定稿版本 | 出處報告 |
|---|---|---|
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

- **P1–P3**：路線甲的三條技術路徑（Deep Dive §3）；**D1–D10**：難點總表；**K1–K12**：知識補完清單（Deep Dive §4–5）
- **E1–E4**：AK21 證明真正消耗的槽位介面（精讀報告 §2）
- **R1–R4**：白板線的四個換槽方向（白板報告 §2–3；R1/R3 已關閉）
- **V1–V3**：白板線剩餘真空格（精讀報告 §4）
- **W1–W6**：第二輪換槽候選（精讀報告 §4+；首選 W1 = unclonable IBE）
- **路線甲～戊**：7 月 survey §四的五條候選路線
