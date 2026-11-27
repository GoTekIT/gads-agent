# gads-agent

Trợ lý AI hỗ trợ Go Media quản lý Google Ads cho nhiều khách hàng: **lập kế hoạch → thiết lập chiến dịch → kiểm tra đo lường → theo dõi → tối ưu → báo cáo**.

**Trạng thái: đã khởi tạo tài liệu, chưa triển khai mã vận hành.** Repo hiện chưa kết nối Google Ads, Google Sheets hay Telegram và chưa có chức năng chạy tự động.

## Tài liệu trong repo

| Tệp | Mục đích |
| --- | --- |
| [BUILD_SPEC.md](BUILD_SPEC.md) | Đặc tả gốc để Claude Code xây dựng Agent. Giữ nguyên nội dung được bàn giao. |
| [CLAUDE.md](CLAUDE.md) | Quy tắc làm việc, giới hạn tự động hóa và cách kiểm chứng kết quả. |
| [README.md](README.md) | Hướng dẫn bắt đầu, tổng quan giải pháp và lộ trình triển khai. |
| [.gitignore](.gitignore) | Loại trừ các đường dẫn thường chứa secret, dữ liệu khách hàng và tệp tạm. |

`BUILD_SPEC.md` là nguồn yêu cầu chính. README và CLAUDE.md là tài liệu bổ sung; khi có điểm khác nhau hoặc chưa rõ, Claude cần nêu cụ thể và xin chủ dự án chốt trước khi triển khai phần liên quan.

## Agent dự kiến làm gì?

- Lập kế hoạch ngân sách, từ khóa, nội dung quảng cáo và phương án đo lường cho từng khách.
- Xuất tệp CSV để người dùng xem lại và nhập qua Google Ads Editor. Chiến dịch, nhóm quảng cáo và quảng cáo mới phải ở trạng thái tạm dừng.
- Theo dõi tài khoản, đối soát số chuyển đổi với số lead trong Sheet, phát hiện bất thường và tạo đề xuất.
- Chỉ áp dụng thay đổi đã được duyệt; chế độ tự động Mức 1 chỉ bật riêng cho từng khách khi người dùng yêu cầu.
- Tạo báo cáo ngày, tuần và tháng; tách số lead trên Ads, lead thực tế và lead hợp lệ.

## Giải pháp theo đặc tả

| Thành phần | Vai trò dự kiến |
| --- | --- |
| Claude Code trên web | Đọc yêu cầu, lập kế hoạch, viết quảng cáo, phân tích và xuất CSV. |
| Google Ads Script cấp MCC | Theo dõi số liệu và thực hiện các thay đổi thuộc loại hỗ trợ, sau khi qua điều kiện duyệt. |
| Google Sheets | Lưu cấu hình, kế hoạch, số liệu, đề xuất, quyết định duyệt và nhật ký. |
| Telegram | Nhận báo cáo và cảnh báo vận hành. |
| Google Ads Editor | Người dùng xem lại, nhập và đăng chiến dịch từ CSV. |
| Các công cụ Python | Đọc/ghi Sheet, chấm điểm từ khóa, dự báo, kiểm tra nội dung và tạo CSV. Chưa được triển khai. |

Ba bước dựng ban đầu sử dụng MCC hiện có; không yêu cầu server riêng hoặc Google Ads API developer token. Quyền truy cập Google Sheets của Claude dự kiến dùng service account và thư viện `gspread`.

Nhịp theo dõi dự kiến theo GMT+7:

- **09:00:** chốt số liệu hôm qua, cập nhật Sheet và gửi báo cáo Telegram cho từng khách đang bật.
- **16:00:** kiểm tra diễn biến trong ngày; chỉ nhắn khi có cảnh báo hoặc đề xuất mới.
- **Thứ Hai:** cập nhật báo cáo tuần. **Ngày 1:** cập nhật báo cáo tháng.

Đây là yêu cầu thiết kế; lịch chạy chưa được cài đặt.

## Bắt đầu với Claude Code

1. Mở Claude Code trên web bằng tài khoản được công ty cho phép sử dụng.
2. Kết nối GitHub và chọn repo private `GoTekIT/gads-agent`. Nếu chưa thấy repo, kiểm tra tài khoản GitHub đang dùng và phạm vi repo đã cấp cho kết nối.
3. Gửi yêu cầu mẫu dưới đây.
4. Xem kế hoạch Claude đề xuất, thống nhất phần việc và duyệt trước khi Claude triển khai.
5. Sau mỗi bước, yêu cầu Claude báo rõ tệp đã tạo, kết quả kiểm thử, thao tác người dùng còn phải làm và phần chưa kiểm chứng trên tài khoản thật.

