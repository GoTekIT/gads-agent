# BUILD_SPEC — AI Agent Google Ads (đặc tả để Claude Code dựng)

> **Gửi Claude Code:** người dùng (chủ dự án, Go Media) **không biết lập trình** và còn ít kinh nghiệm Google Ads.
> Hãy tự quyết mọi chi tiết kỹ thuật theo đặc tả này. Mọi giải thích cho người dùng viết bằng tiếng Việt đơn giản, không thuật ngữ code.
> Chỉ nhờ người dùng làm việc bắt buộc phải là con người (đăng nhập, bấm cấp quyền, dán đoạn mã, tick duyệt), và luôn kèm hướng dẫn từng bước có đường dẫn menu.
> **Trước khi làm bất kỳ việc gì, báo kế hoạch ngắn cho người dùng và chờ họ duyệt.**
> Kiến thức Google Ads ở mục 4 rút từ giáo trình nội bộ 6 ngày + mẫu proposal của Go Media. Ví dụ và case study trong giáo trình không đưa vào Agent; mọi ngưỡng là mặc định ban đầu, chỉnh theo từng khách.

---

## 0. Mục tiêu và nguyên tắc

Agent giúp Go Media làm trọn vòng Google Ads cho nhiều khách: **Set camp → Tracking → Theo dõi → Tối ưu → Báo cáo**, giảm tối đa thời gian của người dùng.

1. **Việc lặp lại chạy trong Google Ads Script, việc cần suy nghĩ mới dùng Claude.** Theo dõi 2 lần/ngày, báo cáo số liệu, áp dụng thay đổi đã duyệt: Script. Lập kế hoạch, viết quảng cáo, set camp, nhận xét và tối ưu sâu: Claude Code.
2. **Không thay đổi nào tự chạy nếu chưa duyệt** (trừ allowlist Mức 1 khi người dùng bật). Chiến dịch/quảng cáo mới luôn PAUSED.
3. **Tracking trước, tối ưu sau.** Lệch tracking = sự cố khẩn, chặn mọi đề xuất tối ưu.
4. **Không quy tắc cứng theo ngày** (không "ngày 8 đổi giá thầu", không "20 click thì tắt"); quyết định theo dữ liệu.
5. **Lead thật quan trọng hơn conversion.** Luôn tách Lead (Ads), Lead thật (Sheet), Lead hợp lệ, CPL, CPQL.

---

## 1. Yêu cầu vận hành của người dùng (bắt buộc)

- Mọi dự án đã có sẵn tài khoản Google Ads nằm dưới **MCC** của Go Media. Không cần bước tạo hay bàn giao tài khoản.
- **Mọi thông báo qua bot Telegram.**
- **Báo cáo tuần/tháng nằm trong Google Sheet**, so sánh tuần này với tuần trước, tháng này với tháng trước (khách chạy chưa đủ kỳ thì so với kế hoạch). Dùng để gửi khách hoặc nội bộ.
- **Lịch chạy sau khi set camp: 2 lần/ngày** – 09:00 (kèm gửi báo cáo ngày hôm qua qua Telegram) và 16:00 (theo dõi để tối ưu, chỉ nhắn khi có cảnh báo/đề xuất). Múi giờ GMT+7.

## 2. Quyết định kỹ thuật (đã chốt)

| Vấn đề | Lựa chọn |
|---|---|
| Phần chạy hằng ngày | **1 Google Ads Script cấp MCC** (`ads_scripts/agent_mcc.js`), dán thành 2 bản: bản `MODE='9H'` lịch hằng ngày 9:00, bản `MODE='16H'` lịch hằng ngày 16:00. Chạy song song cho mọi khách đang bật (`AdsManagerApp.accounts().withIds(...).executeInParallel`, tối đa 50 tài khoản). Không cần developer token, không server. |
| Cấu hình nhiều khách | **Sheet tổng** "GADS Agent – Master", tab `Danh sách khách`: customer_id, client_code, client_name, report_sheet_url, lead_sheet_url, lead_tab, lead_date_col, lead_status_col, qualified_values, active (TRUE/FALSE), automation_level (0/1), go_live_date. |
| Dữ liệu từng khách | **Sheet báo cáo của khách** tạo từ Sheet mẫu, các tab: `Cau hinh`, `Ke hoach`, `Bao cao ngay`, `Bao cao tuan`, `Bao cao thang`, `De xuat`, `Nhat ky`, `Tu khoa tim kiem`. |
| Telegram | Bot riêng (người dùng tạo qua BotFather). Script gọi `https://api.telegram.org/bot<TOKEN>/sendMessage` bằng `UrlFetchApp`, `parse_mode=HTML`. Token và chat_id là hằng số ở đầu script dán trong Google Ads; bản trong repo để trống `'<DIEN_TOKEN>'`. |
| Duyệt đề xuất | Tab `De xuat` có cột checkbox `Duyet`. Lần chạy kế tiếp (9h hoặc 16h) đọc các dòng `Duyet=TRUE` và `Trang thai=Cho duyet`, áp dụng, ghi kết quả + `Nhat ky`. Dòng chưa duyệt quá 3 ngày → `Het han`. |
| Claude Code đọc/ghi Sheet | Service account Google Cloud; JSON lưu trong biến môi trường/secret `GOOGLE_SA_JSON` của môi trường Claude Code (không commit). Người dùng share Sheet tổng + Sheet khách cho email service account (quyền Editor). Thư viện `gspread`. |
| Set camp | Claude Code xuất **CSV cho Google Ads Editor** (người dùng: Tài khoản → Nhập → Dán/Chọn file → Xem lại → Đăng). Mọi chiến dịch `Campaign status = Paused`. Kiểm tra tiêu đề cột theo tài liệu import CSV của Google Ads Editor hiện hành trước khi xuất. |
| Keyword Planner | Người dùng xuất CSV từ Keyword Planner (hướng dẫn trong skill) và gửi vào Claude Code. |
| Google Ads API / developer token | **Không dùng** trong 3 bước dựng. Chỉ cân nhắc sau nếu người dùng muốn bỏ bước Editor. |
| Nơi chạy Claude Code | Claude Code trên web, repo GitHub private `gads-agent`. |
| Ngôn ngữ | Tiếng Việt, câu ngắn. Tên tab/cột Sheet viết không dấu để script ổn định. |

