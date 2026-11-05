# BoE Studio – Tổng hợp kiến thức & yêu cầu (để phân tích / thiết kế)

> Phiên bản: 05-Nov-2026 | Người sở hữu: Hùng Trần | Mục đích: làm đầu vào cho phân tích kiến trúc/thiết kế chi tiết.
>
> **Quy ước nhãn độ tin cậy**
> - **[CHỐT]** = Hùng đã nói/quyết định trực tiếp trong các buổi trao đổi.
> - **[GĐ]** = Giả định/suy luận từ kiến thức chung về BoE ngành F&B, **chưa kiểm chứng với file Excel thật**.
> - **[MỞ]** = Còn chờ xác nhận hoặc chưa có thông tin.
>
> **Hạn chế quan trọng:** 16 file Excel (BE/, Updated_BE/, file GB_40_Vuong_Thua_Vu) đã đính kèm nhưng nội dung đến phía phân tích **bị rỗng**. Vì vậy toàn bộ công thức, cấu trúc sheet, bug của template **chưa được đọc**. Mọi chi tiết tính toán ở mục 6 phải được đối chiếu lại với file gốc trước khi dùng làm đặc tả.

---

## 1. Bối cảnh & vấn đề

### 1.1 BoE là gì trong ngữ cảnh này
- BoE là bản phân tích hiệu quả đầu tư **cho từng cửa hàng (store/location)**, hiện nằm trong **một file Excel cho mỗi store**. **[CHỐT]**
- Dùng để Finance/Management xem xét mở store mới (và cả các store thuộc diện RRR – Renovation, Refresh, Relocation) dựa trên chỉ số hiệu quả. **[CHỐT]**
- Hiện có **2 BU** với template BoE riêng: **GB (Great Bear)** và **COO**. **[CHỐT]**
- Tên file mẫu cho thấy ví dụ store: Stella Restaurant, 75 Phan Đình Phùng Hà Tĩnh, Bưu điện Quy Nhơn, Mizuki Park Bình Chánh, VNPT Buôn Ma Thuột, 99 Nguyễn Thị Ngọc Oanh Ninh Hòa, Quảng trường Phước Hải, 38 Phùng Hưng, Bưu điện Chí Linh, 40 Vương Thừa Vũ (GB). Có thư mục `BE` (bản gốc) và `Updated_BE` (bản cập nhật, 6 store). **[Quan sát từ tên file]**

### 1.2 Pain points hiện tại
- Expansion nháp BoE → Finance **nhập lại và tính lại** = làm hai lần. Hùng muốn ép về **Finance-only input**. **[CHỐT]**
- Một file Excel / store, tốn nhiều công. **[CHỐT]**
- Finance chọn store tương đồng (comparable) **thủ công**: lọc vùng, rồi lọc thành phố, sau đó dự báo ADS/ADTC/AC. **[CHỐT]**
- Template Excel có **bug** (engine mới phải sửa). **[CHỐT]**
- Mã store của Finance **không nhất quán**: store cũ dùng mã NetSuite, store mới dùng mã ERP; Finance lọc theo vùng và địa chỉ. **[CHỐT]**
- Ngưỡng duyệt, %LC (trung bình ~12%) và các tham số tương tự cần định nghĩa theo **format** và **vùng**. **[CHỐT]**

### 1.3 Mục tiêu
1. Thay Excel bằng **BoE Studio** dùng chung cho GB và COO. **[CHỐT, đang cân nhắc "one shared"]**
2. Giảm công Finance, loại bỏ nhập hai lần.
3. Chuẩn hoá cách tính ROIC và Sales/CAPEX cho mọi store.
4. Gợi ý comparable store bằng luật thay vì lọc tay.
5. (Sau này) Đối chiếu **actual vs BoE** theo store; Hùng chưa chắc khả thi ở quy mô đầy đủ. **[CHỐT, mức mong muốn]**

---

## 2. Quyết định đã chốt (tổng hợp theo chủ đề)

### 2.1 Hình thức & phạm vi
- Xây **web app độc lập, chỉ desktop, không mobile**, để Hùng review, sau đó có thể merge vào nền tảng khác. **[CHỐT]**
- BoE Studio là **service độc lập, không phải module trong SLM**; SLM tích hợp vào BoE Studio. **[CHỐT]**
- **Không có approval workflow** trong hệ thống; việc duyệt diễn ra bên ngoài. **[CHỐT]**
- Cách tiếp cận theo pha: **master data + engine trước, UI sau**. **[CHỐT]**

