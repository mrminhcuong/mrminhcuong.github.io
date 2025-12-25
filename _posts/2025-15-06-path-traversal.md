---
title: "Cyber Jutsu: Web Pentest 101 - Path Traversal"
date: 2025-06-15
categories: [cyber-jutsu, writeups]
tags: [web-pentest-101]
---

# PATH TRAVERSAL VULNERABILITIES

![](/assets/img/posts/path_travesal/image.png)

# PATH là gì?

Path là đường dẫn trong thực tế là đường đi từ nơi này đến nơi khác.

![](/assets/img/posts/path_travesal/image1.png)

Còn trên máy tính đó chính là đường dẫn để ta truy cấp đến những file, thư mục được lưu trên máy tính, lấy ví dụ hệ điều hành Linux:

![](/assets/img/posts/path_travesal/image2.png)

Nếu ngoài đường, chúng ta cần xe để di chuyển, thì trong máy tính chúng ta cũng có câu lệnh `cd`

![](/assets/img/posts/path_travesal/image3.png)

Có một cách khác để di chuyển là dấu chấm `(.)` : Dấu chấm tượng trưng cho đường dẫn đến thư mục hiện tại.

## Absolute Path và Relative Path

Ví dụ như sau:

![](/assets/img/posts/path_travesal/image4.png)

- Nếu không phản sinh viên Trường Đại học Luật → sẽ không biết phòng 404 ở đâu. Lúc này, cần Absolute Path để chỉ dẫn địa chỉ cụ thể, cho dù có ở vị trí nào trên thế giới cũng đều có thể đến được
- Nếu đang học tại trường ĐH Luật, thì chỉ cần nói là Phòng 404 thì sẽ biết được vị trí tổ chức đó ở đâu.

Trên máy tính:

![](/assets/img/posts/path_travesal/image5.png)

- Đường dẫn trên máy tính cũng được chia làm 2 loại:
    - Absolute Path (đường dẫn tuyệt đối): đường dẫn bắt đầu từ thư mục gốc và bất kì đâu đều có thể đến được.
    - Relative Path (đường dẫn tương đối): đường dẫn phụ thuộc vào vị trí hiện tại.

Trong hệ điều hành Linux quy ước:

- Dấu chấm (`.`): Tượng trung cho đường dẫn đến thư mục hiện tại
- 2 dấu chấm (`..`): Tượng trung cho đường dẫn đến thư mục cha (parent directory).
    
    ![](/assets/img/posts/path_travesal/image6.png)
    

## Đường dẫn xuất hiện ở đâu trong thế giới Web?

![](/assets/img/posts/path_travesal/image7.png)

Ta thấy $_GET[’file_name’] là Untrusted Data được lưu vào biến `$file_name`, sau đó được cộng chuỗi với `‘/var/www/html/image/’`

Sẽ ra sao nếu ta tác động vào file name này?

![](/assets/img/posts/path_travesal/image8.png)

- Nếu như ta gán biến `$file_name='../../../../'` thì giá trị của biến `$path_name='/var/www/html/images/../../../../’`

⇒ Ta đã “quay xe 4 lần” và quay về thư mục gốc.

![](/assets/img/posts/path_travesal/image9.png)

- Vậy nếu ta gán biến `$file_name="../../../../etc/passwd"`có phải là lúc này ta đã truy cập đến được `file/etc/passwd`
- File `/etc/passwd` là một file mặc định mf bất kì hệ điều hành Linux nào cũng có.

## Lab 1:

Goal: Đọc nội dung /etc/passwd.