## 3. Cấu trúc repo và CLAUDE.md

```
gads-agent/
  CLAUDE.md                # luật làm việc (bên dưới)
  README.md                # tiếng Việt cho người dùng
  .claude/skills/          # 4 skill (mục 5)
  .claude/commands/        # /cai-dat /khach-moi /set-camp /toi-uu-tuan /bao-cao-thang /ap-dung
  ads_scripts/agent_mcc.js # script MCC (mục 6)
  scripts/                 # Python: sheets_io.py, kw_score.py, forecast.py, rsa_lint.py, build_editor_csv.py, weekly_pack.py
  knowledge/               # mục 4, mỗi chủ đề 1 file
  templates/               # client.yaml, cau truc Sheet mau (sheet_template.md), editor_csv_columns.md
  clients/<ma-khach>/
    brief/                 # brief, proposal người dùng gửi
    client.yaml            # bản sao cấu hình (nguồn chính vẫn là tab Cau hinh trong Sheet)
    plans/  build/  notes/
```

**CLAUDE.md phải có:**
- Người dùng không biết code: không bảo họ chạy lệnh terminal; thao tác thủ công thì viết từng bước kèm menu.
- Báo kế hoạch và chờ người dùng duyệt trước khi làm.
- Không áp dụng thay đổi nào khi chưa có duyệt; chiến dịch/nhóm/quảng cáo mới luôn PAUSED.
- Tổng ngân sách ngày × 30,4 không vượt Net Media đã duyệt; một lần đổi ngân sách không quá 30% (Mức 1 tự động: ±10%).
- Quảng cáo chỉ dùng giá/ưu đãi có trong `approved_offers`.
- Không đưa họ tên, số điện thoại, email của lead vào prompt; chỉ đọc số đếm và trạng thái.
- Không commit secret (token Telegram, JSON service account).
- Tỷ lệ tính từ tổng, không lấy trung bình cộng; mẫu số 0 → `N/A`.
- Không dùng tài khoản/domain khác để né chính sách hay đình chỉ.
- Brief thiếu thông tin → dùng mặc định trong `knowledge/`, ghi vào `assumptions`, liệt kê để hỏi người dùng.

---

## 4. Knowledge base (tạo thành các file trong `knowledge/`)

### 4.1 `kpi_formulas.md`
- CTR = Clicks / Impressions; CPC = Cost / Clicks; CPM = Cost / Impressions × 1000
- CVR Lead = Lead / Clicks; CPA = Cost / Conversions; CPL = Cost / Lead đã lọc trùng
- Qualified rate = Qualified Leads / Raw Leads; CPQL = Cost / Qualified Leads
- CAC, ROAS khi có số chốt/doanh thu từ Sales
- CPL = CPC / CVR → CPC mục tiêu = CPL mục tiêu × CVR
- Forecast: Clicks = Budget / CPC; Impressions = Clicks / CTR; Leads = Clicks × CVR; CPA = Budget / Leads
- Ngân sách: Net Media = Gross × (1 − fee%) nếu phí tính trên Gross; = Gross / (1 + fee%) nếu phí tính trên Net. Daily budget = Media tháng / 30,4. Google có thể chi tới 2× ngân sách ngày trong 1 ngày; giới hạn tháng ≈ ngày × 30,4.
- Pacing = Actual Spend / Planned Spend lũy kế. 90–110% bình thường. Ngân sách ngày còn lại = (Media − đã chi) / số ngày còn lại.
- Báo cáo tách Search và Demand Gen; không dùng một ngưỡng CTR chung. Báo cáo cả chi phí media và tổng chi phí gồm phí.
- Lưu ý độ trễ chuyển đổi và ngày chốt số liệu; GA4 và Ads có thể lệch do attribution.

