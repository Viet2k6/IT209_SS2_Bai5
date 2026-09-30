# Bài 5: Quản lý quyền sở hữu thư mục Web

Tài liệu hướng dẫn cấu hình phân quyền thư mục web trên Linux, đảm bảo user quản trị (`devops`) có quyền đọc/ghi và web server (`Nginx` / `www-data`) có quyền đọc và phục vụ website an toàn.

---

## 1. Tạo user `devops`

Tạo user mới dành cho tác vụ quản trị và kiểm tra thông tin user:

```bash
sudo adduser devops
id devops
```

---

## 2. Tạo thư mục Web

Tạo cấu trúc thư mục chứa mã nguồn website:

```bash
sudo mkdir -p /var/www/ptit-web/html
```

Tạo file `index.html` mẫu:

```bash
echo '<h1>PTIT Web</h1>' | sudo tee /var/www/ptit-web/html/index.html
```

---

## 3. Cấu hình Nginx

Cấu hình virtual host cho website tại đường dẫn:

```bash
/etc/nginx/sites-available/ptit-web
```

Kiểm tra cú pháp cấu hình Nginx:

```bash
sudo nginx -t
```

Khởi động lại dịch vụ Nginx:

```bash
sudo service nginx restart
```

Kiểm tra phản hồi từ web server:

```bash
curl http://localhost
```

---

## 4. Phân quyền thư mục và file

Thay đổi owner (`devops`) và group (`www-data`):

```bash
sudo chown -R devops:www-data /var/www/ptit-web
```

Phân quyền cho toàn bộ thư mục (`750` - `rwxr-x---`):

```bash
sudo find /var/www/ptit-web -type d -exec chmod 750 {} \;
```

Phân quyền cho toàn bộ file (`640` - `rw-r-----`):

```bash
sudo find /var/www/ptit-web -type f -exec chmod 640 {} \;
```

**Tóm tắt kết quả phân quyền:**
- **Owner:Group:** `devops:www-data`
- **Thư mục:** `750`
- **File:** `640`

---

## 5. Kiểm tra quyền ghi

Chuyển sang phiên làm việc của user `devops`:

```bash
sudo -u devops bash
```

Kiểm tra user hiện tại:

```bash
whoami
```

*Kết quả:*
```text
devops
```

Ghi dữ liệu kiểm tra vào file `index.html` (thực hiện thành công không cần `sudo`):

```bash
echo "Update" >> /var/www/ptit-web/html/index.html
```

Kiểm tra lại nội dung website:

```bash
curl http://localhost
```

*Kết quả:*
```html
<h1>PTIT Web</h1>
Update
```

---

## Kết luận

Đã hoàn tất cấu hình quyền sở hữu và phân quyền bảo mật cho thư mục `/var/www/ptit-web/`:
- User `devops` có toàn quyền đọc/ghi mà không cần can thiệp quyền `root`/`sudo`.
- Nginx (chạy dưới group `www-data`) vẫn có đầy đủ quyền đọc để phục vụ website bình thường mà không có quyền chỉnh sửa/thực thi nguy hiểm.