Link challenge: [http://pathtraversal.cyberjutsu-lab.tech:8091](http://pathtraversal.cyberjutsu-lab.tech:8091/)

![](/assets/img/posts/path_travesal/image10.png)

```php
// file: loadimage.php
<?php 
$file_name = $_GET['file_name'];
$file_path = '/var/www/html/images/' . $file_name; //tao ra absolute path hoan chinh
if (file_exists($file_path)) {
    header('Content-Type: image/png');
    readfile($file_path);
}
else { // Image file not found
    echo " 404 Not Found";
}
```

- Đoạn code này tạo ra một `$file_path` bằng cách nối chuỗi `'/var/www/html/images/'` với giá trị param `file_name`, sau đó kiểm tra xem có tồn tại file này không. Nếu có, đọc file đó với hàm `readfile()`. Ngược lại, in ra "404 Not Found".
- $_GET['file_name'] là một untrusted data nhưng anh developer của đoạn code không hề có biện pháp phòng chống nào và khiến ứng dụng bị dính lỗi path traversal.

Cách khai thác: Đích cần đến là `/etc/passwd`

- Ta có thể sử dụng `../` để di chuyển đến thư mục cha của thư mục hiện tại.
    
    → Path mà chúng ta cần đến được đích là`/var/www/html/image/../../../../etc/passwd`
    
    và giá trị của param `file_name` là `../../../../etc/passwd.`
    

KẾT QUẢ:

![](/assets/img/posts/path_travesal/image11.png)

Sử dụng BurpSuit hoặc view source để đọc flag:

![](/assets/img/posts/path_travesal/image12.png)

Quá trình untrusted data rơi vào nguy hiểm:

![](/assets/img/posts/path_travesal/image13.png)

⚠️ readfile là một hàm có mức độ nguy hiểm khá cao! Thuật ngữ gọi là những hàm unsafe method. Khi nó cho phép đọc toàn bộ nội dung file của tham số đường dẫn được đưa vào

→ Mà untrusted data đang rơi vào tham số của đường dẫn của nó

→ Cuối cùng, tận dụng 4 nguyên tắc về đường dẫn để “thao túng” và chuyển hướng đường dẫn $file_path sang một tập tin khác mà attacker muốn!

❔ Giả sử mình không biết mã nguồn → Không biết bao nhiêu thư mục. Làm sao để biết có bao nhiêu `../` thì về root?

## Lab 2:

GOAL: Đọc nội dung /etc/passwd

 File `loadimage.php`:

```php
<?php 
$file = $_GET['file'];
if (strpos($file, "..") !== false)
    die("Hack detected");
if (file_exists($file)) {
    header('Content-Type: image/png');
    readfile($file);
}
else { // Image file not found
    echo " 404 Not Found";
}?>
```

So sánh với Lab 1:

![](/assets/img/posts/path_travesal/image14.png)
- Ở level 2, anh dev đã filter ký tự `..` và ta không thể sử dụng cách path traversal như đã làm ở level 1.
- Tuy nhiên, ở level 2 không có phần prefix `$file_path` và ta có thể sử dụng hàm `readfile()` với dạng absolute path `etc/passwd` bình thường.

→ Cách khai thác: 

- Giá trị của param file_name là `/etc/passwd`.
- Kết quả:

![](/assets/img/posts/path_travesal/image15.png)

- Sử dụng Burp Suite để đọc flag

### Level 3:

Goal: Chiếm quyền điều khiển server và đọc một tập tin bí mật ở thư mục gốc.

Source Code:

```php
<?php

    // Create store place for each user (we place this in /var/www/html/upload for easily handle)
    session_start(); //set cookie PHPSESSID = ....
    if (!isset($_SESSION['dir'])) {

        $_SESSION['dir'] = '/var/www/html/upload/' . bin2hex(random_bytes(16));
    } // /var/www/html/upload/<32_ky_tu_random>
    $dir = $_SESSION['dir'];

    if ( !file_exists($dir) )
        mkdir($dir);

    if(isset($_FILES["files"]) && $_POST['album'] !="" ) {
        try {

            //Create Album
            $album = $dir . "/" . strtolower($_POST['album']); 
            // --> /var/www/html/upload/<32_ky_tu_random>/<album>
            // Dấu [] trong khai báo biến ở php -> Ý nói đây là một array -> upload nhieu file mot luc
            if ( !file_exists($album))
                mkdir($album);

            //Count Files: Dem xem upload bao nhieu file
            $files = $_FILES['files'];
            $count = count($files["name"]);
            
            // Save files to user's directory
            for ($i = 0; $i < $count; $i++) {
                
                $newFile = $album . "/" . $files["name"][$i]; // -> /var/www/html/upload/<32_ky_tu_random>/<album>

                move_uploaded_file($files["tmp_name"][$i], $newFile);
            }

       } catch(Exception $e) {
            $error = $e->getMessage();
         }
    }
?>
```

Apache2 config:

```
# This is the main Apache server configuration file.
DefaultRuntimeDir ${APACHE_RUN_DIR}

PidFile ${APACHE_PID_FILE}

Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

HostnameLookups Off

IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

Include ports.conf

<Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
</Directory>

<Directory /usr/share>
        AllowOverride None
        Require all granted
</Directory>

<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

# CHANGELOG: disable execution of php code in upload folder and safely return content-type
<Directory "/var/www/html/upload/">
        AllowOverride None
        Require all granted

        <FilesMatch ".*">
                SetHandler None
        </FilesMatch>

        Header set Content-Type application/octet-stream

        <FilesMatch ".+\.jpg$">
                Header set Content-Type image/jpeg
        </FilesMatch>
        <FilesMatch ".+\.png$">
                Header set Content-Type image/png
        </FilesMatch>
        <FilesMatch ".+\.(html|txt|php)">
                Header set Content-Type text/plain
        </FilesMatch>
</Directory>

AccessFileName .htaccess

<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>

ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn

LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

IncludeOptional conf-enabled/*.conf

IncludeOptional sites-enabled/*.conf
```

Giao diện: Đây là một website cho phép upload album và xem ảnh.

![](/assets/img/posts/path_travesal/image16.png)

Sau khi phân tích source code Level 3, ta có được các mảnh ghép sau:

```php
 $newFile = $album . "/" . $files["name"][$i];

 move_uploaded_file($files["tmp_name"][$i], $newFile);
```

→ Cho phép user upload tập tin vào thư mục $album và không hề có lớp sàng lọc filter nào.

🚧 Tuy nhiên lại không thể nào chạy được mod-php ở trong folder `/var/www/html/upload/` vì đã bị block bằng các config của apache2.conf

```php
<FilesMatch ".*">
      SetHandler None
</FilesMatch>
```

→ Hướng khai thác: Tìm cách nào nó nhảy ra ngoài `/var/www/html/` để thoát khỏi config block mod-php và thực thi được file php.

```php
$album = $dir . "/" . strtolower($_POST['album']);
```

**Untrusted Data:** cho phép kiểm soát đường dẫn $album bằng việc khai thác Path Traversal

→ Thao túng biến album

![](/assets/img/posts/path_travesal/image17.png)

**Đặt giả thuyết:**

- Kiểm tra thử xem liệu chúng ta có thể upload và thực thi file php hay không bằng cách tạo một file tên `test.php` với nội dung là `<?php phpinfo(); ?>` và upload lên website.
- Upload thành công nhưng file test.php không được thực thi mà hiển thị dưới dạng text.
    
    ![](/assets/img/posts/path_travesal/image18.png)
    

→ Nguyên nhân: Do anh developer đã cấu hình trong file apache2.conf mặc định không xử lí cho tất cả các file nằm trong đường dẫn `/var/www/html/upload/` .Một số ngoại lệ như các file .jpg, .png được hiển thị dạng ảnh, các file .html, .txt, .php hiển thị dạng text.

![](/assets/img/posts/path_travesal/image19.png)

- Liệu có thể upload vào thư mục khác có khả năng thực thi code php hay không? Cụ thể là **DocumentRoot,** nơi thực thi được file index.php.

Đọc source code của index.php

- Đoạn code tạo dir từ dòng 5 đến dòng 11:
    
    ![](/assets/img/posts/path_travesal/image20.png)
    
    $dir có giá trị `'/var/www/html/upload/' . bin2hex(random_bytes(16))` và ta không thể kiểm soát được giá trị này.
    
- Đoạn code tạo album từ dòng 17 đến dòng 19:
    
    ![](/assets/img/posts/path_travesal/image21.png)
    
    $album có giá trị `$dir . "/" . strtolower($_POST['album'])` 
    
    → Ta có thể kiểm soát giá trị này thông qua unstrusted data `$_POST['album'].`
    
- Đoạn code save file từ dòng 26 đến dòng 31:
    
    ![](/assets/img/posts/path_travesal/image22.png)
    

Unsafe method ở đây là hàm `move_uploaded_file($files["tmp_name"][$i], $newFile)` sẽ upload file từ `$files["tmp_name"][$i]` vào đường dẫn `$newFile` trên server mà ta có thể kiểm soát được biến `$album` nên có thể điều hướng file upload vào **DocumentRoot.**

**Hướng khai thác:**

- Đường dẫn file mà ta upload lên có dạng:
`'/var/www/html/upload/16_random_bytes/{album}/{file}'`
- Đích: `'/var/www/html/{file}'`

→ Để đi đến đích, giá trị `$_POST['album']` cần truyền vào là `‘../..’`

![](/assets/img/posts/path_travesal/image23.png)

Kết quả: 

![](/assets/img/posts/path_travesal/image24.png)

## Phân tích con bọ Path Traversal:

Các biến thể khác của lỗi bảo mật Path Traversal

![](/assets/img/posts/path_travesal/image25.png)

### Level 4:

Link: http://pathtraversal.cyberjutsu-lab.tech:8094/

Tìm hiểu từng document root:

`register.php` :

```php
<?php
  include './db.php';

  if(isset($_GET["debug"])) die(highlight_file(__FILE__));

  // If is login
  if (isset($_SESSION['name'])) {
    header('Location: /'); //tao ra mot response header dieu huong ve /
    die();
  }
  $error = '';
  if (isset($_POST["name"])) { //unstrusted data xuat hien
    $name = strval($_POST["name"]);
    if (!preg_match('/^[a-z0-9]+$/', $name)) { //kiem tra username chi duoc nhap tu a-j va 0-9
      $error = 'Name must be [a-z0-9]+';
    } else {
      $user = $db->select_user_by_username($name);
      if (!!$user) {
        $error = 'Name already exist';
      } else {
        $_SESSION["name"] = $name; // $_SESSION["name"] tro thanh untrusted data
        $db->create_user($name);
        // Create place for upload
        $dir = '/var/www/html/upload/' . $name; //Tuy nhien khong the thao tung bien name
        if ( !file_exists($dir) )
          mkdir($dir);
        die(header('Location: /'));
      }
    }
  }
?>
```

⚠️Hàm `include` : read file và execute file

Ví dụ ở trên, `include './db.php';` nghĩa là nó đang đọc file và excute file trực tiếp từ `db.php`

Một ví dụ khác: Nếu ta tạo một file text `test.txt` với nội dung

```php
<?php phpinfo(); ?>
```

và include nó thì hoàn toàn thực thi được

![](/assets/img/posts/path_travesal/image26.png)

⇒ Mức độ nguy hiểm: ⭐⭐⭐⭐⭐

Tuy nhiên ở dòng   `include './db.php';` không phải untrusted data vì db.php là của anh dev 

→ Không thể can thiệp

`profile.php` 

```php
<?php
  // error_reporting(0);
  include './db.php';

  // if is not login
  if (!isset($_SESSION['name'])) {
    header('Location: /register.php');
    die();
  }

  $response = "";
  if (isset($_FILES["fileUpload"])) {
    // Always store as avatar.jpg
    // Thuc hien luu file do user upload len
    // Duong dan: var/www/html/upload/<Session_name>/avatar.jpg
    // -> Tat ca cac file duoc upload len deu duoc doi ten thanh avatar.jpg
    move_uploaded_file($_FILES["fileUpload"]["tmp_name"], "/var/www/html/upload/" . $_SESSION["name"] . "/avatar.jpg");
    $response = "Success";
  }
?>
```

`game.php` 

```php
<?php
    include './db.php';

    // if is not login
    if (!isset($_SESSION['name'])) {
        header('Location: /register.php');
        die();
    }
    if (isset($_POST["points"])) {
        $points = intval($_POST["points"]);
        $db->update_point($_SESSION['name'], $points);
        header('Content-Type: application/json');
        echo "Successfully update points";
        die();
    }

    if (!isset($_GET['game'])) { //untrusted data xuat hien
        header('Location: /game.php?game=fatty-bird-1.html');
        die();
    }
    $game = $_GET['game']; // -> untrusted data
?>

<!DOCTYPE html>
<html lang="en">
    <head>
        <?php include './views/header.html'; ?>
    </head>

    <body>
        <script>
            function submitPoint(points) {
                fetch("/game.php", {
                    method: "POST",
                    credentials: 'same-origin',
                    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                    body: `points=${points}`,
                }).then(resp => resp.text()).then(t => alert(t));
            }
        </script>
        <?php include './navbar.php'; ?>
        <br><br><br>

        <br>
        <div style="background-color: white; padding: 20px;">
            <?php include './views/' . $game; ?> 
            <!---> unsafe method "include" + unstrusted data $game = vo mom -->
				</div>
    </body>
</html>
```

Hướng khai thác:

Bước 1: Upload một file chứa mã thực thi lên server, ở đây mình sẽ upload một file có nội dung là 

`<?php phpinfo();?>`

- Dựa vào logic của chương trình, chúng ta có thể xác định được đường dẫn sau khi upload của chương
trình trên hệ thống là `/var/www/html/upload/<user_name>/avatar.jpg`

Bước 2: 

Khai thác include để thực thi file nguy hiểm vừa upload bằng cách truy cập đến endpoint
`/game.php?game=../upload/cyberjutsu/avatar.jpg`

`$game` đang ở thư mục `./views/` tức là `./var/www/html/views/`

Giờ mục tiêu của chúng ta là truy cập thư mục upload

⇒ B1: Quay lại folder document root:  `./var/www/html/views/..` → `./var/www/html/`

    B2: Truy cập folder upload:  `./var/www/html/upload`

    B3: Truy cập file avatar.jpg mà chúng ta vừa upload lên `/var/www/html/upload/<user_name>/avatar.jpg`

⇒ include(unstrusted_data) + Kiểm soát nội dung file = RCE
