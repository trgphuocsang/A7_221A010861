# Bài Tập Nâng Cao - Lab A7

- Họ và tên: Trương Phước Sang
- MSSV: 221A010861
- Môn học: Lập trình trên các thiết bị di động

---

## Các Bài Nâng Cao Đã Triển Khai

Ứng dụng Lab A7 (Android Permission Model) đã được mở rộng và tích hợp thêm các tính năng nâng cao sau:

---

### 1. NC3: Thông báo điều hướng mở lại ứng dụng (Notification Intent)

- Mô tả:
    - Bổ sung khả năng điều hướng thông minh cho tính năng gửi thông báo hệ thống (POST_NOTIFICATIONS).
    - Khi ứng dụng phát thông báo thành công lên thanh trạng thái của thiết bị, người dùng chỉ cần chạm/click vào thông báo đó thì hệ thống sẽ tự động đưa ứng dụng trở lại màn hình chính (MainActivity) ngay lập tức, đồng thời tự động đóng thông báo.

- Kỹ thuật:
    - Khai báo một Intent dẫn tới MainActivity kết hợp với các cờ điều hướng (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK) để khôi phục hoặc làm mới Task chứa ứng dụng.
    - Đóng gói Intent thành PendingIntent với cờ PendingIntent.FLAG_IMMUTABLE nhằm đáp ứng đúng tiêu chuẩn bảo mật bắt buộc từ Android 12 (API 31) và Android 13 (API 33)+.
    - Gắn PendingIntent vào bộ dựng thông báo thông qua phương thức .setContentIntent(pendingIntent) và thiết lập .setAutoCancel(true):

  Intent intent = new Intent(this, MainActivity.class);
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);

  PendingIntent pendingIntent = PendingIntent.getActivity(
  this,
  0,
  intent,
  PendingIntent.FLAG_IMMUTABLE
  );

  NotificationCompat.Builder b = new NotificationCompat.Builder(this, CHANNEL_ID)
  .setSmallIcon(android.R.drawable.ic_dialog_info)
  .setContentTitle(getString(R.string.noti_title))
  .setContentText(getString(R.string.noti_text))
  .setPriority(NotificationCompat.PRIORITY_DEFAULT)
  .setContentIntent(pendingIntent)
  .setAutoCancel(true);

---

### 2. NC4: Sơ đồ luồng xử lý quyền (Permission Flowchart)

- Mô tả:
    - Trực quan hóa toàn bộ quy trình kiểm tra, xin quyền nguy hiểm (CAMERA) và xử lý các kịch bản phản hồi của người dùng theo đúng chuẩn 3 nhánh của Android Permission Model bằng sơ đồ khối.
    - Sơ đồ giúp minh họa rõ ràng cơ chế phân nhánh: đã có quyền, từ chối lần đầu (hiển thị lý do giải thích Rationale), và từ chối vĩnh viễn (hướng dẫn mở App Settings để cấp thủ công).

- Kỹ thuật:
    - Sử dụng ngôn ngữ mô tả sơ đồ khối Mermaid Flowchart để nhúng trực tiếp và render tự động sơ đồ trên các nền tảng báo cáo:

  graph TD
  A[Bắt đầu: Bam nut Chup anh] --> B{Kiem tra quyen CAMERA}
  B -- Da cap --> C[Mo Camera chup anh]
  B -- Chua cap --> D{Da tung tu choi?}
  D -- Dung --> E[Hien thi Dialog giai thich]
  E --> F[Xin quyen he thong]
  D -- Sai --> G[Goi Launcher xin quyen]
  F --> H{Ket qua}
  G --> H
  H -- Dong y --> C
  H -- Tu choi --> I[Hien thi trang thai Chua cap]
  D -- Bi chan vinh vien --> J[Mo Cai dat thu cong]
  J --> K[onResume cap nhat Da cap]

    - Mô tả chi tiết 3 nhánh chính:
        - Nhánh đã cấp quyền (PERMISSION_GRANTED): Kích hoạt trực tiếp chupAnh.launch(null) để chụp ảnh xem trước.
        - Nhánh từ chối tạm thời (shouldShowRequestPermissionRationale = true): Hiển thị AlertDialog giải thích lý do trước khi gọi xinQuyenCamera.launch(...).
        - Nhánh từ chối vĩnh viễn: Hiển thị AlertDialog hướng dẫn sang giao diện Cài đặt (Settings) hệ thống, sau đó cập nhật lại giao diện ứng dụng khi quay lại thông qua vòng đời onResume().