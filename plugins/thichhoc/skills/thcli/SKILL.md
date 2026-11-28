---
name: thcli
description: Làm việc với các ứng dụng nội bộ Thích Học qua thcli (Thích Học CLI), thay cho MCP. share.thichhoc.com (chia sẻ file/thư mục, "gửi link", share tạm/mãi mãi), link.thichhoc.com (rút gọn link, thống kê lượt mở, quảng cáo trung gian), payment.thichhoc.com (thanh toán, hoàn tiền kèm biên lai, hoá đơn điện tử, đối soát sao kê), catalog.thichhoc.org (sản phẩm, giá, chiến dịch giảm giá, mã giảm giá, lô mã kích hoạt, báo cáo doanh thu), clio.thichhoc.org (module content; Content Studio: viết, duyệt, lên lịch, ghi nhận bài đăng mạng xã hội theo thương hiệu). Dùng khi được nhờ thao tác trên một trong các app này, hoặc khi lệnh thcli báo lỗi cần chẩn đoán.
---

# thcli

thcli gọi API `/api/thcli/v1` của từng app qua Cloudflare Access. Bạn có **đúng quyền của người
dùng** trên web: server quyết định từ email trong JWT của Access. Không có cách vượt quyền. Bị từ
chối (exit 4) thì nói cho người dùng biết, kèm vai trò còn thiếu nếu lỗi có ghi (`required_role`).

## Chuẩn bị

```sh
command -v thcli || npm i -g --install-links github:thichhoc-org/thcli
thcli list                      # module có sẵn / đã cài
thcli install <module>          # share | link | payment | catalog | content
thcli whoami <module>           # email và vai trò của bạn ở service đó
```

- **Exit 3 (chưa đăng nhập):** nhờ người dùng tự chạy `! thcli login <module>`. Lệnh này mở trình
  duyệt; bạn không tự đăng nhập thay họ được. Máy bật WARP thì thường không cần đăng nhập.
- **Lỗi lạ về cài đặt hay kết nối:** chạy `thcli doctor`.

## Quy ước

- **Output:** stdout là JSON (khi bị pipe). Lỗi ghi ra stderr dạng `{"error":{"code","message",…}}`.
- **Exit code:**

  | Exit | Nghĩa |
  |---|---|
  | 0 | Thành công |
  | 1 | Lỗi khác |
  | 2 | Sai tham số |
  | 3 | Chưa đăng nhập |
  | 4 | Không có quyền |
  | 5 | Không tìm thấy, hoặc module chưa cài |
  | 6 | Xung đột: version đổi, trạng thái không cho phép, token xác nhận hết hạn |
  | 7 | Quá giới hạn gọi |
  | 10 | Cần xác nhận |

- **Xác nhận hai bước (exit 10)** cho việc không hoàn tác được: xoá, thu hồi, kết thúc chiến dịch,
  hoàn tiền, đổi quyền…
  - Lần đầu, thcli in `{"stage":"preview","preview":{summary…},"confirm_token":…}`.
  - Cho người dùng xem `preview.summary` và chờ họ đồng ý, trừ khi họ đã yêu cầu rõ đúng việc đó.
  - Sau đó chạy lại **đúng lệnh ấy** với `--confirm <token>`. Nếu kết quả có trường `next`, chạy
    lệnh trong `next`.
  - Token hết hạn sau vài phút và gắn với đúng đầu vào ấy.
- **Xung đột version (exit 6, `version_conflict`):** đọc lại dữ liệu mới nhất, rồi làm lại. Không ép
  ghi đè.
- **Xem đủ lệnh:** `thcli <module> --help`, `thcli <module> <lệnh> --help`, hoặc
  `thcli <module> --json` để có mô tả máy đọc được.
- **Chưa có lệnh tiện thì gọi thẳng tool:**
  - `thcli <module> tools [tool]` xem danh sách tool và JSON Schema.
  - `thcli <module> call <tool> '<json>'`, hoặc `--field value`.
  - Lệnh nào cũng nhận `--data '<json>'` để thêm trường.
- **File:** tool trả `download_url` thì tải bằng `thcli <module> file get <url> -o <file>`;
  `-o -` ghi ra stdout.

## Việc thường gặp

### share

```sh
thcli share put ./bao-cao.pdf              # in link công khai, xoá sau 30 ngày (--forever: không hết hạn)
thcli share put ./ke-hoach.pdf --internal  # chỉ nhóm Nội bộ, không hết hạn (--temp: xoá sau 30 ngày)
thcli share put ./thu-muc --main index.html
thcli share ls · thcli share stats <link|mã> · thcli share download <link|mã> -o ./tai-ve
thcli share rm <link|mã>                   # 2 bước
```

