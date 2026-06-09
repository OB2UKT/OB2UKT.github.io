---
layout: post
title: Đại Cương Web
categories: [Cybersecurity, Web-pentest]
tags: []
author: no1hiz9
---

## Chương 1: HTTP Protocol
### Khái niệm
HTTP - Hyper Tranfer Text Protocol - là 1 giao thức truyền đi dữ liệu giữa client và server nằm ở tầng 7 OSI (tầng ứng dụng), **lưu ý nó sẽ không lưu trạng thái (stateless)**, mà chỉ phục vụ mục đích truyền tải thông tin. 
### So sánh được giữa HTTP 1.1 và 2.0
![Compare between version 1.1 and 2.0]({{ site.baseurl }}/assets/images/compareVersionHTTP.png)
Về kiến trúc thì HTTP 2.0 sẽ mang lại tốc độ vượt trội hơn so với 1.1 bởi vì thay vì mỗi request sẽ là 1 quy trình tcp-handshake thì giờ đây trên 2.0 chỉ cần 1 lần tcp-handshake là đủ, mọi dữ liệu đều được truyền tải thông qua đường ống này.
### Phân biệt được giữa HTTP Request và HTTP Response

#### HTTP Request 
Sẽ xuất phát từ phía client/browser đến web server để yêu cầu tài nguyên từ phía web server. 

### Cấu trúc của 1 HTTP Protocol 
Sẽ được chia làm 3 phần, phần đầu tiên sẽ là Start-line, Header, Body 
![Structure of HTTP Request]({{ site.baseurl }}/assets/images/structure_httpRequest.png)

##### Start-line
- Bao gồm Method, Path/URL, Protocol Version (chỉ gồm 3 phần chính và cách nhau bằng khoảng trắng - lưu ý chỉ có 2 khoảng trắng và sẽ không có bất cứ khoảng trắng nào khác, chính vì thế tại path/url mới xuất hiện url_encoding để tránh hiện tượng syntax error nếu như sai cấu trúc trên).

- Tiếp theo sẽ là Headers, phần này cấu trúc sẽ được viết tùy ý, mục đích là phần thông tin thêm để báo cáo cho webserver biết thêm về request. 
\<Blank> Here ----- Phân Header và Body sẽ cách nhau 1 dòng 
- Cuối cùng sẽ là phần body chứa thông tin liên quan (nếu có).

#### HTTP Response 
Sẽ gửi lại phản hồi cho request trước đó và yêu cầu phía client/browser chuẩn bị cho những thứ nó truyền tải về để có thể xử lý.


### Cấu trúc của 1 HTTP Protocol 
Sẽ được chia làm 3 phần, phần đầu tiên sẽ là Response Status Line, Header, Body 
![Structure of HTTP Response]({{ site.baseurl }}/assets/images/structure_httpResponse.png)

##### Response Status Line
- Bao gồm HTTP version, Status Code, Status Phrase 

- Tiếp theo sẽ là Headers, phần này cấu trúc sẽ được viết tùy ý, mục đích là phần thông tin thêm để báo cáo cho webserver biết thêm về request. 
\<Blank> Here ----- Phân Header và Body sẽ cách nhau 1 dòng 
- Cuối cùng sẽ là phần body chứa thông tin liên quan (nếu có).


### Các loại Status Code đáng chú ý 
1xx Informational
2xx Success
3xx Redirection
4xx Client error (404 - Page not found maybe query error page) (400 - Bad Request --> Send Error Format...)
5xx Server Error   (500 - Initial Server Error)

###  Các loại HTTP METHOD đáng chú ý
GET - Read 
HEAD - Thường dùng để đọc check version, độ dài của mục tiêu, ngoài ra thường dùng để brute-force kiểm tra xem tài nguyên có tồn tại không, vì không trả về body - phản hồi nhanh nên được chuộng.
OPTIONS - Check Methods, Content Type - hỏi xem target có hỗ trợ method nào khác không.
POST - Tạo mới 1 giá trị.
PUT - Thường dùng để update/thay thế
PATCH - Update/ thay đổi 
DELETE - Xóa.

### Phân biệt được 2 METHOD GET / POST

### Câu hỏi lưu ý
Vậy thì nếu như nói HTTP là stateless vậy thì làm sao server nhớ mà trả về phản hồi, vì stateless sẽ không lưu bất kì trạng thái nào trên server. 
> Câu trả lời là đối với mỗi request thay vì dựa vào http thì ta sẽ nhờ vào kết nối tcp-handshake trước đó để khi gửi request thì ta sẽ dựa vào đường ống này để trả về response. Lúc thiết lập tcp-handshake ta hoàn toàn có được địa chỉ ip và port nguồn của gói tin. Và ngay sau khi gói tin response được trả về thì sẽ đóng lại quy trình TCP-Handshake.

Vậy thì bởi vì tính stateless của HTTP, liệu có cách nào để ta có thể quản lý được người dùng, phục vụ cho cơ chế xác thực ?
> Lúc này sẽ có những cơ chế như cookie, session được sinh ra, cải tiến hiện đại hơn thì ta sẽ có token và JWT.
