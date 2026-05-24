# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
## Họ tên: Vũ Việt Anh
## Lớp: K58KTP.K01
## MSSV: K225480106082
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS

# BÀI LÀM
# SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:
## 1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file docker-compose.yml chứa:
### Truy cập vào: `docker-compose.yml` thay `n8n-project.vva11004.id.vn` bằng subdomain N8N bạn sẽ đặt trên Cloudflare
```
services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_db
    restart: always
    environment:
      TZ: "Asia/Ho_Chi_Minh"
      MARIADB_ROOT_PASSWORD: rootpassword123
      MARIADB_DATABASE: wordpress_db
      MARIADB_USER: wp_user
      MARIADB_PASSWORD: wp_password123
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - wp_network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin_ui
    restart: always
    depends_on:
      - mariadb
    environment:
      PMA_HOST: mariadb
      PMA_ARBITRARY: 1
    ports:
      - "8081:80"
    networks:
      - wp_network

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    depends_on:
      - mariadb
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password123
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8082:80"
    networks:
      - wp_network

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared_tunnel
    restart: always
    command: tunnel --no-autoupdate run --token eyJhIjoiMGUwYWMxNjZhOTQzOTIyZmEzZjM0NDE2MDkyMDViNTEiLCJ0IjoiZjZlZTRhMTQtNThlOC00ODE3LWJmYmEtMDk4MzExM2Q2ZjY5IiwicyI6Ik0yUm1PRFEwWkRndFptTTBPUzAwTldJMExUaGtZelV0WXpjd1lUWmtaVFEyTVRJMyJ9
    depends_on:
      - wordpress
    networks:
      - wp_network

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_app
    restart: always
    environment:
      WEBHOOK_URL: https://n8n-project.vva11004.id.vn
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - wp_network

volumes:
  mariadb_data:
  wordpress_data:
  n8n_data:

networks:
  wp_network:
    driver: bridge
```

## 2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml:
### Thêm route Cloudflare cho N8N và phpMyAdmin
- Vào Cloudflare Dashboard → Zero Trust → Networks → Connectors → nhấn vào vuvietanh_wordpress → tab Published application routes → Add a published application route
  + Thêm route N8N:
<img width="1919" height="908" alt="image" src="https://github.com/user-attachments/assets/0abece0c-8ef2-49b6-9dbc-dd9ab8e508d3" />

  + Thêm route cho phpMyAdmin:
<img width="1919" height="908" alt="image" src="https://github.com/user-attachments/assets/c6dbbef2-ce35-485b-bcf7-f5a90642bd89" />

<img width="1917" height="906" alt="image" src="https://github.com/user-attachments/assets/0137c938-179b-49a8-be6f-ec337f64d4cc" />

- Truy cập N8N vào: `https://n8n-project.vva11004.id.vn` rồi đăng kí tài khoản
<img width="1918" height="986" alt="image" src="https://github.com/user-attachments/assets/f73024df-efcd-4259-b494-542a7c71ed63" />

<img width="711" height="605" alt="image" src="https://github.com/user-attachments/assets/f9d06280-d1d0-48bf-ba47-fcfbed6e0e70" />

### Kích hoạt License Key
- Vào Settings (góc dưới trái) → Usage and plan → Enter activation key → dán key từ email vào → nhấn Activate
<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/b3e62c71-1fcb-4afa-8520-cde0ceb52f0c" />

### Tạo Telegram Bot
- Mở Telegram trên điện thoại hoặc máy tính
- Tìm kiếm `@BotFather` → nhấn vào → nhấn Start
- Gõ lệnh: `/newbot`
- BotFather sẽ hỏi tên bot → đặt tên: `VietAnh WordPress Bot`
- BotFather sẽ gửi lại Token:
```
8946647823:AAEavxJWSplIx28TTdH8W4A0klZc0CwIt8E
```
<img width="1919" height="982" alt="image" src="https://github.com/user-attachments/assets/95a1bc5a-a675-4204-90a4-473d2d06aec1" />

### Lấy Google Gemini API Key
- Vào: `https://aistudio.google.com/api-keys`
- Đăng nhập bằng tài khoản Google → "Create API Key" → chọn project hoặc tạo mới → copy API Key
<img width="1917" height="982" alt="image" src="https://github.com/user-attachments/assets/33f7ce4f-691d-401f-8a31-81824835c660" />

<img width="1919" height="909" alt="image" src="https://github.com/user-attachments/assets/5ec89abd-0714-472f-a951-0dd2004bc9b3" />

<img width="1919" height="911" alt="Screenshot 2026-05-24 232444" src="https://github.com/user-attachments/assets/449050ba-353e-49f9-940d-7702bd7f6de4" />

  + Key: `AIzaSyCxGMywEHakod6YfJcvw-U_WqN0uj0iuYc`

- Quay lại N8N: `https://n8n-project.vva11004.id.vn` -> Create workflow
- Thêm Node Telegram Trigger -> Nhấn vào ô `Add first step...` -> Ô tìm kiếm hiện ra -> gõ: `Telegram` -> chọn `On message`
<img width="1919" height="990" alt="image" src="https://github.com/user-attachments/assets/9f940960-79bb-4432-acc2-a718aa820c31" />

<img width="1839" height="872" alt="image" src="https://github.com/user-attachments/assets/e43c42f1-6c5d-458b-83f2-ff76c01baa38" />

- Nhấn `Set up credential` để thêm Telegram Bot Token
- Dán token vào ô Access Token: `8946647823:AAEavxJWSplIx28TTdH8W4A0klZc0CwIt8E`
<img width="1919" height="903" alt="image" src="https://github.com/user-attachments/assets/ec5c8227-7d07-4ebc-8293-bb194755fd11" />