### 2.2 Phân vai & quyền
- **Finance chỉ làm việc với số.** **[CHỐT]**
- **Finance ước tính ADTC/AC (và do đó ADS).** **[CHỐT]**
- **Expansion sở hữu master data L0/L1** (dimension và standard) trong hệ thống. Điều này **thay thế** quyết định trước đó "Finance owns BoE master data". **[CHỐT, mới nhất 3-Nov]**
- **Bộ thiết bị chuẩn theo format** do **team khác** xây dựng. **[CHỐT]**
- Expansion có số riêng cho bản Draft, sẽ xin tài khoản **Databricks** để lấy số. **[CHỐT]**

### 2.3 Engine
- Engine **sửa bug** của template Excel. **[CHỐT]**
- **Ramp-up là một đường cong duy nhất**, không tính seasonality. **[CHỐT]**
- **ROIC và Sales/CAPEX tính theo một cách chuẩn hoá** cho mọi store. **[CHỐT]**
- **Mẫu số ROIC cấu hình được** (tick các thành phần CAPEX được tính). Hùng giao cho phía phân tích **tự chọn giả định mặc định dựa trên công thức trong file** (chưa làm được vì file rỗng). **[CHỐT]**
- Hạng mục **phá dỡ (demolish), cầu thang...** thuộc **CAPEX xây dựng**. **[CHỐT]**
- Chi phí **thiết bị chuẩn hoá theo format**, đặt dưới dạng **khoảng tham chiếu theo vùng**. **[CHỐT]**
- **RRR = Renovation, Refresh, Relocation.** Khi một store được chọn RRR thì chi phí RRR **phải được lên kế hoạch trong mô hình**. Giả định hiện tại: chi phí RRR **chảy vào CFI**. Kế hoạch RRR do **Finance hoặc Operation** kích hoạt. **[CHỐT]**
- **IRR thường là trước lãi vay (pre-interest).** **[CHỐT]**
- **Discount rate cho NPV do Group (Finance) quyết định.** **[CHỐT]** (giá trị cụ thể **[MỞ]**)
- **LC/OCE là chi phí bán cố định (semi-fixed)**, Hùng chưa biết nên mô hình hoá thế nào. **[MỞ]**
- GB: **royalty/franchise fee trả cho Highlands** (phí nội bộ). **[CHỐT]**

### 2.4 Format & chuẩn hoá
- Format chuẩn đề xuất: **Flagship, Core, Small format, Kiosk**. **[CHỐT]**
- Có thêm **"extension"** bổ sung sau cho các tiêu chí phụ. **[CHỐT]**

### 2.5 Quy tắc duyệt (đã chốt 3-Nov)
- Người duyệt xem **ROIC và Sales/CAPEX trước**.
- Finance được điều chỉnh **ADS (ADTC/AC), diện tích đầu tư hoặc số tiền đầu tư** để **ít nhất một** chỉ số vượt ngưỡng.
- Bảng quyết định:

| ROIC | Sales/CAPEX | Kết quả |
|---|---|---|
| ≥ ngưỡng | ≥ ngưỡng | Đạt |
| < ngưỡng | ≥ ngưỡng | Cần review/duyệt riêng |
| ≥ ngưỡng | < ngưỡng | Cần review/duyệt riêng |
| < ngưỡng | < ngưỡng | **Không duyệt. Chặn cứng, không ngoại lệ** |

- **IRR chỉ để tham khảo, không nằm trong quy tắc.** **[CHỐT]**
- Ngưỡng định nghĩa theo **format × vùng**. **[CHỐT]**

### 2.6 Versioning (chốt 3-Nov)
- Bắt đầu với **2 loại version**:
  - **Draft**: do Expansion lập, để chốt địa điểm.
  - **Final**: do Finance lập tại **kickoff meeting ngày 1**, sau đó **khoá**.
- Thay đổi sau đó tăng version: **Revise v1, v2...**
- Một version có **hai thuộc tính tách biệt**: **type** (Draft/Final/Revise) và **state** (editing/locked).
- **Khi lock, hệ thống ghim phiên bản master data đã dùng.**
- Giá trị mặc định ADS/ADTC/AC của Draft lấy từ **gợi ý comparable store**.