### 4.2 `account_structure.md`
- Cấp: MCC → Account → Campaign (mục tiêu, ngân sách, giá thầu, khu vực, lịch) → Ad Group (chủ đề keyword + ads) → Keyword/Ads.
- Tách campaign khi cần ngân sách, khu vực, mục tiêu hoặc chiến lược giá thầu riêng. Ngân sách nhỏ thì không chia nhỏ. Ad Group không có ngân sách riêng.
- Khởi đầu theo nhóm sản phẩm, chưa tách sản phẩm × khu vực nếu dữ liệu ít.
- Cấu trúc Search lead gen mẫu: C01 Sản phẩm × Khu vực, C02 Giá & Ưu đãi, C03 Đối thủ, C04 Remarketing (Demand Gen) – chỉ bật C04 khi tệp đủ điều kiện.
- Naming: `<MAKHACH>_<KENH>_<MUCTIEU>_<CHUDE>[_Vxx]`, Ad Group `<SANPHAM>_<CHUDE>`, RSA `RSA_<GOC>_Vxx`, Conversion `<MAKHACH>_WEB_LEAD_FORM_SUCCESS`, Audience `<MAKHACH>_WEB_VISITORS_30D`, Negative list `<MAKHACH>_NEG_IRRELEVANT`.
- Quyền: Admin / Standard / Read only / Billing. MCC liên kết không chuyển quyền sở hữu hay nghĩa vụ thanh toán.

### 4.3 `keywords.md`
- Match type: Broad (rộng, đi với Smart Bidding khi tracking tin cậy), Phrase `"..."`, Exact `[...]`. Tracking chưa tin cậy → ưu tiên Phrase + Exact, chưa dùng Broad đại trà.
- Intent: Transactional (cao), Navigational (cao nếu đúng), Commercial (thử nghiệm), Informational (đánh giá thêm).
- Seed matrix: Sản phẩm + Nhu cầu (giá, báo giá, trả góp, ưu đãi, mua, đăng ký, tư vấn) + Khu vực + Đối thủ.
- Chấm điểm 100: Relevance, Intent, Volume, Cost, Landing page. 80–100 P1 Launch; 60–79 P2 Test; 40–59 P3 Hold; <40 P4 Không chạy. Từ khóa không liên quan hoặc không có landing page phù hợp → xử lý trước, bất kể điểm.
- Keyword Planner: volume là trung bình tháng; Competition là cạnh tranh quảng cáo; Top of page bid là khoảng tham khảo, không phải CPC cam kết. Không cộng máy móc volume biến thể gần nghĩa.
- Negative seed (áp dụng sau khi xác nhận ngành): tuyển dụng/việc làm; hình nền/game/đồ chơi/mô hình; thuê; tài liệu/manual/pdf; phụ tùng/sửa chữa (xác nhận trước); hàng cũ (xác nhận trước). Không phủ định máy móc "giá", "trả góp", "so sánh", "đánh giá".
- Phạm vi phủ định: account-level, shared list, campaign, ad group. Phủ định chéo chỉ khi cần tách thông điệp/ngân sách/báo cáo.
- Keyword Master columns: Keyword, Product, Location, Intent, Category, Avg Monthly Searches, Competition, Bid Low, Bid High, Est CPC, Score, Priority, Match Type, Campaign, Ad Group, Landing Page, Status (Launch/Test/Hold/Exclude), Notes. Sheet nghiên cứu tách khỏi sheet upload; chỉ upload từ khóa đã duyệt.

### 4.4 `tracking.md`
- Vai trò: Google Ads (chi phí, conversion, bidding), GA4 (hành vi, nguồn), GTM (thẻ, trigger, biến), CRM/Sheet (lưu lead, đối soát). GTM không tự lưu lead.
- Mã: Customer ID ≠ Conversion ID (AW-) ≠ GTM-XXXX ≠ G-XXXX. Xác nhận đúng tài khoản trước khi liên kết.
- Ưu tiên đo form bằng Data Layer custom event khi backend xác nhận lưu lead (`lead_form_success`); Thank-you page và Form Submission trigger là phương án phụ, phải test tải lại/validation.
- Primary: gửi form thành công, cuộc gọi đủ điều kiện (nếu đo tin cậy). Secondary: click hotline (`tel:`), click Zalo (`zalo.me`), view, scroll. Không để 2 Primary cho cùng 1 hành động. Conversion cũ không liên quan → không đưa vào mục tiêu.
- Conversion action mặc định: Count = One, click-through 30 ngày, attribution data-driven nếu có, chưa gán value.
- Không gửi PII vào tham số GA4. Enhanced Conversions chỉ qua cơ chế Google hỗ trợ, có consent.
- Audit 7 bước: Inventory → Ownership → Mapping → Testing → Deduplication → Implementation → QA & Publish. Hai GTM/GA4 không mặc nhiên đếm trùng; không xóa container cũ khi chưa rõ.
- Test cases: TC01 vào landing page có page_view; TC02 form hợp lệ lưu lead, tag bắn 1 lần; TC03 form thiếu thông tin không ghi lead; TC04 bấm gửi nhiều lần không tạo trùng; TC05 tải lại trang cảm ơn không phát sinh lead; TC06 click hotline; TC07 click Zalo; TC08 form trên mobile; TC09 redirect giữ gclid; TC10 lead test về CRM/Sheet đúng; TC11 consent; TC12 đúng conversion action và tài khoản. Đánh dấu lead test để Sales không nhầm.
- Lộ trình nâng cao: Form + Sheet → chuẩn hóa Lead Status, lọc trùng → Enhanced Conversions for Leads → import Qualified Lead → import Sale/Value.

