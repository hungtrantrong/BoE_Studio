# BoE Studio – Phản biện tài liệu & đối chiếu với file Excel thật

> Ngày: 05-Nov-2026 | Đầu vào: `BoE_Studio_Knowledge_Base.md`, deck `BoE Studio — từ business đến kiến trúc.html` (03-Nov), Mockup, Prototype, và **3 file Excel đọc được trong repo**: COO `30897 Buu dien Kon Tum_v4` (xlsx), COO `HCSBDU0037 Nguyen Tri Phuong Di An_v2` (xlsm), GB `03__GB_40_Vuong_Thua_Vu_RE_slim` (xlsm, 17 sheet, 13 sheet ẩn).
>
> Mục đích: tài liệu để trao đổi với Finance/Expansion/Dev, không phải đặc tả. Mọi con số đều trích từ ô cụ thể trong file để có thể kiểm tra lại.

---

## 0. Tóm tắt điều hành

1. **Mục 6 của Knowledge Base (engine) không còn là giả định nữa.** 3 file trong repo đọc được đầy đủ công thức. Khoảng 70% câu hỏi [MỞ]/[GĐ] về cách tính đã có câu trả lời (xem mục 2). Nên cập nhật KB trước khi làm đặc tả engine.
2. **Template có lỗi phương pháp luận, không chỉ lỗi tham chiếu.** Với file Kon Tum, IRR 10Y dao động từ **44,6% (file) → 47,7% → 50,3% → 55,8%** tuỳ sửa 3 lỗi: trừ hai lần pre-opening opex, hiệu ứng "đóng cửa 1 tháng" chạy mỗi năm, và IRR đang là **sau lãi vay & sau thuế** (ngược với quyết định [CHỐT] "IRR pre-interest"). Deck 03-Nov mới nêu 2 trong số này.
3. **Ngưỡng duyệt hiện không có chủ.** Mỗi file một bộ ngưỡng dán tay (Kon Tum ROIC 42,8% / S-C 2,07; Dĩ An 50,7% / 2,14; GB 35% / 1,5 hard-code đè lên bảng lookup). Không file nào có chiều "vùng". Đây là master data số 1 phải chốt, trước cả engine.
4. **Decision rule "ít nhất 1 chỉ số vượt ngưỡng bằng cách chỉnh ADS/diện tích/vốn"** là lỗ hổng governance lớn nhất của thiết kế. Cần rào chắn mạnh hơn mức "gắn cờ P75" trong deck.
5. **RRR trong file không phải "kế hoạch do Finance kích hoạt"** mà là lịch mặc định theo % CAPEX ban đầu (0,3%/năm + 3% mỗi 2 năm + 20% năm 5 + 70% năm 7 + 15% năm 9) và kèm bug. Thiết kế RRR trong KB và thực tế đang mô tả hai thứ khác nhau; cần thống nhất.
6. **GB và COO hợp nhất được bằng cấu hình**: khung P&L, ramp-up, khấu hao, thuế, vay, RRR giống nhau ~90%. Khác biệt gói gọn ở: Royalty 5%, Franchise & opening fee (vốn hoá, khấu hao 5 năm), cách tính construction, và GB có master data theo format (sheet Simulation) còn COO chưa có. Template COO thậm chí đã có trường `Entity (HC or GB)`.
7. **Thiếu trong cả KB và deck**: kiểm tra sức chứa theo daypart (có trong Excel), thời hạn thuê 5–6 năm vs mô hình 10 năm, thuế không chuyển lỗ, deposit không hoàn, NPV chưa từng tồn tại trong file, mapping format cũ.

---

## 1. Phạm vi và cách làm

- Đọc toàn bộ công thức (không chỉ giá trị) của sheet `Core` ở cả 3 file; sheet `Direction`, `Summary2`, ` Simulation.29.12.2025` của GB.
- Diff công thức Kon Tum vs Dĩ An (58 ô khác công thức, cùng template "Core" 487 dòng).
- Tính lại IRR bằng Python với các kịch bản sửa lỗi để định lượng ảnh hưởng.
- Đối chiếu với deck 03-Nov (15 slide) và mục "Tham khảo: cách tính không thống nhất" trong Mockup để **không lặp lại** những gì đã biết, chỉ bổ sung và phản biện.
- Không đọc được: 13 file còn lại (BE/, Updated_BE/) vì không có trong repo. Nhận định về "16 file" lấy từ deck.

---

## 2. Những gì file Excel trả lời cho các câu hỏi [MỞ]/[GĐ] trong KB