Trang HTML gọi CSS, ảnh bằng đường dẫn tương đối thì share cả thư mục (giữ cấu trúc) với `--main`, đừng
share riêng file HTML. `put` từ chối file có vẻ chứa bí mật (khoá, .env…). Chỉ dùng `--force` khi người dùng xác nhận.

### link

```sh
thcli link create https://vi-du.vn/trang --slug sale --title "Giảm giá" --expires 2026-12-31
thcli link stats <slug> --days 7 · thcli link visits <slug> · thcli link update <slug> --url …
thcli link disable <slug>                  # link không bao giờ bị xoá
thcli link ad create --name … --title … --url … --image banner.png   # [admin]
```

### catalog

Vai trò: viewer, editor, accountant, admin. Số tiền nhận `199000`, `199.000`, `199k`, `2tr`.

```sh
thcli catalog overview · thcli catalog products -q "goi nam" · thcli catalog product get prd_…
thcli catalog product create demo --name "Gói năm" --code GOI-NAM --unit Gói --tax KCT --price 1.990.000
thcli catalog invoice approve prd_…        # kế toán; không tự duyệt đề xuất của mình
thcli catalog campaign create --name "Black Friday" --terms @the-le.md --percent 30 --max 300k \
  --start 2026-12-01 --end 2026-12-07 --code BLACKFRIDAY   # hoặc --unique cho mã riêng
thcli catalog campaign activate cmp_…      # 2 bước
thcli catalog campaign codes generate cmp_… 500 -o ma.csv
thcli catalog batch revoke bat_… 0-4,7     # 2 bước
thcli catalog report sales --from 2026-11-01 --to 2026-11-30
```

Mã giảm giá và mã kích hoạt là bí mật. Lưu ra file bằng `-o` rồi đưa đường dẫn cho người dùng; đừng
dán hàng loạt mã vào cuộc trò chuyện.

### payment

Vai trò: viewer, operator, refund_operator.

```sh
thcli payment tasks                        # việc đang chờ: hoàn tiền, webhook lỗi, đối soát, hoá đơn lỗi
thcli payment find <mã bất kỳ> · thcli payment payment get pay_…
thcli payment payment create demo 390k "Luyện đề Lý 11" --order-id KH-1 --email a@b.vn
thcli payment refunds --status pending · thcli payment refund get rf_…
thcli payment refund confirm rf_… --proof bien-lai.png --bank-ref FT…
#   → exit 10, rồi chạy lệnh trong trường "next" (dùng --proof-id, không tải lại ảnh)
thcli payment invoice pdf inv_… -o hoa-don.pdf
thcli payment recon statement <account_id> sao-ke.csv --format mbbank
```

Hoàn tiền là chuyển tiền thật. Chỉ xác nhận khi người dùng đưa biên lai và bảo làm.

### content (Clio)

Bạn hành động **với tư cách người dùng**, không phải AI: duyệt được nếu họ có vai trò duyệt.

```sh
thcli content work                            # việc của tôi (canReview/canEdit)
thcli content context <brandId>               # đọc brand kit + playbook trước khi viết
thcli content story create <brandId> --file bai.json [--submit]   # schema: thcli content tools create_story
thcli content upload <brandId> anh.png --story <storyId>
thcli content variant update <variantId> --story <storyId> --caption @caption.md
thcli content submit|approve|reject|request-changes <variantId>@<version> …   # hoặc --story để tự lấy version
thcli content schedule <brandId> <variantId>=2026-12-10T12:00:00Z
thcli content published <brandId> <variantId> <url bài đăng>
thcli content mentions                        # bình luận đang gắn AI
thcli content brand get <brandId> --dir kit && thcli content brand set <brandId> --version N --playbook kit/playbook.md
```

## Chẩn đoán

| Hiện tượng | Làm gì |
|---|---|
| exit 3 | Người dùng chạy `! thcli login <module>`, hoặc bật WARP |
| exit 4 | Thiếu vai trò. Báo người dùng; người quản trị của app đó mới cấp được |
| exit 5 "chưa cài" | `thcli install <module>` |
| Lỗi mạng hoặc 5xx | `thcli doctor`; thử lại sau |
| Cần trỏ sang máy dev | `THCLI_<MODULE>_URL=http://localhost:<port> THCLI_DEV_EMAIL=<email>` (chỉ localhost) |