### 4.5 `bidding_budget.md`
- Tracking chưa kiểm chứng → chưa launch.
- Tracking đúng, ít dữ liệu → Manual CPC hoặc Maximize Clicks (có giới hạn CPC) để kiểm soát truy vấn; hoặc Maximize Conversions nếu phù hợp.
- Tracking đúng, có dữ liệu → Maximize Conversions; cân nhắc tCPA ≈ CPA thực tế gần đây (không đặt quá thấp).
- Có giá trị lead/doanh số → value-based bidding.
- Learning phase: hạn chế đổi liên tục ngân sách, chiến lược, tCPA, conversion; vẫn xử lý ngay lỗi tracking, chính sách, truy vấn lãng phí.
- Phân bổ: chiến lược → theo nhu cầu (Keyword Planner) → theo hiệu quả thực tế. Khi Demand Gen chưa đủ tệp: dồn tạm sang Search.
- Phân pha 30 ngày (tham khảo): ngày 1–7 kiểm chứng; 8–14 tối ưu truy vấn/quảng cáo; 15–21 mở rộng nhóm hiệu quả, remarketing nếu đủ điều kiện; 22–30 tập trung nhóm tốt, tổng kết. Không bắt buộc theo ngày.
- Điều kiện scale (đủ cả 4): CPA chấp nhận được và lead đã được Sales xác nhận; tracking chính xác, dữ liệu đủ; còn nhu cầu tìm kiếm (Lost IS budget); có ngân sách duyệt và landing page/Sales đủ năng lực. Không có quy tắc tăng cố định 10–20%.
- Scale dọc = tăng ngân sách; scale ngang = mở keyword, khu vực, tệp, chiến dịch.

### 4.6 `ads_creative.md`
- RSA: 3–15 headline ≤30 ký tự, 2–4 description ≤90, 2 path ≤15, Final URL đúng trang sản phẩm. Headline đứng độc lập, không lặp ý. Chỉ pin khi bắt buộc. Ad Strength không bảo đảm CPA/QS.
- 6 góc: Product, Price, Promotion, Finance, Trust, Experience/Action. Message match: keyword ↔ ad ↔ landing page.
- Assets: Sitelink (≤25, mô tả ≤35×2, URL thật), Callout ≤25, Structured snippet, Call (theo giờ trực), Location (đúng Business Profile), Image (1200×1200, 1200×628), Price/Promotion chỉ khi đã xác nhận.
- Demand Gen: ảnh 1:1, 1.91:1, 4:5; logo 1:1; headline ≤40 (có 1 cái ≤30), description ≤90; audience remarketing 30/90 ngày; loại trừ người đã chuyển đổi; kiểm tra Optimized Targeting.
- Từ/cụm cấm hoặc rủi ro (lint tự động): "rẻ nhất", "số 1", "duy nhất", "duyệt vay 100%", "cam kết", số tiền giảm/ưu đãi chưa có trong brief, "chỉ còn X" chưa xác minh, viết HOA toàn bộ, "!!!". Tên thương hiệu đối thủ trong nội dung quảng cáo → cảnh báo.

### 4.7 `setup_qa.md`
- Setup Search mặc định: mục tiêu Leads; chọn đúng conversion goal (không dùng mặc định toàn tài khoản máy móc); Search Partners OFF, Display OFF lúc đầu; Location option = Presence; ngôn ngữ theo thị trường; audience = Observation; AI Max / Broad / Final URL expansion OFF khi cần kiểm soát; xem từng nhóm Auto-apply recommendations; ad schedule cả ngày lúc đầu; call asset theo giờ trực.
- Checklist bàn giao TKQC: Customer ID, quyền, MCC, múi giờ, tiền tệ, billing/hạn mức, xác minh nhà quảng cáo, chính sách, lịch sử thay đổi, conversion cũ, liên kết GA4/GTM, auto-apply.
- Pre-launch QA (mọi mục phải PASS): Account & Campaign (ID, billing, currency/timezone, verification, naming, objective/type, budget = media plan, bidding duyệt, networks, location, language/schedule, tự động hóa) · Keywords & Audience (đã duyệt, match type, negatives, không chặn nhầm, exclusions) · Ads & Assets (đúng giới hạn, giá/ưu đãi đã xác nhận, sitelink/callout đúng, hotline/location đúng, creative được phép) · Landing & Tracking (URL đúng, mobile, form gửi được, lead về Sheet/CRM, Primary không trùng, hotline/Zalo tracking, consent) · Approval (Media Plan duyệt, nội dung duyệt, có người theo dõi, chưa tiêu tiền trước duyệt).
- GO chỉ khi: ngân sách chính xác + tracking chính xác + nội dung đã duyệt.