| # | KB nói gì | File thật nói gì (ô tham chiếu, file Kon Tum trừ khi ghi khác) | Hệ quả cho thiết kế |
|---|---|---|---|
| 2.1 | ADS = ADTC × AC **[GĐ]** | Đúng: `D35 = D36*D37/1000` (ADS triệu VND, AC nghìn VND). TC tháng = ADTC × 30 ngày (`W7:AN7 = 30`, 360 ngày/năm). | Chốt đơn vị: ADS tr.VND/ngày, AC nghìn VND, 30 ngày/tháng. |
| 2.2 | Ramp-up một đường cong **[CHỐT]** | Ramp-up 6 tháng trên **ADTC** (AC giữ nguyên): COO `W10:AB10 = 0.8, 0.9, 0.95, 1.0, 0.9, 0.95` (không đơn điệu!); GB `S10:X10 = 0.5, 0.75, 0.8, 0.85, 0.9, 0.95`. 6 tháng này gọi là "Y0", đánh nhãn Month −6…−1. | Ramp-up là tham số master data theo BU/format (6 giá trị). COO curve có vết sửa tay (tháng 5–6 tụt) → phải hỏi lại Finance. Nhãn "Y0 = 6 tháng đầu hoạt động" cần giữ hoặc đổi tên rõ ràng. |
| 2.3 | SSSG **[GĐ, MỞ]** | Có: tăng trưởng áp lên **ADTC** 7%/năm Y1–Y6, 5%/năm Y7–Y10 (`C66:C75`); **AC phẳng 10 năm**. Rent tăng theo hợp đồng (`D66:D75`), lương +10% mỗi 2 năm (`C77:C86`), % COGS/MKT/CMS/OCE phẳng. | Mô hình đang trộn: traffic tăng 7% nhưng giá bán không tăng, lương tăng 10%/2 năm. Cần Finance xác nhận đây là chủ ý (bảo thủ) hay sót. |
| 2.4 | LC, OCE bán cố định, chưa biết mô hình **[MỞ]** | **LC = số tiền cố định/tháng** suy từ % mục tiêu (`D45 = Revenue_target × 14,18%`, rồi `W21:AN21 = $U21` phẳng cả trong ramp-up), tăng theo wage inflation hằng năm, **không theo doanh thu**. **OCE = % doanh thu** (`W24 = W17 × 9,26%`). Rent cố định. MKT, CMS = % doanh thu. Dĩ An có bảng nhân sự bottom-up (`H37:L40`: ca × 2 × lương) nhưng **không nối** vào % LC (D56 = 0,14 gõ tay). | Câu trả lời sẵn có: LC fixed-from-target, OCE variable. Đề xuất engine: LC = max(bottom-up headcount × lương, %LC × ADS) với cả hai hiển thị; OCE tách phần cố định (điện nước, bảo vệ) nếu Finance muốn. |
| 2.5 | Royalty GB % doanh thu **[GĐ công thức]** | GB: `D62 = 5%` doanh thu, dòng riêng trong P&L (`O25`). Thêm **Franchise fee & opening fee = (268,92 + 224,1)/1,08 = 456,5 tr** (`L45`), vốn hoá, khấu hao 5 năm (`Q113`), nằm trong mẫu số ROIC. COO không có cả hai. | Hai tham số BU-level: `royalty_pct`, `franchise_fee` (+ cờ có đưa vào mẫu số). |
| 2.6 | Khấu hao **[GĐ]** | Xây dựng 5 năm; thiết bị 50% 5 năm + 50% 3 năm; pre-opening vốn hoá 3 năm; GB franchise fee 5 năm. Y0 tính 6 tháng (`W106 = U106/5/12*6`). Reinvestment khấu hao 5 năm từ năm chi. | Bảng `depreciation_policy` theo cấu phần CAPEX. |
| 2.7 | ROIC: lợi nhuận nào, mẫu số gì **[MỞ]** | **Tử số = EBITDA "Year 1"** (`AO26` = tổng 12 tháng *sau* 6 tháng ramp-up, tức tháng 7–18). Mẫu số Kon Tum: Key money + Xây dựng + Thiết bị + Pre-opening − MKT fee (`D5`); Dĩ An: Xây dựng + Thiết bị; GB: Key money + Xây dựng + Thiết bị + Pre-opening + Franchise fee − MKT fee. Deposit không file nào tính. | Tử số đã rõ: EBITDA năm ổn định đầu tiên. Nên đặt tên KPI đúng bản chất ("Y1 EBITDA/Investment", cash-on-cash) để khỏi nhầm với ROIC = NOPAT/IC. |
| 2.8 | Sales/CAPEX dùng năm nào **[MỞ]** | Doanh thu "Year 1" (tháng 7–18) / cùng mẫu số với ROIC (`D8`). | Hai KPI dùng chung mẫu số → chỉ cần 1 bộ cờ. |
| 2.9 | IRR pre-interest **[CHỐT]** | **Sai so với file.** FCF = CFO + CFI, trong đó CFO = NI + Dep + reinvestment effect; NI = NOI − **lãi vay** − **CIT** (`W83`, `W85`, `W87`). Vay 70% vốn, lãi 10%, 5 năm (`W197`, `W200`, `W201`) nhưng **không có dòng CFF** trong FCF dùng cho IRR. Tức là IRR hiện tại = dòng tiền unlevered trừ lãi vay = sai cả hai hướng. | Quyết định "pre-interest" là đúng về phương pháp, nhưng engine phải nêu rõ đây là **thay đổi kết quả** (+~5,5 điểm IRR ở Kon Tum), không phải "sửa bug nhỏ". Finance cần biết trước khi so sánh số cũ/mới. |
| 2.10 | NPV, discount rate **[MỞ]** | **Không file nào có NPV.** | NPV là KPI mới hoàn toàn → cần định nghĩa từ đầu (dòng tiền nào, kỳ hạn, terminal value có/không). |
| 2.11 | RRR: thời điểm, khấu hao, ảnh hưởng doanh thu **[MỞ]** | Lịch mặc định % CAPEX ban đầu: `D77:D86 = 0,3% / 3,3% / 3,3% / 3,3% / 20,3% / 3,3% / 70,3% / 3,3% / 15,3% / 3,3%` (GB Simulation `L277` ghi rõ: "0,3% replacement; 3% refresh mỗi 2 năm; 20% năm 3 minor innovation; 70% năm 7 major"). Chi vào CFI (`X86 = −X125`), khấu hao 5 năm. Ảnh hưởng doanh thu: `X84 = (−GP + 35%·LC + 35%·OCE)/12 × (1−20%)` khi RRR > 0 = mất 1 tháng GP, tiết kiệm 65% LC+OCE. | Xem bug 3.2. Thiết kế RRR nên theo **sự kiện** (loại, số tiền, số ngày đóng, đời khấu hao), lịch % chỉ là mặc định. |
| 2.12 | Thời gian dự báo, lạm phát **[MỞ]** | 10 năm (Y0 + Y1…Y10). Lạm phát chỉ ở lương (10%/2 năm) và rent (theo hợp đồng). Tỷ giá 25.000 (`D2`). CIT 20%. | Horizon 10 năm là chuẩn; cần tham số hoá tỷ giá, CIT. |
| 2.13 | Thiết bị chuẩn theo format **[CHỐT]** | GB có: `Summary2!D30:G33` Equipment / Tools / IT / Setup theo 4 format (ví dụ Core: 700 + 51 + 308 + 143 = 1.202 tr). COO: một số `H44 = 1203` gõ tay (Dĩ An 1220). | Seed master data thiết bị từ GB; COO hiện chưa có chuẩn. |
| 2.14 | CAPEX xây dựng gồm phá dỡ, cầu thang **[CHỐT]** | COO Kon Tum: `I43 = (8,575tr/m² × 150 + 3,25tr × 50 ngoài trời + shipping 0,75tr × 200 + FSC 50tr + façade 2,05tr×18 + vách kính 2,6tr×2×18 + **Addin 471,3tr**)`, trong đó `O54 = N54 − J43 = 0` là ô kiểm tra khớp với **báo giá 2.250.586.326** gõ ở `N54`. Dĩ An công thức khác (`H43*H23 + P47 + P54*O17 + P51*H24`). | Construction thực chất là **báo giá nhập vào**, công thức chỉ để back-solve. Engine nên nhận "báo giá" làm input, chuẩn/m² theo format làm **tham chiếu + cảnh báo lệch**, không cố mô hình hoá từng hạng mục. |
| 2.15 | Comparable dựa diện tích **[CHỐT]** | Diện tích trong file: Leasing 300 m², Investment 200 m² (= Inside 150 + Outside 50), BOH 28%, FOH gross 144, dead space 15%, FOH net 122,4, **1,5 m²/ghế → 81,6 ghế** (`J21`, `L21`). KB nói 1,2–1,5 ghế/m² — **ngược chiều** với file (file: 1 ghế/1,5 m² ≈ 0,67 ghế/m²). | Sửa KB: 1,5 m²/ghế trên FOH net. Dùng "Investment size" (diện tích đầu tư) làm diện tích so sánh, không phải leasing size. |
| 2.16 | Dimension vùng/format | Dimension có trong file: Store format (Core/Flagship/Small/Kiosk), **Location type** (`D13`: "T3 Central", "Non CBD"), **Trade Area type** (`D17`: "Residential"), Entity (HC/GB). **Không có Region/City.** Địa chỉ dạng text. | "Vùng" phải định nghĩa mới. Location type và trade area type là dimension sẵn có, chưa xuất hiện trong KB. |