### 2.7 Comparable store
- Hiện Finance chọn thủ công trong Excel; dữ liệu Excel của Finance **gồm cả store đã đóng**. **[CHỐT]**
- Hệ thống sẽ **gợi ý comparable theo luật**, **bắt đầu bằng diện tích** làm proxy cho mức doanh thu kỳ vọng. **[CHỐT]**
- Diện tích so sánh = **diện tích sử dụng** (ví dụ thuê 80 m² nhưng 2 tầng = 160 m²). **[CHỐT]**
- Số ghế suy ra ~**1,2–1,5 ghế/m²**. **[CHỐT]**
- Sẽ xin dữ liệu lịch sử để nạp sau; thiết kế sẵn khả năng import lịch sử dù chưa có quyền truy cập. **[CHỐT]**

### 2.8 Định danh store & location
- Muốn **một store_id dùng chung** cho BoE, SDC Tracker, TS Ranking. **[CHỐT]**
- Về lâu dài store_id đến từ **nền tảng master data riêng (chưa xây)**. **Tạm thời** mỗi hệ thống giữ id riêng; **TS Ranking là nơi lập kế hoạch/sinh location nên là nguồn id tạm**. **[CHỐT]**
- **Location (trong TS Ranking)** = **một mặt bằng cụ thể có địa chỉ** (dùng để xây store sau này). **Một trade area có thể có nhiều location** (nhiều store sẽ mở). **[CHỐT]**

---

## 3. Mô hình miền (Domain model)

### 3.1 Thực thể chính

```
BU (GB | COO)
  └── Format (Flagship | Core | Small | Kiosk) [+ Extension]
Region / City
MasterDataVersion (L0/L1)         <- do Expansion sở hữu, có version + hiệu lực
  ├── Dimensions (BU, Format, Region, City, Extension...)
  ├── Standards  (thiết bị chuẩn theo format, khoảng giá theo vùng,
  │               %LC, tỷ lệ chi phí, ngưỡng ROIC & Sales/CAPEX, discount rate...)
Location (từ TS Ranking)          <- 1 mặt bằng có địa chỉ; nhiều location / trade area
Store (location đã triển khai)    <- store_id chung (tạm: TS Ranking cấp)
BoE
  └── BoEVersion (type: Draft|Final|Revise-n ; state: editing|locked ;
                  pinned_master_data_version)
        ├── Inputs: ADTC, AC, ADS, diện tích đầu tư, diện tích sử dụng, ghế,
        │           CAPEX (xây dựng, thiết bị, ...), RRR plan, tham số override
        ├── Engine outputs: P&L theo kỳ, dòng tiền, ROIC, Sales/CAPEX,
        │                   IRR, NPV, Payback
        └── Decision outcome: Đạt | Cần review riêng | Chặn cứng
ComparableSet (gợi ý theo luật, gồm store đang mở và đã đóng)
HistoricalStoreData (import sau: ADS/ADTC/AC, diện tích, ghế, vùng, ngày mở/đóng)
StoreIdMapping (NetSuite code | ERP code | TS Ranking id | SDC id | BoE id)
```

### 3.2 Thuộc tính chính cần có [GĐ về chi tiết]
- **Location/Store:** store_id, tên, địa chỉ, vùng, thành phố, BU, format, diện tích thuê, số tầng, diện tích sử dụng, số ghế, ngày dự kiến khai trương, trạng thái.
- **BoE Version:** id, boe_id, type, state, số revise, người tạo/vai trò, thời điểm lock, master_data_version ghim, ghi chú.
- **Master data standard:** theo (BU, format, vùng): %LC, các % chi phí, ngưỡng ROIC, ngưỡng Sales/CAPEX, khoảng giá thiết bị (min/max/chuẩn), discount rate.

---

## 4. Ma trận vai trò [CHỐT phần chính, GĐ phần chi tiết]

| Vai trò | Quyền / trách nhiệm |
|---|---|
| **Expansion** | Sở hữu master data L0/L1 (dimension + standard). Lập Draft. Xin Databricks để lấy số Draft. |
| **Finance** | Chỉ làm việc với số. Ước tính ADTC/AC (ADS). Lập **Final** tại kickoff ngày 1 rồi khoá. Điều chỉnh ADS/diện tích/số tiền đầu tư để vượt ngưỡng. Kích hoạt kế hoạch RRR (cùng Operation). |
| **Operation** | Có thể kích hoạt RRR plan. |
| **Team thiết bị (team khác)** | Xây bộ thiết bị chuẩn theo format. |
| **Group (Finance)** | Quyết định discount rate NPV. |
| **Approver (ngoài hệ thống)** | Xem ROIC, Sales/CAPEX; duyệt ngoài hệ thống. |

