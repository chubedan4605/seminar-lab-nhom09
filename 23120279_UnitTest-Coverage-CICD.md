# Báo cáo lab Nhóm 09

**Họ tên:** Nguyễn Lê Quang Kan · **MSSV:** 23120279 · **JDK đã dùng** (kết quả `java -version`): `openjdk version "17.0.15" 2025-04-15 LTS`

## Bài 1: Giá trị biên

| Ngày trễ | Hạng | Số tiền bạn điền | Cách tính |
|---:|---|---:|---|
| 24 | NORMAL | 48000 | 24 × 2.000 = 48.000đ (dưới mức trần 50.000đ) |
| 25 | NORMAL | 50000 | 25 × 2.000 = 50.000đ (đúng tại mức trần 50.000đ) |
| 26 | NORMAL | 50000 | 26 × 2.000 = 52.000đ → áp trần tối đa 50.000đ |
| 2 | VIP | 0 | Nằm trong 2 ngày ân hạn đầu (≤ 2 ngày → 0đ) |
| 3 | VIP | 1000 | (3 − 2) × 1.000 = 1.000đ (ngày đầu tiên tính phí sau ân hạn) |
| 51 | VIP | 49000 | (51 − 2) × 1.000 = 49.000đ (dưới mức trần 50.000đ) |
| 52 | VIP | 50000 | (52 − 2) × 1.000 = 50.000đ (đúng tại mức trần 50.000đ) |
| 53 | VIP | 50000 | (53 − 2) × 1.000 = 51.000đ → áp trần tối đa 50.000đ |

**Vì sao chọn các ngày này mà không chọn ngày 10?**
Theo kỹ thuật **Phân tích giá trị biên (Boundary Value Analysis - BVA)**, lỗi lập trình (như lỗi off-by-one dùng `<` thay vì `<=`, quên trừ ngày ân hạn, hoặc tính sai ngưỡng trần) thường tập trung tại các điểm ranh giới giữa các phân vùng tương đương:
- Với hạng **NORMAL**: ngưỡng trần 50.000đ rơi vào đúng ngày 25 (`25 × 2.000 = 50.000`), nên chọn cụm 3 điểm biên `24, 25, 26` (dưới biên, tại biên, trên biên).
- Với hạng **VIP**: có 2 điểm ranh giới là kết thúc thời gian ân hạn tại ngày `2` (chọn `2, 3`) và chạm ngưỡng trần 50.000đ tại ngày `52` (`(52 − 2) × 1.000 = 50.000`, chọn `51, 52, 53`).
- Ngày `10` nằm sâu bên trong một phân vùng tương đương bình thường, không sát bất kỳ ranh giới nào nên ít khả năng phát hiện lỗi biên hơn nhiều so với các giá trị biên trên.

## Bài 2a: Coverage

| | Branch coverage của `LoanService` |
|---|---|
| Trước khi viết test | 66% (0.66 — 4/6 nhánh) |
| Sau khi viết 2 test | 100% (1.00 — 6/6 nhánh) |

**Test `daDatHanMuc` của bạn dùng `countActiveByMemberId` trả về giá trị nào? Vì sao chọn giá trị đó?**
- Ở bước 2a ban đầu, dùng `4L` để đi vào nhánh `true` của câu điều kiện hiện có trong code và đạt 100% branch coverage.
- Sau khi chạy PIT ở bước 2b và đối chiếu lại đặc tả (*"Mỗi thành viên được mượn tối đa 3 cuốn: đang mượn 3 cuốn thì không được mượn thêm"*), đã sửa lại thành giá trị biên **`3L`** vì đây là điểm biên chính xác bắt đầu bị từ chối (`LIMIT_REACHED`), kết hợp với test `muonThanhCongKhiDangMuonHaiCuon` (`2L`) tạo thành cặp kiểm thử biên hoàn chỉnh (`2` cho phép, `3` từ chối).

## Bài 2b: Mutation testing

**Lần chạy PIT đầu tiên:** có bao nhiêu mutant sống sót (SURVIVED)? Nằm ở dòng nào, là loại mutant gì?
- Có **1 mutant sống sót (SURVIVED)** trên tổng số **9 mutant** (Killed 8/9 — 89%).
- Nằm ở **dòng 28** của `src/main/java/vn/edu/seminar/loan/LoanService.java` (`if (loans.countActiveByMemberId(memberId) > 3)`).
- Loại mutant: **`ConditionalsBoundaryMutator`** (`changed conditional boundary` — đổi toán tử `>` thành `>=`).

**Lỗi cài sẵn nằm ở đâu? Bạn phát hiện ra nó bằng cách nào?**
- Lỗi nằm ở `src/main/java/vn/edu/seminar/loan/LoanService.java`, **dòng 28**: code đang kiểm tra `loans.countActiveByMemberId(memberId) > 3` thay vì `>= 3`. Điều này cho phép thành viên đang mượn 3 cuốn vẫn có thể mượn thêm cuốn thứ 4, vi phạm đặc tả nghiệp vụ (*tối đa 3 cuốn*).
- Phát hiện bằng cách: PIT báo mutant `ConditionalsBoundaryMutator` tại dòng 28 sống sót (vì test chỉ kiểm tra `2L` và `4L`, bỏ sót biên `3L`). Khi đối chiếu đặc tả và cập nhật test `daDatHanMuc` sang kiểm tra giá trị biên `3L`, test chạy đỏ vì `3 > 3` là `false` → khẳng định lỗi nằm ở code sản phẩm.

**Bạn đã sửa test và code như thế nào?**
- **Test (`LoanServiceTest.java`):** Sửa `daDatHanMuc()` để mock `loans.countActiveByMemberId(1)` trả về **`3L`** (đúng giá trị biên theo đặc tả), kiểm tra ném `LoanException` với mã `LIMIT_REACHED`, đồng thời kiểm tra không gọi `loans.save` và `books.save`.
- **Code (`LoanService.java`, dòng 28):** Sửa `if (loans.countActiveByMemberId(memberId) > 3)` thành `if (loans.countActiveByMemberId(memberId) >= 3)`.

**Kết quả PIT sau khi sửa** (Killed bao nhiêu / tổng bao nhiêu):
- **9 / 9 KILLED (100%)** — 0 SURVIVED, 0 NO_COVERAGE, Test strength: 100%.

## Câu hỏi tổng kết
**Coverage 100% có nghĩa là code không còn lỗi không? Hãy giải thích dựa trên những gì bạn thấy trong lab.**
**Không.** Đạt 100% coverage (kể cả line coverage lẫn branch coverage) chỉ chứng minh rằng mọi dòng lệnh và mọi nhánh điều kiện đều đã được *thực thi qua* trong quá trình chạy test, chứ không chứng minh được code đúng với đặc tả nghiệp vụ hay bộ test đủ mạnh để bắt lỗi. Trong bài lab này, ở cuối Bài 2a, `LoanService` đã đạt 100% branch coverage và build xanh hoàn toàn, nhưng lỗi off-by-one (`> 3` thay vì `>= 3`) vẫn tồn tại vì bộ test chỉ kiểm tra giá trị `2` và `4` mà bỏ sót giá trị biên `3`. Nhờ **Mutation Testing (PIT)** tạo đột biến biên, lỗ hổng của bộ test mới bị lộ ra, từ đó giúp tìm và sửa đúng lỗi trong code.
