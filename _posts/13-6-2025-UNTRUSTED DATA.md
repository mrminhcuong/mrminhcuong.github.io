# UNTRUSTED DATA

# 1. Untrusted Data  - Khởi nguồn của lỗi bảo mật

## 1.1. Untrusted Data là gì?

- Untrusted Data là dữ liệu có thể mang những mã tấn công hệ thống. Chỉ cần một cái lọt vào, hệ thống của chúng ta đã bị xâm nhập.
- Để định nghĩa được các Untrusted Data một cách chính xác, ta cần đặt mình vào vị trí Dev - người trực tiếp làm việc bên trong hệ thống:
    - Luôn cho rằng tất cả dữ liệu bên ngoài không đáng tin cậy, trừ khi kiểm soát nó.
    - Không chắc chắn kiểm soát được → Giả sử trường hợp tệ nhất

**❔ Tại sao các dữ liệu bên ngoài (External Data) là không đáng tin?**

![image.png](image.png)

- Hackers là tập con của Users, chỉ khác ở, đây là những người dùng tinh nghịch
- Dữ liệu Hackers và Users lẫn vào nhau → Buộc phải xem tất cả dữ liệu đi vào là không đáng tin cậy

⇒ Thực hiện xét nghiệm toàn bộ.

Đòi hỏi: Tỉ mỉ, khả năng phân tích và cái nhìn tổng quát về hệ thống.

❔ **Dưới góc nhìn của developer, đâu là Untrusted Data? Tại sao?**

- Gói tin HTTP request **là untrusted data** vì nội dung gói tin này có thể bị thay đổi bởi Hackers.
- Config của developer **không phải là untrusted data** vì developer kiểm soát được nội dung config của mình.
- Dữ liệu của người dùng **là untrusted data** vì có thể chứa các nội dung không mong muốn như payload tấn công đến hệ thống.
- Dữ liệu từ máy chủ khác là untrusted data vì dữ liệu được gửi có thể bị thay đổi trên môi trường mạng hoặc chưa được xửlý an toàn trước khi được gửi.

## 1.2. Rủi ro từ Untrusted Data

![image.png](image%201.png)

Trong mô hình web này, chúng ta cần bảo vệ:

- Database: nơi chứa dữ liệu người dùng: Họ tên, số điện thoại, thông tin tài khoản thanh toán, địa chỉ,…
- Web server Back-end: chứa mã nguồn → Tài sản, trí tuệ của doanh nghiệp. Trong mã nguồn chứa các dữ liệu quan trọng: Access token, mật khẩu kết nối cơ sở dữ liệu,…
    
    ⇒ Bị tấn công → bàn đạp giúp tin tặc tấn công vào bên trong thành phần khác mạng nội bộ.
    
- Web front-end: Hiển thị thông tin, nội dung người dùng tương tác: tin nhắn, thông tin tài khoản, thông tin cá nhân,… Một số dữ liệu quan trọng như HTTP cookie cũng được lưu phía trình duyệt.
- Innocent User (Quan trọng): Doanh nghiệp hoạt động nhờ có users. Người dùng có thể bị đánh cắp thông tin, phishing. Thông tin người dùng không chỉ có trong database, front-end hay back-end mà còn trong suy nghĩ, trí nhớ của users nữa.
    
    ⇒ Mục tiêu bảo vệ chính: Tài nguyên doanh nghiệp và users.
    
    **⇒ Bất kì đối tượng nào xử lý untrusted data đều có thể gặp rủi ro.**
    

## 1.3. Cách attacker tác động đến Web Server:

![image.png](image%202.png)

- Trong mô hình này, untrusted data là toàn bộ gói tin HTTP Request.
- Lưu ý: Trong các trường hợp phức tạp hơn, untrusted data có thể đến từ rất nhiều nguồn khác nhau.
- Thay vì dùng trình duyệt, Hackers có thể tác động vào Web Server bằng việc can thiệp vào các gói tin HTTP nguyên bản để đọc và chỉnh sửa nó tùy ý.
    
    ⇒ BurpSuite là công cụ dùng để kiểm thử xâm nhập web, hỗ trợ quan sát, phân tích và thay đổi nội dung gói tin HTTP nguyên bản.
    

## Write-up for lab

Một số untrusted data được tìm thấy:

1. User-Agent:
    
    Từng có trường hợp tấn công SQL Injection thông qua User-Agent.
    
2. Untrusted name and email
    
    Xuất hiện ở 2 get parameter *name* và *email* tại endpoint */register.php*
    
    ```html
    GET /register.php?name=test&email=test%[40gmail.com](http://40gmail.com/) HTTP/1.1
    ’’’
    ```
    
3. Referer
    
    Xuất hiện ở Header Referer
    
    ```html
    ...
    Referer: http://192.168.49.128:12000/register.php?name=test&email=test%40gmail.com
    ...
    ```
    
4. Hidden Path
    
    Xuất hiện tại một endpoint (path ẩn)
    
    ```html
    GET /test.php HTTP/1.1
    ```
    
5. Untrusted Sign-up (name/email)
    
    Xuất hiện ở 2 post parameter *username* và *password* tại endpoint */sign-up.php*
    
    ```html
    POST /sign-up.php
    ...
    username=test&password=test
    ```
    
6. Untrusted Sign-in (name/email)
    
    ```html
    POST /sign-in.php
    ...
    username=test&password=test
    ```
    
7. Hidden methods:
    
    Một trong các method sau ["OPTIONS", "TRACE", "HEAD", "PUT", "PATCH", "DELETE", "PATCH"]
    
    ```html
    OPTION /index.php
    ...
    ```
    
8. Profile:
    
    Xuất hiện ở các Content-Disposition: bio, filename và các parameter: Content-Type của filename, File
    Content
    
    ```html
    POST /profile.php HTTP/1.1
    ...
    ------WebKitFormBoundaryZURz268sjPl6o9Q5
    Content-Disposition: form-data; name="bio"
    test
    ------WebKitFormBoundaryZURz268sjPl6o9Q5
    Content-Disposition: form-data; name="file"; filename="test"
    Content-Type: test
    test
    ------WebKitFormBoundaryZURz268sjPl6o9Q5--
    ```
    
9. Hidden Header:
    
    GET / HTTP/1.1
    ...
    test: test
    
    Thêm một Header test
    
10. Hidden Parameter:
    
    Thêm tham số test vào gói GET nào cũng được
    
    ```html
    GET /?test=test HTTP/1.1
    ...
    ```
    
    Thêm tham số test vào gói POST nào cũng được
    
    ```html
    POST /sign-in.php HTTP/1.1
    ...
    username=test&password=test&test=test
    ```
    
11. Cookie
    
    Xuất hiện ở Header Cookie.
    
    ```html
    POST /sign-in.php HTTP/1.1
    ...
    Cookie: PHPSESSID=39b9706993a802a0b3c523f3a6fdf8e5;test=test
    ...
    ```
    
12. Track:
    
    Xuất hiện tại endpoint */track.php*
    
    ```html
    GET /track.php?id=test HTTP/1.1
    ...
    ```