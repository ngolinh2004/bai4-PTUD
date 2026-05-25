# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421

Lớp: 58KTPM

**Bài tập 04:**  
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
- pull các images về và chạy chúng (up -d)
- Kiểm tra các service đã running ok (ko bị restart liên tục)
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn **Phát triển ứng dụng với mã nguồn mở**
- Truy cập sub-domain3 để cấu hình n8n:
  + tạo tài khoản admin : nhớ điền đúng email
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
  + Create workflow  (home page => overview => Create workflow)
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys
    + cần tạo project mới, sẽ lấy được API KEY
    + Nhập API Key lên giao diện n8n
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
    + Turn on Output Content as JSON : để kết quả trả về dạng json
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
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

  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
    + Ignore SSL Issues (Insecure): TURN ON
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger
   
+ Kết quả cuối cùng cần đặt được:
  + từ điện thoại, chát với telegram bot
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
  + f5 wordpress để thấy bài viết mới đã lên sóng.

+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được
+ Nhận xét thành quả đạt được!!!



# BÀI LÀM·
## 1.Tạo thư mục

  <img width="645" height="82" alt="image" src="https://github.com/user-attachments/assets/6938480f-3df6-4dfa-9842-3a784ec1914e" />

  
## 2.Tạo file

   <img width="940" height="457" alt="image" src="https://github.com/user-attachments/assets/b8f79cc9-9cd3-49e4-810b-e1dd8e0cc0bb" />

   <img width="940" height="459" alt="image" src="https://github.com/user-attachments/assets/ab6c9fb1-44b1-4ee1-b670-270afcf12bff" />


## 3. Chạy docker
   <img width="940" height="181" alt="image" src="https://github.com/user-attachments/assets/f52dd0cb-12c9-4dc0-862d-c98ac7e9a174" />

## 4. Cấu hình Public hostname trên Cloudflare
   
  - WordPress
    
   <img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/6e04e1d0-e940-4e1a-82ed-118ba5827702" />
   
  - phpMyAdmin
 
   <img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/d2c6f7e5-c738-4ed1-9d3a-3730bd43a779" />

  - Tổng quan các routes/tunnels đã tạo

    <img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/6b2e3954-a143-47d2-bcb3-09fef3f8891e" />

## 5. Giao diện đăng nhập phpMyAdmin
   
   <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/25a1fc30-a660-4946-97ee-ecfed9379b69" />

## 6. Cài đặt WordPress

  - Cài ngôn ngữ
    
   <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/a221e531-8814-4e41-9980-958a2847d3b5" />
   
  - Điền thông tin quản trị viên

    <img width="940" height="504" alt="image" src="https://github.com/user-attachments/assets/d601c5b0-5e22-4105-99c4-aab189c85df8" />

  - Giao diện trang chủ blog

    <img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/e3604a58-93fe-42d2-a248-ec1ffff97616" />

  - Giao diện trang quản trị

    <img width="940" height="506" alt="image" src="https://github.com/user-attachments/assets/e8fee3e2-0857-4128-98f7-bee4ef30e23e" />

  - Bài viết tự động đăng lên WordPress "Người bạn tài giỏi của tôi - Hoàng Kim Ngọc

    <img width="940" height="495" alt="image" src="https://github.com/user-attachments/assets/9fc31450-3751-4d21-9059-16ba35069561" />

## 7. Khởi tạo và Kích hoạt n8n
    
    <img width="940" height="517" alt="image" src="https://github.com/user-attachments/assets/0a79de80-b9d9-44f1-b6a5-ac186d321fc0" />

    
    <img width="940" height="466" alt="image" src="https://github.com/user-attachments/assets/e6879997-2a62-47ba-8891-bbb8b8622847" />


## 8. Thiết lập Workflow - Telegram & Gemini

  <img width="654" height="1454" alt="image" src="https://github.com/user-attachments/assets/2c12674a-b0fe-45e8-8345-17f462f4d741" />

  <img width="654" height="1454" alt="image" src="https://github.com/user-attachments/assets/c11b0cbe-cd17-401e-a62e-938816414b7e" />

  <img width="940" height="464" alt="image" src="https://github.com/user-attachments/assets/827699e0-dc27-4ed4-8e23-6505f2e1f2a2" />

  <img width="940" height="380" alt="image" src="https://github.com/user-attachments/assets/2d518ef5-b5e4-499c-a8cf-6e81ef703865" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/c8c9a16a-aaad-4af6-84b0-972c9958c8b5" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/b1b17b7b-d3d0-4f88-8236-e6a92e59d245" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/7c231779-3a07-49a1-a58c-ea80f761f1f5" />

## 9. Thiết lập Workflow - Code JS & WordPress

  <img width="940" height="481" alt="image" src="https://github.com/user-attachments/assets/5d89c2bc-decf-490f-ae9a-0b6a296e85fa" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/f8c9da3f-bc65-4220-a77c-c410b7668dd4" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/6053d923-dd00-448b-8201-ce53954dbde6" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/f1431b81-4f26-492b-8b0b-ed618072c2b3" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/c238249a-c570-46ba-ae63-09d705881437" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/d454f1fd-3e54-4165-9c2b-bd80c46c94e2" />

## 10. Chạy thử nghiệm và Kết quả

  <img width="720" height="1600" alt="image" src="https://github.com/user-attachments/assets/da945b60-3403-42f0-bee7-054d724d2286" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/1907d525-c8f3-4f89-9a37-92b3be51f8e8" />

  <img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/727d5f22-5bd0-46c0-a1b2-6441d39005ba" />


## Nhận xét thành quả đạt được
  - Trong quá trình thực hiện bài tập, em đã triển khai thành công hệ thống WordPress chạy bằng Docker Compose kết hợp với MariaDB, PhpMyAdmin, Cloudflare Tunnel và n8n. Sau khi hoàn tất cấu hình, các service đều hoạt động ổn định và có thể truy cập từ internet thông qua các sub-domain đã cấu hình trên Cloudflare.

  - Em đã cài đặt WordPress thành công, kiểm tra cơ sở dữ liệu bằng PhpMyAdmin và tạo được các bài viết theo yêu cầu. Bên cạnh đó, em cũng cấu hình workflow trên n8n để tự động hóa quá trình đăng bài từ Telegram lên WordPress thông qua Google Gemini AI. Hệ thống có thể nhận nội dung từ Telegram Bot, gửi prompt đến Gemini AI để tạo bài viết HTML, xử lý dữ liệu JSON bằng JavaScript rồi tự động publish bài viết lên website WordPress.

  - Trong quá trình thực hiện, em gặp một số khó khăn như lỗi container restart liên tục do sai biến môi trường, lỗi Cloudflare Tunnel chưa public đúng service, lỗi Telegram Trigger không nhận tin nhắn vì chưa chat bot lần đầu và lỗi AI trả về dữ liệu JSON không đúng định dạng. Ngoài ra còn có lỗi credential của WordPress và Gemini bị sai khiến workflow không hoạt động đúng. Sau khi kiểm tra log container, chỉnh sửa workflow và cấu hình lại credential, em đã khắc phục được các lỗi và hệ thống hoạt động bình thường.

  - Qua bài tập này, em học được cách sử dụng Docker để triển khai nhiều service cùng lúc, cách public ứng dụng bằng Cloudflare Tunnel, sử dụng n8n để xây dựng workflow automation và tích hợp AI vào hệ thống thực tế. Đây là một bài tập giúp em hiểu rõ hơn về tự động hóa quy trình và cách kết nối nhiều nền tảng khác nhau để tạo thành một hệ thống hoàn chỉnh.