Yêu cầu mẫu để bắt đầu:

```text
Hãy đọc toàn bộ BUILD_SPEC.md, CLAUDE.md và README.md trong repo này.
Tôi muốn xây dựng AI Agent Google Ads cho Go Media theo đúng đặc tả.

Trước tiên hãy kiểm tra hiện trạng repo, liệt kê điểm chưa rõ hoặc mâu thuẫn,
và đề xuất kế hoạch Bước 1 bằng tiếng Việt đơn giản để tôi duyệt.
Chờ tôi duyệt trước khi triển khai. Chưa kết nối hoặc thay đổi tài khoản Ads thật.

Khi được duyệt, dựng theo từng bước, kiểm thử rồi mới chuyển bước.
Phân biệt rõ chức năng đã viết, đã kiểm thử mô phỏng và đã xác minh trên tài khoản thật.
```

Các lệnh `/cai-dat`, `/khach-moi`, `/set-camp`, `/toi-uu-tuan`, `/bao-cao-thang` và `/ap-dung` được mô tả trong đặc tả; **hiện chưa có trong repo**. Claude sẽ tạo chúng trong quá trình triển khai.

## Chuẩn bị khi tới bước kết nối

Claude cần hướng dẫn từng thao tác bằng tiếng Việt, kèm menu để người dùng thực hiện. Không cần chuẩn bị toàn bộ secret ngay khi tạo repo.

| Nội dung | Cách sử dụng theo đặc tả |
| --- | --- |
| Tài khoản Google Ads và MCC hiện có | Xác nhận đúng Customer ID, quyền, tiền tệ và múi giờ trước khi thử. |
| Bot Telegram | Tạo qua BotFather; token thật chỉ điền trong bản script dán vào Google Ads. Repo dùng `<DIEN_TOKEN>`. |
| Google Cloud service account | Bật Sheets API và Drive API theo hướng dẫn; lưu JSON trong secret `GOOGLE_SA_JSON` của môi trường Claude Code. |
| Sheet tổng và Sheet mẫu | Tạo theo schema trong đặc tả; chia sẻ quyền Editor cho đúng email service account khi được yêu cầu. |
| Dữ liệu khách đầu tiên | Brief, ngân sách và cách tính phí, KPI, URL trang đích, ưu đãi đã duyệt, nguồn lead và CSV Keyword Planner khi tới bước lập kế hoạch. |

Không dán token hoặc JSON khóa vào tin nhắn, README, issue hay commit. Không đưa tên, số điện thoại hoặc email của lead vào prompt; chỉ dùng số đếm và trạng thái.

## Lộ trình triển khai và nghiệm thu

Thời lượng dưới đây là ước lượng trong BUILD_SPEC.md, chưa phải lịch cam kết.

| Bước | Đầu ra phải có | Điều kiện nghiệm thu chính |
| --- | --- | --- |
| **1. Theo dõi và báo cáo** — 2–3 ngày | Knowledge base, mẫu Sheet, lệnh `/cai-dat`, script MCC hai chế độ 9H/16H. Bắt đầu bằng `DRY_RUN=true`. | Thử một tài khoản đã thống nhất; báo cáo 9h lệch không quá 1% so với số đối chiếu Ads cùng kỳ; bản 16H chỉ nhắn khi cần. Sau khi được duyệt cho phép ghi thật, thử một đề xuất đã tick duyệt, xác minh thay đổi và nhật ký. |
| **2. Lập kế hoạch và set camp** — 2–3 ngày | Skill khách mới, skill set camp, chấm điểm từ khóa, dự báo, kiểm tra RSA và CSV Editor. | Brief mẫu cho ra đủ kế hoạch; CSV nhập Editor không lỗi; QA bắt được lỗi ngân sách, thiếu negative và headline quá 30 ký tự. |
| **3. Tối ưu tuần và Mức 1** — 1 tuần dựng | Skill tối ưu/báo cáo tháng, áp dụng thay đổi qua Editor; allowlist Mức 1 theo từng khách. | Chỉ bật tự động khi được yêu cầu; theo dõi đủ hai tuần không có thay đổi ngoài ý muốn. Thời gian theo dõi này cần tính riêng khi chốt lịch. |