### 4.8 `operations.md`
- Theo dõi sau launch: 0–2h trạng thái/billing; 2–24h impressions, clicks, spend, landing page; 24–48h search terms, CPC, CTR, tracking; 48–72h đối soát lead; ngày 4–7 tổng hợp. Không thay đổi hàng loạt chỉ vì chưa có lead.
- Search terms: sắp theo chi phí. KEEP (liên quan, hiệu quả) · ADD (giá trị, cần kiểm soát riêng → thêm keyword) · MONITOR (phù hợp, chưa đủ dữ liệu) · EXCLUDE (không liên quan → phủ định). Không loại truy vấn liên quan chỉ vì chưa có lead; so chi phí với CPA mục tiêu.
- Ma trận chẩn đoán: Impr thấp → nhu cầu/targeting/Ad Rank (IS, keyword) · CTR thấp → nội dung/truy vấn (search terms, RSA, assets) · CPC cao → cạnh tranh/giá thầu (Auction Insights, QS) · CVR thấp → landing/traffic/tracking · CPA cao → CPC hoặc CVR · Lead ảo → spam/mục tiêu sai (CRM, tracking, Sales) · Doanh số thấp → chất lượng lead/tư vấn.
- Lost IS Budget cao + CPA tốt → cân nhắc ngân sách. Lost IS Rank cao → chất lượng, landing, giá thầu (tăng ngân sách không giải quyết).
- Quality Score: Expected CTR, Ad Relevance, Landing Page Experience; là chẩn đoán, không phải đầu vào đấu giá.
- Chẩn đoán sự cố theo thứ tự: Không tiêu tiền: billing → trạng thái → targeting/lịch/phủ định → giá thầu/Ad Rank. Click không lead: thử form → GTM/conversion → CRM → search terms/landing. CPA cao: CPC & CVR → độ trễ → search terms/ad group/thiết bị → change history. Lead không hợp lệ: định nghĩa → nguồn → spam/trùng → tốc độ Sales. Vượt tiến độ: net media → change history → ngân sách còn → giới hạn chi.
- Mức sự cố: P1 Critical (tracking sai, vượt hạn mức, web lỗi, đình chỉ) → ngay · P2 High (CPA tăng mạnh, mất phân phối, lead bất thường) → trong ngày · P3 Medium (CTR thấp, nhóm kém) → theo lịch tuần · P4 Low (assets, đặt tên) → theo kế hoạch.
- Nhịp tuần: Thứ Hai báo cáo + ưu tiên; Thứ Ba search terms/keyword; Thứ Tư RSA/assets/landing; Thứ Năm campaign/bidding/ngân sách; Thứ Sáu đối soát lead.
- Change log cột: date, item, change, reason (kèm số liệu), expected_effect, review_date, approved_by, applied_by.

### 4.9 `reporting.md`
- Báo cáo tuần 5 phần: Executive Summary (3–5 câu, kết quả vs KPI) · Performance Dashboard (spend, impr, clicks, CTR, CPC, leads, CPL, qualified, CPQL, pacing) · Data Analysis (theo campaign, sản phẩm, khu vực, thiết bị; vấn đề + nguyên nhân có bằng chứng) · Optimization Completed · Next Action Plan.
- Báo cáo tháng: thêm tổng spend gồm phí, CAC/ROAS nếu có, bài học, Media Plan kỳ sau.
- Mỗi nhận định phải kèm con số. Không coi CPC thấp là thành công nếu lead không hợp lệ.
- Khung Media Plan (8 sheet): Project Brief, Budget Allocation, Campaign Structure, KPI Forecast, Scenario Planning (thận trọng/kỳ vọng/tích cực), Timeline & Pacing, Tracking Plan, Reporting & Optimization.
- Khung Proposal: Bối cảnh & yêu cầu → Thị trường → Đối thủ (Ads Transparency Center, Google Search) → Audit website → Media Plan & KPI → Thông điệp mẫu → Kế hoạch 30 ngày → Bước tiếp theo.

---

## 5. Bốn skill Claude Code

Mỗi `SKILL.md`: mô tả 1 câu khi dùng · input bắt buộc và điều kiện dừng · các bước (ghi rõ bước nào chạy script Python) · mẫu đầu ra cố định · file `knowledge/` cần mở.

1. **gads-khach-moi** (`/khach-moi <ma>`) — Khách mới & lập kế hoạch.
   - Input: file trong `clients/<ma>/brief/`, customer_id, link Sheet lead, CSV Keyword Planner (nếu chưa có thì hướng dẫn người dùng xuất).
   - Bước: đọc brief → điền `client.yaml` + liệt kê câu hỏi thiếu → kiểm tra nhanh website (PageSpeed Insights API không cần key cho lượng nhỏ; quét `tel:`, `zalo.me`, form, mã GTM/GA4/AW có trên trang) → seed matrix + chấm điểm từ khóa (`kw_score.py`) + Claude phân loại intent, mapping → negatives → chia ngân sách + daily budget + forecast 3 kịch bản (`forecast.py`) → tracking plan + hướng dẫn gắn tracking cho web dev (ngắn, theo `tracking.md`).
   - Đầu ra: hướng dẫn người dùng tạo bản sao Sheet mẫu và share cho service account; Claude ghi tab `Cau hinh`, `Ke hoach` (1 trang), file `plans/tracking_guide.md`. Thêm 1 dòng vào Sheet tổng với `active=FALSE`.
   - **Cổng 1**: người dùng duyệt kế hoạch (ghi `approved` trong tab `Ke hoach`).
