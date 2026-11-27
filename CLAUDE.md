# Quy tắc làm việc với gads-agent

## Nguồn yêu cầu và hiện trạng

- Đọc toàn bộ `BUILD_SPEC.md` trước khi lập kế hoạch hoặc thay đổi mã. Đây là đặc tả nguồn; README chỉ giải thích cách bắt đầu.
- Repo ban đầu chỉ có tài liệu. Kiểm tra tệp thực tế trước khi nói một chức năng đã tồn tại.
- Khi yêu cầu mâu thuẫn hoặc chưa đủ rõ, nêu điểm cụ thể và tác động để chủ dự án chốt; không tự đổi mục tiêu hoặc sửa đặc tả gốc.
- Triển khai theo ba bước tại mục 9. Không tự chuyển sang bước tiếp theo khi điều kiện nghiệm thu bước hiện tại chưa đạt.

## Cách trao đổi

- Người dùng không biết lập trình. Giải thích bằng tiếng Việt, câu ngắn, từ quen thuộc.
- Trước mỗi phần việc, báo kế hoạch ngắn, đầu ra và thao tác cần duyệt; chờ người dùng duyệt rồi mới làm.
- Không yêu cầu người dùng chạy lệnh terminal. Việc buộc con người làm cần hướng dẫn từng bước kèm menu và dấu hiệu thành công.
- Brief thiếu thông tin: dùng mặc định phù hợp trong `knowledge/` nếu đã có, ghi vào `assumptions` và liệt kê câu hỏi cho người dùng. Không tự bịa ngân sách, giá, ưu đãi hoặc kết quả kinh doanh.
- Kết thúc mỗi bước: nêu đã làm gì, đã kiểm chứng gì, chưa kiểm chứng gì và việc tiếp theo.

## Quyền duyệt và ngân sách

- Không áp dụng thay đổi Google Ads khi chưa được duyệt, trừ đúng allowlist Mức 1 đã được người dùng bật cho từng khách.
- Chiến dịch, nhóm quảng cáo và quảng cáo mới luôn PAUSED. Xuất CSV cũng phải giữ trạng thái này.
- Chỉ GO khi ngân sách chính xác, tracking chính xác và nội dung đã duyệt. Không tự bật chiến dịch thay người dùng.
- Tổng ngân sách ngày × 30,4 không vượt Net Media đã duyệt. Mỗi lần đổi ngân sách tối đa 30%; Mức 1 tối đa ±10% và không vượt tổng.
- Mức 1 mặc định tắt. Chỉ ba nhóm trong mục 6.b được phép tự động khi đã bật: phủ định truy vấn thuộc nhóm rõ ràng đã cấu hình, pause ad Disapproved/URL lỗi, điều chỉnh daily budget trong giới hạn.
- Kiểm tra cờ `tracking_suspect`, quy tắc ba ngày đầu sau go-live và guardrail trước mọi nhánh áp dụng, kể cả thay đổi đã duyệt và Mức 1.
- `DRY_RUN=true` tuyệt đối không thay đổi Ads; không để nhánh Mức 1 hoặc đề xuất đã duyệt bỏ qua cờ này. Phân biệt thông báo thử với kết quả áp dụng thật.
- Đề xuất phải đúng trạng thái và được duyệt; đề xuất quá hạn hoặc đã áp dụng không được chạy lại. Mỗi lần áp dụng phải ghi kết quả và nhật ký trước/sau, nguồn, người duyệt và người thực hiện.
- Chỉ xuất CSV cho `EDITOR_CHANGE` đã duyệt qua quy trình `/ap-dung`; không báo đã áp dụng chỉ vì đã xuất tệp.

## Tracking, số liệu và nội dung quảng cáo

- Tracking trước, tối ưu sau. Nghi lỗi tracking thì chặn đề xuất tối ưu khác, chỉ giữ hành động an toàn theo đặc tả.
- Tách Lead Ads, lead thực tế, lead hợp lệ, CPL và CPQL. Xác nhận kỳ dữ liệu, múi giờ và độ trễ trước khi đối soát.
- Tỷ lệ tính từ tổng tử số và mẫu số, không lấy trung bình cộng các tỷ lệ. Mẫu số 0 → `N/A`.
- Báo cáo tách Search và Demand Gen; không dùng chung một ngưỡng CTR.
- Mỗi nhận định phải có số liệu; không kết luận thành công chỉ vì CPC thấp.
- Chỉ dùng giá/ưu đãi nằm trong `approved_offers`. Kiểm tra độ dài, từ rủi ro và nội dung theo mục 4.6.
- Không dùng tài khoản hoặc domain khác để né chính sách hay đình chỉ.

## Secret và dữ liệu khách hàng

- Không commit token Telegram, chat ID thật, JSON service account, mật khẩu hay credential khác. Repo dùng placeholder.
- `GOOGLE_SA_JSON` chỉ lưu trong secret/biến môi trường phù hợp của Claude Code; không in giá trị ra log, prompt hoặc báo cáo.
- Token Telegram thật chỉ điền trong bản script tại Google Ads theo đặc tả, không đưa ngược bản có token vào Git.
- Không đưa họ tên, số điện thoại, email của lead vào prompt; chỉ đọc số đếm và trạng thái cần thiết.
- Giữ nguyên `.gitignore` bảo vệ dữ liệu khách. Dùng dữ liệu giả cho fixture và kiểm thử; không `git add -f` dữ liệu thật hoặc secret.
- Chỉ hướng dẫn cấp đúng quyền và đúng Sheet cần dùng. Không tự thay đổi quyền truy cập hay cài kết nối ngoài kế hoạch đã duyệt.
- `.gitignore` không phải công cụ phát hiện secret. Kiểm tra nội dung staged/diff trước mỗi commit.

## Triển khai và xác minh

- Bước đầu chưa dùng Google Ads API/developer token và chưa dựng server riêng. Tuân thủ kiến trúc Google Ads Script + Sheets + Telegram + CSV Editor.
- Kiểm tra tài liệu chính thức hiện hành trước khi dùng schema CSV, GAQL hoặc API thay đổi theo thời gian.
- Chốt tên tab/cột trong template trước khi mã hóa; đặc biệt `Danh sách khách` và `Trang thai`/`trang_thai`. Không tự đổi tên trên Sheet thật.
- Lỗi một khách không được làm dừng khách khác; luôn xác nhận Customer ID và phạm vi dữ liệu trước khi ghi.
- Kiểm thử mô phỏng cho luồng duyệt, chống chạy lặp, hạn đề xuất, ngân sách, tracking suspect, ba ngày đầu, Mức 1 và DRY_RUN.
- Tách kiểm thử đọc số liệu, Preview/DRY_RUN và kiểm thử thay đổi thật. Chỉ tắt DRY_RUN cho bài thử thật đã được duyệt, trên đúng tài khoản và đối tượng đã thống nhất.
- Dùng số liệu cùng kỳ/múi giờ để nghiệm thu báo cáo 9h với ngưỡng lệch ≤1% theo đặc tả.
- Phân biệt rõ: đã viết mã, đã kiểm thử mô phỏng, đã kiểm chứng tích hợp và đã nghiệm thu vận hành. Không lấy kết quả cục bộ để khẳng định hệ thống thật hoạt động.
- Ghi lại kết quả kiểm thử và hạn chế thực tế; không đánh dấu hoàn thành nếu còn thiếu đầu ra hoặc cổng duyệt.