- Sau khi Save xong -> nhấn `Test this trigger` -> qua Telegram nhắn -> cho bot `@vietanh_wp_bot` -> quay lại N8N xem OUTPUT bên phải có hiện data không
<img width="1919" height="905" alt="image" src="https://github.com/user-attachments/assets/03236823-7591-40cb-9568-f3a4a4ca6eac" />

<img width="1919" height="994" alt="image" src="https://github.com/user-attachments/assets/e2586008-9cd8-44e7-a605-83707caf0d24" />

### Thêm Node Google Gemini
<img width="1919" height="905" alt="image" src="https://github.com/user-attachments/assets/f659b8a1-2636-4222-8ef3-9abb01dec1ca" />

- Chọn `Google Gemini Chat Model` -> `Message a model`
<img width="1919" height="903" alt="image" src="https://github.com/user-attachments/assets/9691fe4d-813f-43a6-9c0a-2a6cb0c9ec90" />

- Nhấn `Set up credential` → dán API Key: `AIzaSyCxGMywEHakod6YfJcvw-U_WqN0uj0iuYc`
<img width="1919" height="908" alt="image" src="https://github.com/user-attachments/assets/4f0e51cc-ce25-4204-b76e-40b33452ba9e" />

### Cấu hình Prompt
```
Hãy đóng vai là một chuyên gia công nghệ, viết một bài viết chuẩn SEO, chi tiết và dài bằng tiếng Việt dựa trên yêu cầu sau: {{ $json.message.text }}. Trả về kết quả CHỈ dưới dạng JSON thuần túy (không có markdown, không có backtick) với đúng 2 trường: post_title và post_content (nội dung HTML có dùng thẻ h2, p, ul, li).
```
<img width="1918" height="902" alt="image" src="https://github.com/user-attachments/assets/809e2187-ffdd-49dc-ab45-fd03f0bbc35f" />

### Thêm Node Code in JavaScript
- Ở `Message a model` -> chọn dấu `+` rồi nhập `code` xong chọn ngôn ngữ Javascript
```
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```
<img width="1918" height="907" alt="image" src="https://github.com/user-attachments/assets/8d277634-af0f-4c5a-8222-0a90f41c2b99" />

### Thêm node WordPress
- Nhấn `+` bên phải node Code -> tìm kiếm: `WordPress` rồi chọn `Create a post`
<img width="1919" height="899" alt="image" src="https://github.com/user-attachments/assets/1d55f4f1-8880-440f-9308-05f557c5b385" />

- Nhấn `Set up credential` → điền:
  + WordPress URL: `https://wordpress-project.vva11004.id.vn`
  + Username: vuvietanh
  + Password: chuỗi 24 ký tự
<img width="1919" height="985" alt="image" src="https://github.com/user-attachments/assets/6a352bea-aef0-4de7-9386-b569d01f8bf7" />

```
wzEE ktCb pzJc IASR oYJF vInO
```

<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/6cde6547-f2d7-4106-8c7e-0697760b8d5e" />

<img width="1919" height="910" alt="image" src="https://github.com/user-attachments/assets/15356cf2-6c8c-498d-a03c-b0010c502a6b" />

### Kiểm tra kết quả cuối cùng
- Chát với telegram bot
<img width="1433" height="902" alt="image" src="https://github.com/user-attachments/assets/b6f1a958-c655-4546-abae-3dd0d34711bf" />

<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/dda01979-45a5-4024-a56f-c3ce33503235" />

<img width="1918" height="983" alt="image" src="https://github.com/user-attachments/assets/53e30518-a49b-4cd7-a929-de3474b47ea3" />

<img width="1919" height="989" alt="image" src="https://github.com/user-attachments/assets/55ab77d8-116b-44fd-908d-8b7822a8ac2e" />

### Nhận xét thành quả đạt được
- Sau khi hoàn thành bài tập, em đã xây dựng thành công một hệ thống tự động đăng bài lên WordPress thông qua việc kết hợp các công nghệ mã nguồn mở gồm Docker, N8N, Telegram Bot và Google Gemini AI.
- Kết quả đạt được cụ thể như sau: hệ thống Docker Compose chạy ổn định với 5 service gồm MariaDB, phpMyAdmin, WordPress, Cloudflared và N8N. Ba subdomain đã được cấu hình thành công trên Cloudflare Tunnel, cho phép truy cập WordPress, phpMyAdmin và N8N từ internet mà không cần VPS hay địa chỉ IP tĩnh.
- Workflow trong N8N hoạt động đúng theo yêu cầu: khi người dùng nhắn tin nội dung chủ đề cho Telegram Bot, hệ thống sẽ tự động gửi nội dung đó đến Google Gemini AI để tạo bài viết chuẩn SEO bằng tiếng Việt, sau đó node Code in JavaScript xử lý định dạng JSON trả về, và cuối cùng node WordPress tự động tạo bài viết với trạng thái Publish ngay lập tức.
- Qua quá trình thực hiện, em nhận thấy N8N là một công cụ automation mã nguồn mở rất mạnh mẽ, cho phép kết nối nhiều dịch vụ khác nhau mà không cần viết nhiều code. Việc tích hợp AI vào quy trình tạo nội dung giúp tiết kiệm rất nhiều thời gian và công sức. Tuy nhiên, quá trình cấu hình ban đầu khá phức tạp, đặc biệt là phần kết nối Cloudflare Tunnel và cấu hình credential cho từng node trong N8N.
- Nhìn chung, đây là một bài tập thực tế và bổ ích, giúp em hiểu rõ hơn về cách các hệ thống tự động hóa hiện đại hoạt động và cách ứng dụng AI vào công việc thực tiễn.








 