---

## 5. Vòng đời BoE (luồng nghiệp vụ)

1. **Location được sinh/quản lý ở TS Ranking** (mặt bằng cụ thể) → cấp id tạm.
2. **Expansion lập BoE Draft** để chốt vị trí. Giá trị mặc định ADS/ADTC/AC lấy từ **gợi ý comparable** (hoặc số từ Databricks của Expansion).
3. Hệ thống **tính** ROIC, Sales/CAPEX (và IRR/NPV/Payback tham khảo) bằng engine chuẩn, áp **ngưỡng theo format × vùng**.
4. **Kickoff ngày 1:** Finance lập **Final** (nhập/điều chỉnh số), chốt rồi **lock** (ghim master data version).
5. Nếu thay đổi sau lock → tạo **Revise vN** (type Revise, state editing).
6. **Đánh giá theo quy tắc duyệt** (mục 2.5). Việc duyệt thực tế diễn ra ngoài hệ thống; hệ thống chỉ hiển thị kết quả và chặn cứng khi cả hai chỉ số dưới ngưỡng.
7. **(Sau này)** So sánh actual vs BoE theo store.

> Lưu ý thiết kế: *Draft* và *Final* là **loại**; *editing/locked* là **trạng thái**. Hai trục này độc lập.

---

## 6. Logic tính toán (engine) – **[GĐ] toàn bộ, cần đối chiếu Excel**

> Mục này mô tả khung phổ biến của BoE F&B. Công thức, tên dòng, thứ tự và các bug cụ thể phải lấy từ template GB/COO thật.

### 6.1 Doanh thu
- ADS (Average Daily Sales) = ADTC (Average Daily Transaction Count) × AC (Average Check). **[CHỐT quan hệ ADS = ADTC × AC theo ngữ cảnh "ADTC/AC"; công thức chính xác GĐ]**
- Doanh thu kỳ = ADS × số ngày hoạt động × **hệ số ramp-up** (một đường cong duy nhất).
- Có thể có tăng trưởng hằng năm (SSSG) **[GĐ, MỞ]**.

### 6.2 Chi phí vận hành
- **COGS** (% doanh thu).
- **LC** (labor cost, trung bình ~12%, bán cố định) **[CHỐT giá trị TB; mô hình MỞ]**.
- **Rent** (thuê, có thể gồm tăng giá theo kỳ, đặt cọc) **[GĐ]**.
- **OCE** (other controllable expenses, bán cố định) **[MỞ cách mô hình]**.
- **Royalty/franchise fee** (GB, trả cho Highlands, % doanh thu) **[CHỐT bản chất, GĐ công thức]**.
- Khấu hao (từ CAPEX theo thời gian khấu hao) **[GĐ]**.

### 6.3 Kết quả P&L
- Store-level (four-wall) EBITDA → EBIT sau khấu hao. IRR **trước lãi vay**. **[CHỐT pre-interest]**

### 6.4 CAPEX (cấu phần) **[GĐ trừ nhận xét đã chốt]**
- Xây dựng (gồm **phá dỡ, cầu thang...** – **CAPEX xây dựng [CHỐT]**).
- Thiết bị (**chuẩn hoá theo format**, khoảng giá theo vùng **[CHỐT]**).
- Các cấu phần khác (deposit, pre-opening, thiết kế, bản quyền... **[GĐ]**).
- **Tick chọn** cấu phần nào vào mẫu số ROIC **[CHỐT]**.

### 6.5 Chỉ số
- **ROIC** = lợi nhuận hoạt động (sau thuế hay trước thuế: **[MỞ]**) / vốn đầu tư (**mẫu số cấu hình được**, mặc định sẽ chọn từ công thức file).
- **Sales/CAPEX** = doanh thu (kỳ chuẩn, ví dụ năm ổn định **[MỞ]**) / tổng CAPEX (cấu phần nào tính **[MỞ]**).
- **IRR** (pre-interest), **NPV** (discount rate do Group quyết), **Payback** – chỉ để tham khảo.

### 6.6 RRR
- Khi store được chọn RRR, chi phí RRR phải được lập kế hoạch trong mô hình; giả định chảy vào **CFI**; do Finance/Operation kích hoạt. **[CHỐT]** Cách mô hình dòng tiền chi tiết (thời điểm, khấu hao, tác động doanh thu trong giai đoạn đóng cửa tạm) **[MỞ]**.