2. **gads-set-camp** (`/set-camp <ma>`) — dừng nếu kế hoạch chưa duyệt.
   - Bước: viết RSA (15 headline/4 description mỗi nhóm), sitelink, callout, snippet theo `ads_creative.md` → `rsa_lint.py` (ký tự, từ cấm, giá/ưu đãi ngoài `approved_offers`, naming) → `build_editor_csv.py` (campaign, ad group, keyword, negative, RSA, asset, budget, bidding, location, language, networks; status Paused) → tự kiểm tra theo `setup_qa.md` và so với tab `Ke hoach` → phiếu test tracking rút gọn 5 mục (form hợp lệ bắn 1 lần; form lỗi không ghi; hotline/Zalo; mobile; lead test về Sheet).
   - Đầu ra: `build/editor_import.csv`, bảng tự kiểm tra, phiếu test, hướng dẫn nhập Editor từng bước.
   - **Cổng 2 (GO)**: phiếu test đạt + người dùng nhập Editor, bật chiến dịch, đổi `active=TRUE`, điền `go_live_date` trong Sheet tổng.
3. **gads-toi-uu** (`/toi-uu-tuan <ma>`, `/bao-cao-thang <ma>`) — phiên ~15 phút.
   - Bước: `weekly_pack.py` đọc Sheet (báo cáo tuần, ngày, từ khóa tìm kiếm, nhật ký, đề xuất cũ) → tóm tắt ≤300 dòng → Claude chẩn đoán theo `operations.md` → ghi ô **Nhận xét** (Executive Summary 3–5 câu, vấn đề + nguyên nhân có số liệu, kế hoạch tuần tới) vào tab `Bao cao tuan`/`Bao cao thang` → phân loại truy vấn mới (bỏ qua truy vấn đã có quyết định) → ghi đề xuất vào tab `De xuat` (loại script áp dụng được, hoặc `EDITOR_CHANGE`) → đoạn tóm tắt 5 dòng để người dùng dán Zalo/email cho khách.
   - Cuối tháng: thêm Media Plan tháng sau (tab `Ke hoach`, chờ duyệt).
4. **gads-ap-dung** (`/ap-dung <ma>`) — chỉ cho đề xuất `EDITOR_CHANGE` đã duyệt: xuất CSV Editor + hướng dẫn; ghi `Nhat ky`.

Thêm lệnh **`/cai-dat`** (chạy 1 lần): kiểm tra secret, hướng dẫn người dùng từng việc ở mục 10, tạo Sheet tổng và Sheet mẫu (hoặc hướng dẫn tạo), điền script với token, kiểm tra script bằng chế độ Xem trước (Preview).

---

## 6. Đặc tả `ads_scripts/agent_mcc.js`

Hằng số đầu file: `MODE` ('9H'|'16H'), `MASTER_SHEET_URL`, `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`, `TIMEZONE='Asia/Ho_Chi_Minh'`, `DRY_RUN` (true khi thử: không áp dụng, chỉ nhắn).

Luồng chính:
1. Đọc tab `Danh sách khách`, lấy các dòng `active=TRUE` → `executeInParallel(processAccount, allDone, JSON cấu hình)`.
2. `processAccount` (mỗi tài khoản, giới hạn 30 phút):
   a. **Áp dụng đề xuất đã duyệt** (cả 2 MODE). Loại hỗ trợ: `ADD_NEGATIVE` (campaign hoặc shared list, match type), `PAUSE_KEYWORD`, `PAUSE_AD`, `SET_DAILY_BUDGET` (kiểm guardrail tổng ngân sách và % thay đổi), `ENABLE_KEYWORD`, `ADJUST_KEYWORD_CPC` (chỉ khi Manual CPC). Loại khác → bỏ qua, ghi "Cần Claude/Editor". Mỗi lần áp dụng ghi `Nhat ky` (thời gian, loại, đối tượng, trước/sau, người duyệt = "Sheet tick", nguồn đề xuất).
   b. **Mức 1** (nếu `automation_level=1`): tự áp dụng không cần duyệt chỉ 3 loại: phủ định truy vấn khớp danh sách nhóm từ rõ ràng trong tab `Cau hinh` (tuyển dụng, việc làm, hình nền, game, đồ chơi, ...), tạm dừng quảng cáo Disapproved hoặc Final URL lỗi 4xx/5xx, chỉnh daily budget ±10% để kéo pacing về 90–110% mà không vượt tổng. Vẫn ghi nhật ký và nhắn Telegram.
   c. **MODE 9H**: chốt số hôm qua (campaign, ad group, keyword, search term, device, geo) qua `AdsApp.search` GAQL; ghi 1 dòng `Bao cao ngay`; đọc Sheet lead (đếm lead theo ngày, lead hợp lệ theo `qualified_values`) → đối soát; chạy rules (mục 7); ghi đề xuất; cập nhật `Tu khoa tim kiem` (truy vấn mới 7 ngày, chi phí, click, conv; không ghi đè cột quyết định); thứ Hai cập nhật `Bao cao tuan` (tuần trước vs tuần trước nữa); ngày 1 cập nhật `Bao cao thang` (tháng trước vs tháng trước nữa); giữa kỳ thì so cùng số ngày; khách chưa đủ kỳ thì so với `Ke hoach`.
   d. **MODE 16H**: số hôm nay đến giờ (pacing trong ngày so với daily budget × tỷ lệ giờ), trạng thái quảng cáo, kiểm tra Final URL (cache URL duy nhất, `muteHttpExceptions`), chi phí không lead hôm nay, truy vấn lãng phí → rules → đề xuất.
   e. Trả về tóm tắt JSON cho `allDone`.