---

## 3. Lỗi và điểm yếu của template (có định lượng)

Mức độ: 🔴 ảnh hưởng quyết định, 🟠 ảnh hưởng số liệu, 🟡 nhất quán/bảo trì.

| # | Mức | Lỗi | Bằng chứng | Ảnh hưởng (Kon Tum) |
|---|---|---|---|---|
| 3.1 | 🔴 | **Pre-opening opex bị trừ hai lần** | Phần opex (213 tr: LC 75 + MKT 25 + Travel 113) cộng vào P&L tháng đầu (`W21 = U21 + R21`, `W22`, `W24`) → giảm EBITDA → NI → CFO; đồng thời **toàn bộ** 333 tr pre-opening nằm trong `D32` → `W86 = −D32` (CFI). | IRR 44,6% → 47,7% nếu sửa. |
| 3.2 | 🔴 | **"Hiệu ứng đóng cửa" chạy mỗi năm** | `X84 = IF(X49>0, …)`; `X49 = 0,3%` (replacement) năm nào cũng > 0 → năm nào cũng mất 1 tháng GP (−239 → −410 tr/năm, ≈ **11% EBITDA mỗi năm**). | IRR 47,7% → 50,3% nếu chỉ áp khi refresh/renovation. |
| 3.3 | 🔴 | **IRR sau lãi vay, sau thuế nhưng không có CFF** | Mục 2.9. | 50,3% → 55,8% nếu unlevered. Tổng 3 lỗi: **+11 điểm IRR**. |
| 3.4 | 🔴 | **Ngưỡng dán tay, mỗi file một bộ** | Kon Tum `E3:E8` = 21,8% / 8,2% / 42,8% / 3,81 / 2,83 / 2,07; Dĩ An 24,4% / 0 / 50,7% / 0 / 0 / 2,14; GB `E5 = SUMPRODUCT(...)*0 + 35%`, `E8 = ...*0 + 1,5` (bảng Direction bị vô hiệu hoá bằng `*0`). | Không thể nói store "đạt" theo chuẩn nào. |
| 3.5 | 🔴 | **Cờ IRR đảo chiều** (đã biết) | Kon Tum `F3 = IF(E3>=D3,"Yes","No")`; Dĩ An `IF(D3>=E3,…)`; GB `IF(E3>=D3,"No","Yes")`. 3 file 3 kiểu. Dĩ An thêm `F6/F7` payback "≥ ngưỡng = Yes" với ngưỡng 0. | Cờ không dùng được. |
| 3.6 | 🟠 | **Y0 = 6 tháng nhưng IRR/Payback xem như 1 kỳ năm** | `D3 = IRR(W87:AG87)` 11 kỳ, kỳ đầu 6 tháng. Payback `D6` trừ 2 rồi cộng phân số → lệch ~0,5 năm. | IRR hơi cao, payback hơi thấp; so sánh giữa store vẫn công bằng nhưng tuyệt đối sai. |
| 3.7 | 🟠 | **Thuế không chuyển lỗ** | `X82 = IF(NOI − interest < 0, 0, …×20%)` từng năm độc lập. | Store lỗ Y0 (GB −113 tr) không được khấu trừ. |
| 3.8 | 🟠 | **Rent theo hợp đồng ≠ tham số** | `D19` text: "Năm 1,2: 30 tr; Năm 3–5: 33 tr" nhưng `D68 = 10%` Y3 **và** `D70 = 10%` Y5 → Y5 = 36,3 tr. Sau Y5 rent phẳng 5 năm dù hợp đồng 5 năm (`D18`) — không có giả định gia hạn. | Rent Y5–Y10 sai ~10%; horizon 10 năm trên hợp đồng 5 năm không có rủi ro tái ký. |
| 3.9 | 🟠 | **Mẫu số trừ MKT fee** | `−H58/10^6` chỉ trừ MKT fee (25 tr) khỏi mẫu số, không trừ các opex pre-opening khác. Không có logic. | Nhỏ nhưng vô lý; bỏ. |
| 3.10 | 🟠 | **Ramp-up COO không đơn điệu** | `W10:AB10 = 0,8 / 0,9 / 0,95 / 1,0 / 0,9 / 0,95`. Dĩ An giống hệt → là template, không phải riêng store. | Có thể là seasonality lén đưa vào ramp-up, trái với "một đường cong, không seasonality". |
| 3.11 | 🟠 | **Hằng số cộng tay vào % chi phí (GB)** | `D55 = lookup + 2,9% − 0,2%`, `D61 = lookup − 0,7% + 1,5%`, `D58 = 12,5% + 0,6%`. | Master data bị "vá" ngay trong file store. |
| 3.12 | 🟠 | **Bảng master GB có ô gãy** | `Direction!W7` EBITDA margin Core = −47,13; `Summary!E6` ADS Core = 4,05 trong khi `Simulation!I20` = 16,985. | Seed master data từ GB phải qua kiểm tra, không tin số đã lưu. |
| 3.13 | 🟡 | **Deposit không hoàn, Key money không khấu hao** | Deposit 140 tr vào CFI Y0 (`D32`), nằm trong BS 10 năm, không có dòng hoàn. | Payback/IRR bảo thủ hơn thực tế; nếu muốn hoàn thì phải có lease-end logic. |
| 3.14 | 🟡 | **`#REF!`, named range rác** | `A88 = #REF!` (Kon Tum), `A91 = #REF!` (GB); >80 defined names dạng `______NOS4`. | Dấu hiệu copy-paste qua nhiều đời file. |
| 3.15 | 🟡 | **Bảng nhân sự không nối với %LC** (Dĩ An) | `H40 = SUMPRODUCT(...)` tính lương tháng nhưng `D56 = 0,14` gõ tay. | Mất kiểm tra chéo top-down/bottom-up. |
| 3.16 | 🟡 | **Diff ẩn giữa hai file cùng template** | 58 ô khác công thức: `D5`, `D8` (mẫu số), `I43` (construction), `H16/H23/H24` (hard-code vs formula), `I44` (thiết bị), `D28`, `D30`. | "Updated_BE vs BE" chắc chắn cũng lệch công thức, không chỉ số liệu. |