### 6.7 Nguyên tắc engine
- Hàm thuần (input → output), không phụ thuộc UI.
- **Golden test:** chạy cùng input với Excel (sau khi sửa bug) để đối chiếu; ghi lại danh sách bug đã sửa và ảnh hưởng.
- Mọi tham số lấy từ master data có version; lock thì ghim version.

---

## 7. Gợi ý comparable store

- **Dữ liệu nguồn:** file Excel của Finance (có cả store đã đóng); sẽ nạp dữ liệu lịch sử sau; Expansion có nguồn Databricks. **[CHỐT]**
- **Luật v1:** dùng **diện tích sử dụng** làm proxy cho mức doanh thu; lọc **vùng → thành phố** như cách Finance làm tay. **[CHỐT]**
- **Đầu ra:** tập comparable → gợi ý mặc định **ADS/ADTC/AC** cho Draft; Finance có thể điều chỉnh. **[CHỐT]**
- **Mở rộng sau [GĐ]:** thêm trọng số theo format, tuổi store, loại mặt bằng, mật độ dân cư; xử lý outlier; loại store đã đóng/tái cấu trúc khi cần.
- **Câu hỏi mở:** cách xử lý store đã đóng (loại hẳn, hay dùng làm tham chiếu có cảnh báo), cửa sổ thời gian lấy ADS (ví dụ 12 tháng ổn định), cách chuẩn hoá ADS theo thời điểm mở.

---

## 8. Định danh & tích hợp

### 8.1 store_id
- Mục tiêu: **một store_id** xuyên suốt BoE, SDC Tracker, TS Ranking. **[CHỐT]**
- Tạm thời: mỗi hệ thống giữ id riêng; **TS Ranking là nguồn id tạm** vì là nơi lập kế hoạch/sinh location. **[CHỐT]**
- Dài hạn: **nền tảng master data riêng** (chưa xây). **[CHỐT]**
- Cần **bảng mapping**: NetSuite code (store cũ) ↔ ERP code (store mới) ↔ id TS Ranking ↔ id SDC ↔ id BoE. **[GĐ giải pháp, nhu cầu CHỐT]**

### 8.2 Quan hệ Location – Trade area – Store
- 1 trade area : N location; 1 location = 1 mặt bằng có địa chỉ; location → store khi được xây dựng. **[CHỐT]**

### 8.3 Tích hợp dự kiến
| Hệ thống | Vai trò với BoE Studio |
|---|---|
| **TS Ranking** | Sinh location + id tạm |
| **SDC Tracker** | Dùng chung store_id |
| **SLM (Store Lifecycle Management)** | Tích hợp vào BoE Studio (không phải ngược lại) |
| **Databricks** | Nguồn số Draft của Expansion (cần xin account) |
| **Finance Excel / ERP / NetSuite** | Dữ liệu lịch sử, mapping mã store |

---

## 9. Kiến trúc đề xuất [GĐ]

### 9.1 Phân lớp
1. **Master data service:** dimension, tham số, thiết bị chuẩn; version + hiệu lực; Expansion sở hữu.
2. **Calculation engine:** thuần hàm, test độc lập, golden test với Excel.
3. **BoE service:** location/store, version (type + state), lock + ghim master data, audit log.
4. **Comparable service:** luật gợi ý, dữ liệu lịch sử.
5. **Integration layer:** store_id mapping, Databricks, TS Ranking, SDC, SLM.
6. **UI (desktop web):** làm **sau cùng**.

### 9.2 Nguyên tắc
- Standalone service, API-first, để SLM và nền tảng khác tích hợp.
- Không có workflow duyệt, nhưng **có kết quả đánh giá ngưỡng** và **chặn cứng**.
- Audit log đầy đủ cho mọi thay đổi input, lock, revise.
- Dữ liệu lịch sử import-ready dù chưa có quyền truy cập.
- Chuẩn bị chỗ cho **actual-vs-BoE** (lưu BoE ở mức đủ chi tiết để so sánh sau).

### 9.3 Phasing đề xuất
- **Phase 1:** master data (L0/L1) + engine + test đối chiếu Excel (GB và COO).
- **Phase 2:** BoE service (version, lock, ghim master data) + comparable v1 + import lịch sử.
- **Phase 3:** UI desktop + tích hợp TS Ranking/SDC/SLM.
- **Phase 4 (tuỳ chọn):** actual-vs-BoE.

---

## 10. Hợp nhất GB và COO