3. `allDone`: MODE 9H gửi 1 tin Telegram mỗi khách theo mẫu mục 8.1; MODE 16H chỉ gửi khi có cảnh báo hoặc đề xuất mới. Cảnh báo khẩn gửi tin riêng có tiền tố `[KHẨN]`. Tin dài >4000 ký tự thì tách.
4. Lỗi của 1 tài khoản không làm dừng tài khoản khác; lỗi được nhắn Telegram.
5. 3 ngày đầu sau `go_live_date`: dùng ngưỡng chặt hơn (bên dưới), không sinh đề xuất `SET_DAILY_BUDGET` hay `PAUSE_KEYWORD` trừ khi do chính sách/link lỗi.

---

## 7. Rules (ngưỡng mặc định, chỉnh trong tab `Cau hinh`)

| Rule | Mức | Đầu ra |
|---|---|---|
| Ad Disapproved / Final URL lỗi / tài khoản hạn chế / spend = 0 khi chiến dịch bật (sau 2h làm việc) | Khẩn | Telegram ngay + đề xuất `PAUSE_AD` nếu là ad |
| Lead thật (Sheet) lệch conversion Ads >30% trong 2 ngày liên tiếp (khi mỗi bên ≥3) | Khẩn | Báo nghi lỗi tracking, đặt cờ `tracking_suspect` → chặn đề xuất tối ưu |
| Chi phí kể từ lead gần nhất > 2 × CPL mục tiêu (3 ngày đầu: 1,5 ×) | Khẩn | Báo kiểm tra form/landing |
| Pacing lũy kế tháng ngoài 90–110% | Cao | Đề xuất `SET_DAILY_BUDGET` = (Net Media − đã chi) / ngày còn lại, chia theo tỷ trọng kế hoạch |
| Truy vấn khớp nhóm từ phủ định rõ ràng | Cao | Đề xuất `ADD_NEGATIVE` (phrase) |
| Truy vấn chi phí ≥ 1 × CPL mục tiêu, 0 conversion, không chứa tên sản phẩm/thương hiệu khách | Cao | Đề xuất `ADD_NEGATIVE` (exact) kèm số liệu |
| Keyword chi phí ≥ 2 × CPL mục tiêu, 0 conversion, ≥7 ngày dữ liệu | Cao | Đề xuất `PAUSE_KEYWORD` |
| CPC > 130% trung bình 7 ngày, hoặc CPL 3 ngày > 150% mục tiêu | Cao | Ghi danh sách cho phiên tối ưu tuần |
| CTR Search dưới cận dưới KPI 3 ngày liền | Thường | Ghi danh sách cho phiên tuần (viết lại RSA) |

Khi cờ `tracking_suspect` bật: chỉ sinh đề xuất liên quan an toàn (pause ad lỗi), không sinh đề xuất tối ưu khác.

---

## 8. Mẫu báo cáo và Sheet

### 8.1 Tin Telegram 9h (mỗi khách)
```
[<TÊN KHÁCH>] Báo cáo ngày dd/mm
Chi tiêu: x đ (kế hoạch y đ) | Lũy kế tháng: z% kế hoạch
Click a | CTR b% | CPC c đ
Lead: Ads d / Sheet e / hợp lệ f | CPL g đ (mục tiêu h đ)
So với hôm trước: lead ±, CPL ±%
Cảnh báo: ... (nếu có)
Đề xuất chờ duyệt: n -> <link tab De xuat>
```