---

## 4. Phản biện các quyết định đã [CHỐT]

### 4.1 "Finance điều chỉnh ADS/diện tích/vốn để ít nhất một chỉ số vượt ngưỡng" (KB 2.5)
- Đây là **back-solving để qua cửa**, và hệ thống còn định tính "khoảng cách tới ngưỡng" (deck slide 10). Cộng với rule "chỉ cần 1 trong 2 đạt là review riêng", xác suất một store yếu được đẩy qua bằng cách nâng ADTC vài % là rất cao.
- Bằng chứng từ file: Kon Tum ADTC 203 vs chuẩn Core GB 215; chỉ cần ADTC 190 là Sales/CAPEX rơi dưới 2,07 nhưng ROIC vẫn trên 42,8% → vào ô "review riêng" chứ không bị chặn. Tức rule C (chặn cứng) gần như không bao giờ kích hoạt với một người biết chỉnh số.
- Đề xuất rào chắn (cần Hùng + Finance chọn):
  1. ADS/ADTC/AC của Final **phải nằm trong dải P25–P75 của comparable** hoặc phải có lý do + người phê duyệt ngoài hệ thống ghi tên; ngoài dải → tự động chuyển trạng thái B dù cả hai KPI đạt.
  2. Khi "chỉnh vốn đầu tư" để đạt → số vốn đó trở thành **trần CAPEX** truyền sang SLM/SDC; vượt trần khi thi công → tự động Revise.
  3. Ghi "số trước khi chỉnh" (first-pass KPI) trong snapshot, hiển thị cạnh số Final. Người duyệt nhìn thấy store "được chỉnh từ 1,9 lên 2,1".
  4. Cân nhắc rule: A chỉ khi **cả hai** đạt **và** ADS ≤ P75; còn lại B; C khi cả hai hụt.