- Hùng đang cân nhắc **một BoE Studio chung** cho cả hai BU. **[CHỐT hướng, đang cân nhắc]**
- Cần phân tích: phần chung (khung P&L, ROIC, Sales/CAPEX) vs phần khác biệt (ví dụ **royalty của GB**, cấu trúc chi phí, danh mục CAPEX, format, ngưỡng).
- Đề xuất cách tiếp cận **[GĐ]**: một engine chung + **cấu hình theo BU** (cost structure, loại phí, format hợp lệ) thay vì hai engine.
- **Việc cần làm sau khi có file thật:** đối chiếu hai template (GB vs COO) theo từng sheet/dòng, lập bảng khác biệt.

---

## 11. Câu hỏi còn mở (cần xác nhận)

**Nghiệp vụ / tài chính**
1. Discount rate NPV cụ thể (Group/Finance). IRR dùng để duyệt là cấp BU hay Group?
2. Ngưỡng ROIC và Sales/CAPEX cụ thể theo format × vùng.
3. ROIC dùng lợi nhuận nào (EBIT, NOPAT, EBITDA?) và mẫu số mặc định gồm cấu phần CAPEX nào.
4. Sales/CAPEX dùng doanh thu năm nào (năm ổn định, năm 1, trung bình)?
5. Mô hình hoá **LC** và **OCE** bán cố định (bậc thang, công thức theo doanh thu, theo ghế/m²?).
6. Cơ chế dòng tiền RRR: thời điểm chi, khấu hao, ảnh hưởng doanh thu khi cải tạo/di dời, cách tính so với store hiện hữu.
7. Có tăng trưởng/lạm phát chi phí theo năm không? Thời gian dự báo là bao nhiêu năm?

**Dữ liệu / tích hợp**
8. Khi nào có quyền Databricks và truy cập dữ liệu lịch sử của Finance?
9. Chiến lược mapping mã NetSuite ↔ ERP ↔ TS Ranking id; ai sở hữu bảng mapping?
10. Xử lý store đã đóng trong comparable.

**Phạm vi**
11. Actual-vs-BoE: ở mức store hay mức dòng chi phí; thời điểm đưa vào lộ trình.
12. Cơ chế "extension" của format hoạt động thế nào (thêm tiêu chí gì, ảnh hưởng ngưỡng/chi phí ra sao).
13. Bản `Updated_BE` khác bản `BE` ở điểm nào (logic hay số liệu)? Vì sao 6 store được cập nhật?

**Các việc cần làm với file Excel (khi đọc được)**
- Liệt kê sheet, input cell, công thức ROIC/Sales-CAPEX/IRR/NPV/Payback.
- Phát hiện và ghi lại **bug** (tham chiếu sai, hard-code, công thức không nhất quán).
- So sánh GB vs COO; so sánh BE vs Updated_BE (diff).
- Trích bộ tham số thực tế (%LC, tỷ lệ chi phí, ramp-up curve, thời gian khấu hao).

---

## 12. Gợi ý prompt cho bước phân tích tiếp theo

1. *"Dựa trên tài liệu này, hãy phản biện thiết kế: điểm mâu thuẫn, rủi ro, và các quyết định còn thiếu."*
2. *"Đề xuất data model chi tiết (bảng, khoá, quan hệ) cho master data có version, BoE version (type/state), comparable, store-id mapping."*
3. *"Đề xuất đặc tả engine: input/output schema, thứ tự tính, cách cấu hình mẫu số ROIC, test golden với Excel."*
4. *"Đề xuất phương án hợp nhất GB và COO bằng cấu hình thay vì hai engine."*
5. *"Thiết kế lộ trình Phase 1–4 với deliverable, phụ thuộc và rủi ro."*

---

## Phụ lục A – Tóm tắt nguồn thông tin

- **Quyết định ngày 3-Nov-2026:** BoE Studio chung GB/COO; format chuẩn; royalty GB; master data; comparable theo diện tích; store_id chung; versioning Draft/Final/Revise; Databricks; ROIC denominator cấu hình; RRR; quy tắc duyệt; service độc lập; định nghĩa location; Expansion sở hữu L0/L1.
- **Quyết định trước đó:** web app standalone desktop-only; không approval workflow; Finance ước tính ADTC/AC; engine sửa bug; ramp-up một đường cong; chuẩn hoá ROIC & Sales/CAPEX; thiết bị chuẩn hoá theo format; phá dỡ thuộc CAPEX xây dựng.
- **Chưa đọc được:** nội dung 16 file Excel (rỗng khi đính kèm).