### 8.2 Các tab Sheet khách
- `Cau hinh`: key | value (budget_gross, fee_pct, fee_base, net_media, period_start, period_end, kpi_ctr_min, kpi_cpl_target, kpi_qualified_rate, campaign_split JSON, approved_offers, negative_groups, thresholds..., automation_level).
- `Ke hoach`: chiến dịch, tỷ trọng, media tháng, daily budget, CPC/CTR/CVR dự kiến, click, lead, CPL; 3 kịch bản; dòng `approved` (người, ngày).
- `Bao cao ngay`: date, cost, impr, clicks, ctr, cpc, conv_ads, leads_sheet, leads_qualified, cpl, cpql, pacing_mtd, ghi chú.
- `Bao cao tuan` / `Bao cao thang`: khối Tổng quan (kỳ này, kỳ trước, chênh lệch %, kế hoạch), khối theo chiến dịch, theo nhóm sản phẩm, theo khu vực, theo thiết bị; ô **Nhận xét** (Claude điền); ô **Kế hoạch kỳ tới** (Claude điền). Tháng thêm: chi phí gồm phí, CPQL, CAC/ROAS nếu có. Không chứa đề xuất nội bộ để gửi được cho khách.
- `De xuat`: id, created_at, nguon (Script/Claude), loai, cap, doi_tuong, gia_tri_hien_tai, gia_tri_moi, ly_do, so_lieu, rui_ro, Duyet (checkbox), trang_thai (Cho duyet/Da ap dung/Loi/Tu choi/Het han/Can Editor), applied_at, ghi_chu.
- `Nhat ky`: thời gian, loại, đối tượng, trước, sau, lý do, nguồn, duyệt bởi, áp dụng bởi.
- `Tu khoa tim kiem`: search_term, campaign, ad_group, clicks, cost, conv, lan_dau_thay, quyet_dinh (KEEP/ADD/MONITOR/EXCLUDE), ly_do, nguoi_quyet.

---

## 9. Thứ tự dựng (mỗi bước: báo kế hoạch → người dùng duyệt → làm → kiểm thử → tóm tắt tiếng Việt)

**Bước 1 – Theo dõi + báo cáo tự động (2–3 ngày)**
1. Repo, `CLAUDE.md`, `README.md`, `knowledge/` (mục 4), `templates/`.
2. Lệnh `/cai-dat`: hướng dẫn người dùng tạo bot Telegram, Sheet tổng, Sheet mẫu, service account.
3. `ads_scripts/agent_mcc.js` đủ mục 6–8. Bắt đầu với `DRY_RUN=true`.
4. Kiểm thử chấp nhận trên 1 tài khoản đang chạy: tin 9h đúng số với giao diện Google Ads (lệch ≤1%); bản 16H chỉ nhắn khi có vấn đề; tạo 1 đề xuất phủ định thử, tick duyệt, lần chạy sau áp dụng đúng và ghi nhật ký; sau đó `DRY_RUN=false`.

**Bước 2 – Lập kế hoạch + set camp (2–3 ngày)**: skill 1, 2 và scripts Python liên quan. Kiểm thử bằng một brief mẫu do Claude tự tạo: ra đủ kế hoạch; CSV nhập Editor không lỗi; QA bắt được lỗi cố ý cài (sai ngân sách, thiếu negative, headline >30 ký tự).

**Bước 3 – Tối ưu tuần + Mức 1 (1 tuần)**: skill 3, 4; bật allowlist Mức 1 theo từng khách khi người dùng yêu cầu. Kiểm thử: 2 tuần không có thay đổi ngoài ý muốn.

---

## 10. Việc chỉ con người làm (Claude Code hướng dẫn từng bước khi tới lúc)
- Tạo tài khoản GitHub, mở Claude Code trên web, tạo repo `gads-agent` (private).
- Tạo bot Telegram qua @BotFather, gửi 1 tin cho bot, lấy chat_id (Claude hướng dẫn mở link getUpdates).
- Tạo Google Cloud project, bật Google Sheets API + Drive API, tạo service account, tải khóa JSON, dán vào phần secret/biến môi trường của Claude Code.
- Share Sheet tổng và Sheet khách cho email service account (Editor).
- Đảm bảo MCC đặt múi giờ GMT+7. Dán script vào MCC: Công cụ → Hành động hàng loạt → Tập lệnh → dấu + → dán → Ủy quyền → Xem trước → Lưu → đặt lịch Hằng ngày 9:00 (bản 16H: 16:00).
- Xuất CSV Keyword Planner cho mỗi khách mới.
- Nhờ web dev gắn tracking theo hướng dẫn Agent viết; làm phiếu test 5 mục.
- Nhập CSV vào Google Ads Editor, xem lại, Đăng; bật chiến dịch khi GO.
- Tick duyệt đề xuất; gửi báo cáo cho khách.

---

## 11. Mẫu `templates/client.yaml`

```yaml
client: "<tên khách>"
code: "<MAKHACH>"
customer_id: "<xxx-xxx-xxxx>"
business: {industry: "<ngành>", products: ["<sản phẩm>"], website: "<url>", usp: ["<điểm mạnh đã xác nhận>"]}
budget: {gross: 0, fee_pct: 0, fee_base: gross, period_days: 30, start_date: "YYYY-MM-DD"}
product_split: {}
locations: []
campaign_split: {}
kpi: {ctr_search: [null, null], cpl_target: null, qualified_rate: null}
lead_definition: "<định nghĩa lead hợp lệ>"
lead_source: {sheet_url: "", tab: "", date_col: "", status_col: "", qualified_values: []}
report_sheet_url: ""
primary_conversions: []
approved_offers: []        # chỉ ưu đãi trong danh sách này mới được viết vào quảng cáo
competitors: []
automation_level: 0
assumptions: []            # Agent tự ghi các giả định
```