### 4.2 "IRR chỉ tham khảo" + "IRR pre-interest"
- Trong cả 3 file, IRR 10Y là KPI **đứng đầu** bảng và có ngưỡng. Finance đang dùng nó. Nếu BoE Studio hạ IRR xuống "tham khảo" thì cần nói rõ với Finance là ai quyết định việc này; nếu không, người dùng sẽ nhìn IRR mới (cao hơn 11 điểm vì sửa lỗi) và mất niềm tin vào engine ("sao số khác thế").
- Đề xuất: Phase 0 engine xuất **cả hai**: `irr_legacy` (tái hiện đúng file, kể cả lỗi, để golden test) và `irr_unlevered` (chuẩn mới). Bảng đối chiếu từng store có cột "chênh do lỗi 3.1 / 3.2 / 3.3". Đây là cách duy nhất để Finance chấp nhận số mới.

### 4.3 "Ngưỡng theo format × vùng"
- Không file nào có "vùng". Thứ khác nhau theo địa lý trong dữ liệu hiện có là **Lease/Sqm** (`Direction!AF4:AH5`: expected vs actual theo format) và Location type (CBD/T3/Non-CBD).
- Câu hỏi phản biện: ngưỡng ROIC có thật sự cần khác theo vùng, hay vùng chỉ ảnh hưởng **đầu vào** (rent/m², ADS kỳ vọng, chi phí xây)? Nếu là đầu vào thì ngưỡng giữ theo format, và master data L1 (rent/m², capex/m²) mới theo vùng. Làm vậy số chiều của bảng ngưỡng giảm từ 4×N xuống 4, dễ chốt hơn nhiều.
- Nếu Finance vẫn muốn ngưỡng theo vùng: cần định nghĩa vùng (Bắc/Trung/Nam/Tây Nguyên? theo tỉnh? theo cụm thành phố?) và ai sở hữu danh sách — hiện chưa có ở đâu.

### 4.4 "ROIC denominator cấu hình bằng tick"
- Đồng ý cơ chế cờ, nhưng **không nên để mặc định mở cho người dùng tick trên từng BoE** — nếu vậy sẽ tái hiện đúng tình trạng "6 cách tính" của Excel, chỉ là trong database thay vì trong file.
- Đề xuất: bộ cờ là thuộc tính của **master data version theo BU** (Finance chốt, có hiệu lực), BoE chỉ hiển thị, không sửa. Người dùng muốn xem mẫu số khác → what-if, không lưu vào KPI chính thức.
- Mặc định đề xuất (khác deck): **Xây dựng + Thiết bị + Pre-opening vốn hoá + Franchise fee (GB) + Key money**; loại Deposit (hoàn được) và pre-opening opex (đã nằm trong P&L Y0). Lý do: đây là phần vốn *sunk* và cũng chính là cơ sở khấu hao → ROIC và khấu hao nhất quán. Với Kon Tum: 0,559 (file 0,563; constr+equip 0,613; all-in 0,540) — quyết định không đổi nhưng ở store biên thì đổi.

### 4.5 "Không có approval workflow"
- Nhưng rule C "chặn khoá Final" + trạng thái A/B/C + "ghi kết quả review" (deck slide 10) đã là một workflow tối giản. Vấn đề không phải có hay không, mà là **ai được bấm Lock Final và hệ thống có kiểm tra điều kiện trước khi lock không**. Cần nói rõ: lock = hành động của FP&A, điều kiện = KPI tính xong + trạng thái ≠ C + comparable set đã chọn. Đó là "guard", không phải "workflow", và nên ghi vào KB để sau này không bị hiểu là scope creep.

### 4.6 "Expansion sở hữu L0/L1" vs "Finance quyết định giá trị"
- KB (3-Nov) nói Expansion sở hữu; deck slide 7 phân người quyết định cho từng nhóm (Finance quyết hurdle, royalty, CIT; team thiết bị quyết thiết bị; Group quyết discount rate). Hai tài liệu không mâu thuẫn nhưng dùng từ "sở hữu" khác nhau. Cần tách rõ 3 vai cho mỗi nhóm tham số: **quyết định giá trị / nhập & bảo trì / được xem**. Nếu không, lúc implement quyền sẽ cãi nhau.

### 4.7 "Ramp-up một đường cong, không seasonality"
- COO curve `0,8/0,9/0,95/1,0/0,9/0,95` tụt ở tháng 5–6 → hoặc là seasonality đã lọt vào, hoặc là lỗi. GB curve đơn điệu `0,5→0,95`. Hai BU ramp khác nhau rất xa (GB tháng 1 chỉ 50%). Khi hợp nhất, ramp-up phải là tham số theo **BU × format**, và cần Finance xác nhận curve COO.

### 4.8 "Comparable: diện tích làm proxy doanh thu"
- Dữ liệu GB cho thấy quan hệ diện tích–ADS **không tuyến tính** theo format: Flagship 250 m² → ADS 38,3; Core 165 m² → 17,0; Small 110 m² → 10,6; Kiosk 40 m² → 7,2 (`Simulation!H20:K20`, `H66:K66`). Kiosk 40 m² ra 7,2 còn Small 110 m² ra 10,6 → ADS/m² của Kiosk gấp 2 lần Small. Nên comparable **phải lọc cứng theo format trước**, rồi mới dùng diện tích; deck đã làm đúng, KB cần sửa thứ tự.
- Thêm một kiểm tra rẻ mà file đã có: **sức chứa theo daypart** (`H28:L34`): ghế cần ở giờ cao điểm = ADTC × %daypart × group size / turns / utilization. Kon Tum: Mid-morning cần 73 ghế / có 81,6. Đây là "sanity check" tự nhiên cho ADTC do Finance nhập: ADTC mà cần > 100% ghế thì cảnh báo. Không thấy trong KB/deck.

---

## 5. Khoảng trống thiết kế (chưa tài liệu nào nói)