Mỗi bước đều theo trình tự: **báo kế hoạch → người dùng duyệt → làm → kiểm thử → tóm tắt kết quả**. Kiểm thử tại máy hoặc mô phỏng không thay cho kiểm chứng kết nối thật.

## Các giới hạn phải giữ

- Chỉ bắt đầu chạy quảng cáo khi kế hoạch, ngân sách, nội dung và kiểm tra tracking đã đạt điều kiện GO.
- Khi nghi lỗi tracking, chặn đề xuất tối ưu; chỉ giữ các hành động an toàn theo đặc tả.
- Tổng ngân sách ngày × 30,4 không vượt Net Media đã duyệt. Một lần đổi ngân sách không quá 30%; Mức 1 tối đa ±10% và vẫn phải trong tổng ngân sách.
- Mức 1 chỉ gồm ba nhóm: phủ định truy vấn khớp nhóm đã cấu hình, tạm dừng quảng cáo lỗi/chính sách và chỉnh ngân sách trong giới hạn. Mặc định `automation_level=0`.
- Không áp dụng lặp lại đề xuất đã xử lý; ghi kết quả, người duyệt và nhật ký cho mọi thay đổi. Đề xuất chưa duyệt quá ba ngày hết hạn.
- Tỷ lệ tính từ tổng số liệu; mẫu số bằng 0 hiển thị `N/A`. Mỗi nhận định trong báo cáo phải có số liệu hỗ trợ.
- Chỉ dùng giá và ưu đãi có trong `approved_offers`. Không né chính sách bằng tài khoản hoặc domain khác.

## Cấu trúc sẽ được dựng

Đây là cấu trúc mục tiêu trong mục 3 của đặc tả. Ngoài bốn tệp tài liệu đã liệt kê đầu trang, các thành phần dưới đây chưa được tạo:

```text
.claude/skills/           # 4 skill nghiệp vụ
.claude/commands/         # 6 lệnh điều khiển quy trình
ads_scripts/agent_mcc.js  # script vận hành tại MCC
scripts/                 # công cụ Python
knowledge/               # 9 chủ đề kiến thức tại mục 4
templates/               # cấu hình khách, cấu trúc Sheet, cột CSV Editor
clients/<ma-khach>/       # brief, cấu hình, kế hoạch, tệp build và ghi chú
```

`clients/` được bỏ qua trong Git để tránh đưa dữ liệu khách hàng lên repo theo mặc định. Khi cần dữ liệu mẫu để kiểm thử, dùng dữ liệu giả và ghi rõ đó là mẫu.

## Điểm cần chốt trước khi viết mã liên quan

- **Tên tab/cột:** đặc tả có tab Master `Danh sách khách` có dấu và hai cách ghi cột trạng thái (`Trang thai`, `trang_thai`). Cần chốt schema thống nhất trong mẫu Sheet, script và hướng dẫn; không tự đổi tên Sheet đang có dữ liệu.
- **Chế độ thử:** `DRY_RUN=true` không được áp dụng thay đổi. Bài kiểm thử áp dụng đề xuất thật phải là bước riêng, được duyệt trước khi tắt chế độ thử.
- **Thứ tự và giới hạn:** quy tắc ba ngày đầu, cờ nghi lỗi tracking và giới hạn ngân sách phải được kiểm tra trước cả đề xuất đã duyệt lẫn hành động Mức 1.
- **Quy mô:** thiết kế hiện dùng `executeInParallel` tối đa 50 tài khoản theo đặc tả. Nếu số khách vượt giới hạn thiết kế, cần chốt cách chia lượt và kiểm thử trước khi mở rộng.
- **Tài liệu thay đổi theo thời gian:** trước khi xuất CSV hoặc tích hợp, kiểm tra tài liệu chính thức hiện hành về Ads Editor, Google Ads Scripts, GAQL và khả năng cấp secret của môi trường Claude Code. Không coi nội dung đặc tả là bằng chứng các kết nối đã hoạt động.

## Ghi nhận thay đổi

Mọi thay đổi yêu cầu cần ghi rõ lý do, ảnh hưởng tới ngân sách/tracking/quyền truy cập và kết quả kiểm chứng. Không sửa âm thầm BUILD_SPEC.md để làm khớp với mã đã viết.

Tài liệu khởi tạo ngày **27/11/2026** cho repo private của **GoTekIT**.