| # | Khoảng trống | Vì sao quan trọng | Đề xuất |
|---|---|---|---|
| 5.1 | **Hợp đồng thuê 5–6 năm, mô hình 10 năm** | IRR 10Y và payback "có reinvestment" phụ thuộc 4–5 năm *sau* khi hợp đồng hết. Không có giả định gia hạn, rent reset, hay relocation. | Thêm vào hồ sơ location: `lease_term`, `renewal_option`, `rent_reset_at_renewal`. Engine: sau lease end áp rent reset (mặc định +X% master data) **hoặc** cắt horizon = lease term và hiển thị IRR 5Y/IRR lease-term cạnh IRR 10Y. |
| 5.2 | **NPV chưa có định nghĩa** | KB coi như KPI có sẵn. | Định nghĩa: NPV của FCF unlevered, horizon = 10 năm hay lease term, có terminal value không (đề xuất không). Discount rate theo BU hay Group. |
| 5.3 | **Thuế**: không chuyển lỗ, CIT 20% cố định | Ảnh hưởng store lỗ Y0. | Tham số `cit_rate`, `loss_carry_forward_years` (VN: 5 năm). |
| 5.4 | **Deposit / key money lifecycle** | Deposit hoàn khi kết thúc; key money không hoàn. | Cờ `refundable` theo cấu phần; dòng hoàn tại lease end. |
| 5.5 | **Mapping format cũ → 4 format** | File dùng "Core" nhưng mẫu COO không có chiều format thật (chỉ text `D25`). | Bảng mapping + rule: BoE không có format hợp lệ → không tính được. |
| 5.6 | **Channel mix & commission** | ADTC chia IS / 3P / 1P (`K10:K12`), CMS = Rev × %3P × 11%. Không có trong KB. | Tham số BU × format; cần cho actual-vs-BoE sau này (delivery mix thay đổi nhanh). |
| 5.7 | **Working capital** | Có trong BS (AR 1 ngày, AP 27, tồn kho 10) nhưng FCF cho IRR bỏ qua. | Quyết định: bỏ hẳn (đơn giản) hay đưa vào FCF (chuẩn). Đề xuất bỏ ở Phase 0, giữ chỗ tham số. |
| 5.8 | **Số liệu "Updated_BE" vs "BE"** | Diff Kon Tum vs Dĩ An cho thấy lệch cả công thức. | Script diff công thức tự động cho 16 file, xuất bảng ô khác nhau → đây là bước 1 của golden test, trước khi tin bất kỳ file nào làm "chuẩn". |
| 5.9 | **Đơn vị và tỷ giá** | Investment/m² nhập USD (343 $/m² × 25.000), chi phí khác VND. | Lưu VND là đơn vị gốc; USD chỉ hiển thị; tỷ giá có version. |
| 5.10 | **Golden test bằng LibreOffice** (deck slide 13) | File dùng dynamic array (`TRANSPOSE`, `ANCHORARRAY`) và `TODAY()` (`D12`). LibreOffice có thể không tính giống Excel. | Golden test trên **giá trị đã cache** của file + tái hiện có chủ đích các lỗi 3.1–3.3 bằng cờ `legacy_mode`; sau đó mới tắt cờ để ra số chuẩn. Không phụ thuộc LibreOffice. |
| 5.11 | **GB Simulation = master data 1B đã có sẵn** | GB có bảng ADS/ADTC/AC, % chi phí, T&E, pre-opening, daypart mix theo 4 format từ actual 2024/2025 (163 Core, 25 Flagship…). COO không có. | Seed L1 cho GB từ sheet này (sau khi sửa ô gãy 3.12); COO phải lấy từ Databricks/Excel Finance → Phase 2 phụ thuộc dữ liệu COO nhiều hơn deck đang giả định. |

---

## 6. Đề xuất cụ thể

### 6.1 Cập nhật Knowledge Base ngay
- Chuyển các dòng [GĐ]/[MỞ] ở mục 6 thành **[FILE]** (đã đối chiếu) với ô tham chiếu, theo bảng mục 2 ở trên.
- Sửa: ghế = FOH net / 1,5 m² (không phải 1,2–1,5 ghế/m²); comparable lọc format trước diện tích; IRR trong file hiện là post-interest.
- Thêm dimension: Location type, Trade area type, Entity. Ghi rõ "Region: chưa tồn tại, cần định nghĩa".
- Thêm mục "Lỗi template đã xác nhận" (bảng mục 3) làm backlog cho engine.

### 6.2 Đặc tả engine Phase 0 – các mặc định đề xuất

| Khối | Mặc định đề xuất | Nguồn |
|---|---|---|
| Thời gian | Y0 = 6 tháng ramp-up (tháng 1–6), Y1 = tháng 7–18, …, Y10. IRR tính theo **tháng** rồi quy năm (hoặc Y0 = 0,5 kỳ) để hết lỗi 3.6. | file + sửa |
| Doanh thu | ADS = ADTC × AC; ramp-up trên ADTC theo BU×format; growth ADTC theo năm; AC growth = tham số (mặc định 0 như file, Finance xác nhận). | file |
| Chi phí | COGS/MKT/CMS/OCE = % doanh thu; LC = fixed-from-target + wage inflation, có bottom-up để đối chiếu; Rent theo lịch hợp đồng; Royalty = % (GB). | file |
| Pre-opening | Tách opex (vào P&L Y0, **không** vào CFI) và capex (vào CFI, khấu hao 3 năm). Sửa lỗi 3.1. | sửa |
| CAPEX | Construction = báo giá nhập + chuẩn/m² để cảnh báo; Thiết bị = bộ chuẩn format (sửa được, ghi lý do); Franchise fee (GB); Key money; Deposit (refundable). | file + 2.14 |
| Khấu hao | Theo bảng chính sách: constr 5y, equip 50/50 5y/3y, preop capex 3y, franchise 5y, RRR 5y. | file |
| RRR | Danh sách **sự kiện** {năm, loại, % hoặc số tiền, số ngày đóng cửa, đời khấu hao}. Lịch % của file là preset mặc định. Hiệu ứng đóng cửa chỉ khi `closure_days > 0`. Sửa lỗi 3.2. | sửa |
| Thuế | CIT 20%, chuyển lỗ 5 năm (tham số). | sửa |
| Dòng tiền | FCF unlevered = EBITDA − CIT(unlevered) − CAPEX − RRR (± WC nếu bật). IRR/NPV/Payback trên FCF này. Lãi vay/CFF chỉ ở báo cáo tham khảo "levered". Sửa lỗi 3.3. | sửa |
| KPI quyết định | `y1_ebitda_over_investment` (tên hiển thị: ROIC 1Y), `y1_sales_over_investment` (Sales/CAPEX); mẫu số = bộ cờ **theo master data BU**, mặc định mục 4.4. | file + 4.4 |
| Legacy mode | Cờ tái hiện 3.1/3.2/3.3/3.6/3.9 để golden test ra đúng số file. | 5.10 |

### 6.3 Master data – thứ tự ưu tiên chốt
1. **Ngưỡng ROIC & Sales/CAPEX theo BU × format** (có vùng hay không — mục 4.3). Nguồn gợi ý: `Direction!V29:Y33` "Box Economics Approved 2025" (ROIC 0,50–0,54; S/C 1,56–2,03) và "Target NSO2026" (ROIC 0,35–0,43; S/C 1,56–2,57). Hai bảng này chênh nhau lớn → Finance phải nói bảng nào là chuẩn.
2. Bộ cờ mẫu số theo BU.
3. Ramp-up curve theo BU × format (xác nhận COO curve).
4. % chi phí chuẩn theo BU × format (GB có sẵn; COO cần).
5. Thiết bị chuẩn, capex/m², rent/m² theo format (× vùng nếu định nghĩa được).
6. Royalty, franchise fee (GB); CIT; tỷ giá; discount rate.
7. Chính sách khấu hao; lịch RRR mặc định.

### 6.4 Data model – điều chỉnh so với KB mục 3
- `Location`: thêm `location_type`, `trade_area_type`, `lease_term_months`, `renewal_option`, `handover_date`, `opening_date`, `rent_schedule[]` (thay vì text "Rent review").
- `BoEVersion`: thêm `first_pass_kpi` (số trước khi Finance chỉnh), `comparable_set_id`, `master_data_version_id`, `engine_version`, `denominator_flagset_id` (đọc từ master data, không nhập).
- `CapexItem`: `category` (construction/equipment/preopening_capex/preopening_opex/franchise/keymoney/deposit), `amount`, `refundable`, `in_roic_denominator` (dẫn xuất từ flagset), `depreciation_years`.
- `RrrEvent`: `year`, `type`, `amount_or_pct`, `closure_days`, `depreciation_years`, `triggered_by`.
- `ThresholdSet`: `bu`, `format`, `region?`, `roic_min`, `sales_capex_min`, `effective_from`, `decided_by`.
- `StoreIdMapping`: thêm cột `source_system`, `valid_from` — vì mã NetSuite (30897) và mã ERP (HCSBDU0037) đều xuất hiện trong tên file hiện tại.

### 6.5 Lộ trình – điều chỉnh
- **Phase 0 thêm bước 0a**: script diff công thức 16 file → bảng "ô nào khác" → mới chọn file chuẩn cho từng BU. Không có bước này, golden test sẽ khớp với một file ngẫu nhiên.
- **Phase 0 xuất "Bảng đối chiếu số cũ/mới" cho Finance** (IRR legacy vs unlevered, ROIC theo 3 mẫu số) cho 16 store — đây là deliverable thuyết phục Finance, quan trọng hơn UI.
- **Phase 2 (comparable)**: tách COO và GB. GB có thể chạy ngay với sheet Simulation; COO chờ Databricks/Excel Finance. Không nên gộp làm một mốc.
- **Phase 3**: thêm "trần CAPEX từ BoE → SLM" (mục 4.1) để rule chỉnh vốn có răng.

---

## 7. Câu hỏi cần Finance/Expansion chốt (thay cho mục 11 của KB)

Ưu tiên theo mức chặn thiết kế. ★ = chặn Phase 0.

| # | Câu hỏi | Ai | Gợi ý trả lời từ file |
|---|---|---|---|
| ★1 | Bộ ngưỡng chính thức theo BU × format là bảng nào: "Approved 2025", "Target NSO2026", hay số đang dán trong từng file? Có cần chiều vùng không? | Finance | Mục 3.4, 6.3 |
| ★2 | Xác nhận 3 lỗi 3.1/3.2/3.3 là lỗi (không phải chủ ý). Chấp nhận IRR mới cao hơn ~11 điểm ở store mẫu? | Finance | Mục 3 |
| ★3 | Mẫu số ROIC/Sales-CAPEX chuẩn: chọn 1 trong 3 (constr+equip / +preop capex+franchise+keymoney / all-in). | Finance | Mục 4.4 |
| ★4 | Ramp-up COO `0,8/0,9/0,95/1,0/0,9/0,95` là chủ ý hay lỗi? GB `0,5…0,95` giữ? | Finance COO/GB | 2.2, 4.7 |
| ★5 | RRR: giữ lịch % mặc định (0,3%/3%/20%/70%/15%) hay chuyển sang sự kiện do Finance/Ops nhập? Số ngày đóng cửa thực tế cho refresh/renovation? | Finance + Ops | 2.11 |
| 6 | Rule "chỉnh để đạt": chấp nhận rào chắn P25–P75 + trần CAPEX + lưu first-pass KPI? | Hùng + Finance | 4.1 |
| 7 | AC phẳng 10 năm, ADTC +7%/5%: chủ ý? | Finance | 2.3 |
| 8 | Sau khi hết hạn thuê (5–6 năm): giả định gia hạn/rent reset, hay cắt horizon? | Finance + Expansion | 5.1 |
| 9 | NPV: định nghĩa dòng tiền, horizon, discount rate (Group). | Group Finance | 5.2 |
| 10 | LC: giữ fixed-from-target hay bắt buộc bottom-up headcount? | Finance + Ops | 2.4 |
| 11 | Thuế: chuyển lỗ? CIT theo entity? | Finance | 5.3 |
| 12 | Định nghĩa "vùng" (nếu cần) và ai sở hữu danh sách. | Expansion | 4.3 |
| 13 | COO có master data % chi phí / ADS theo format ở đâu (tương đương sheet Simulation của GB)? | Finance COO | 5.11 |
| 14 | `Updated_BE` khác `BE`: xin cả hai bộ file để chạy diff tự động. | Finance | 5.8 |
| 15 | Entity "HC" trong template COO là gì so với BU COO? Có BU/entity thứ ba không? | Hùng | 2.16 |

---

## Phụ lục A – Tham số thực trích từ file (để seed master data)

**COO Core (Kon Tum v4 / Dĩ An v2)**

| Tham số | Kon Tum | Dĩ An | Ô |
|---|---|---|---|
| ADTC / AC (nghìn) | 203 / 78 | 160 / 78 | D36, D37 |
| % COGS | 28,80% | 26,84% | D53 |
| % LC | 14,18% | 14,00% | D56 |
| % MKT | 3,39% | 3,27% | D57 |
| % CMS (= %3P × 11%) | 0,55% | 0% (K11 = 0) | D58 |
| % OCE | 9,26% | 8,28% | D59 |
| Channel mix IS/3P/1P | 85/5/10 | 100/0/0 | K10:K12 |
| Rent/tháng (tr) | 30 | 60 | L16 |
| Leasing / Investment m² | 300 / 200 | formula | H16, H17 |
| BOH % | 28% | 25% | J18 |
| Construction $/m² | 343 | 450 | G43 |
| Thiết bị (tr) | 1.203 | 1.220 | H44 |
| Pre-opening (tr) | 333 (opex 213 / capex 120) | — | H57, J57, K57 |
| Deposit | 3 tháng rent + 50 điện nước = 140 | 180 tr | G74:G76, D23 |
| Vay / lãi / kỳ hạn | 70% / 10% / 5 năm | như nhau | W200, W197, W201 |
| CIT | 20% | 20% | W196 |
| Ngưỡng IRR10/ROIC/S-C | 21,8% / 42,8% / 2,07 | 24,4% / 50,7% / 2,14 | E3, E5, E8 |

**GB theo format (sheet Simulation / Summary2 / Direction)**

| Tham số | Flagship | Core | Small | Kiosk | Ô |
|---|---|---|---|---|---|
| ADS (tr) / ADTC / AC | 38,3 / 412 / 93 | 17,0 / 215 / 79 | 10,6 / 125 / 85 | 7,2 / 84 / 86 | Sim H20:K22 |
| Leasing m² | 250 | 165 | 110 | 40 | Sim H66:K66 |
| Investment $/m² | 530 | 432 | 540 | 540 | Sim H71:K71 |
| Lease $/m² (expected / actual) | 55 / 42 | 35 / 26 | 27 / 23 | — | Dir AF4:AH5 |
| % COGS / LC / MKT / CMS / OCE | 27,6 / 12,8 / 2,0 / 0,7 / 7,6 | 29,8 / 14,2 / 3,4 / 2,4 / 9,3 | 29,6 / 13,4 / 3,2 / 1,3 / 8,1 | 27,4 / 13,7 / 1,2 / 0,4 / 7,2 | Sim H40:K46 |
| T&E: Equip / Tools / Setup / IT (tr) | 1.581 / 156 / 183 / 281 | 740 / 45 / 114 / 245 | 568 / 41 / 94 / 209 | 568 / 41 / 94 / 209 | Sim H137:K140 |
| Pre-opening (tr) | 562 | 328 | 289 | 208 | Sim H86:K86 |
| Royalty | 5% | | | | Core D62 |
| Franchise & opening fee (tr) | 456,5 | | | | Core L45 |
| Ngưỡng ROIC / S-C "Approved 2025" | 0,50 / 1,56 | 0,52 / 1,80 | 0,54 / 1,90 | 0,54 / 2,03 | Dir V31:Y32 |
| Ngưỡng ROIC / S-C "Target NSO2026" | 0,41 / 2,57 | 0,35 / 1,56 | 0,40 / 1,62 | 0,43 / 2,02 | Dir V8:Y9 |
| Ngưỡng thực dùng trong Core sheet | 35% / 1,5 (hard-code) | | | | Core E5, E8 |

**Lịch mặc định 10 năm (cả hai BU)**: growth ADTC 7%×6 năm rồi 5%; wage +10% năm chẵn; RRR 0,3% / 3,3% / 3,3% / 3,3% / 20,3% / 3,3% / 70,3% / 3,3% / 15,3% / 3,3%.
