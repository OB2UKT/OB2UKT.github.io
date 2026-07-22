---
title: Write up Hack101 CTF
date: 2026-04-05 09:30:00 +0700
categories: [write up, Web Pentest]
tags: [Write up, Pentest, RedTeam]
math: true
mermaid: true
---


## Micro-CMS v1 - Easy
### Exploit

![Intercepted Request](assets/img/material_posts/Hack101CTF/img1.png){: width="800" height="500" }
_Hình 1: Giao diện trang web ban đầu._

Thử tạo 1 page xem cách nó được tạo và vận hành 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img2.png){: width="800" height="500" }
_Hình 2: Mô tả chức năng web._

Có thể thấy được page được tạo ra có số id ở sau, ứng với page, liệu nó có quy luật hay gì không? Và liệu ta có thể chuyển sang page khác bằng việc đổi phần này? 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img3.png){: width="800" height="500" }
_Hình 3: Trang web bị từ chỗi quyền truy cập._

Ta tiến hành đổi id phía sau và vô tình ta đã tìm kiếm ra 1 trang khác biệt với status code 403 khác với các trang 404 trước đó. Tức là page này có tồn tại và ta đang bị chặn khả năng đọc. Đối với endpoint **/page**. Vậy thì liệu rằng ta có thể truy cập vào page này bằng con đường khác?

![Intercepted Request](assets/img/material_posts/Hack101CTF/img4.png){: width="800" height="500" }
_Hình 4: Kết quả flag ta thu được khi thực hiện khai thác lỗi Broken Access Control._

Ở đây ta thử chuyển sang endpoint **/page/edit** thì ta đã thấy được nội dung bên trong của page thông qua việc chỉnh sửa.

Tiếp đó vấn đề đặt ra là các page này được quản lý bằng id phía sau, vậy thì các id này sẽ được truy vấn như thế nào? Liệu rằng id này sẽ được gửi thắng tới db để truy xuất thông tin ra mà không cần qua kiểm duyệt.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img5.png){: width="800" height="500" }


Ta thực hiện test trên từng endpoint bằng cách thêm dấu ' vào đằng sau phía id để nhằm kích hoạt lỗi hoặc trả về hint nào đó từ phía server. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img6.png){: width="800" height="500" }

Vậy là cuối cùng tại chính endpoint này ta đã có thêm flag.

Ta phân tích kỹ hơn về page source, và những thứ ta truyền vào và cách nó được hiển thị trên trang web. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img7.png){: width="800" height="500" }

Có thể thấy được field title mà ta truyền vào sẽ được gán chính cho thẻ \<h1> vậy thì liệu rằng ta có thể thao túng ở đây và khai thác lỗ hổng XSS?

![Intercepted Request](assets/img/material_posts/Hack101CTF/img8.png){: width="800" height="500" }

Câu trả lời là có khi ta đã thực hiện chèn vào đoạn \<script> bên trong và thực thi thành công 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img9.png){: width="800" height="500" }


Tiếp theo ta phân tích trên page **Markdown Test** thấy được 2 khả năng khai thác XSS tìm ẩn ở chính trang này. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img10.png){: width="800" height="500" }

Có thể thấy được ngay tại vị trí **src** ta có thể thao túng được và thẻ **\<button>**. Ta thử khai thác XSS như sau. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img11.png){: width="800" height="500" }

Có thể thấy được flag đã được trả về ở 2 vị trí mà ta tiến hành khai thác

## Micro-CMS v2 - Moderate
### Flag 1

![Intercepted Request](assets/img/material_posts/Hack101CTF/img12.png){: width="800" height="500" }

Dựa vào cách tra cứu các trang web ẩn trước đó ta đã thử với /admin, /page/... Và ta có thêm hint trong việc tìm kiếm ra 1 trang web với status 403. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img13.png){: width="800" height="500" }

Ta thử đổi method ở đây và nó nhả ra lỗi, ta tiếp tục thử với các endpoint khác để tìm ra lỗ hổng. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img14.png){: width="800" height="500" }

Sai lầm ở đây đến từ việc trang web chỉ yêu cầu xác thực ở HTTP METHOD GET(method mà thông thường client sẽ nhìn thấy), mà không bảo vệ đối với các method khác và ta hoàn toàn có thể thực hiện khai thác đổi method bằng cách thay thế bằng POST và cuối cùng ta cũng kiếm được flag, khá bất ngờ ở điểm này.

### Flag 2 
![Intercepted Request](assets/img/material_posts/Hack101CTF/img15.png){: width="800" height="500" }

Ngay từ trang web đã gợi ý cho ta thông tin rằng đã yêu cầu Authentication, đối với những endpoint **/edit**,...

![Intercepted Request](assets/img/material_posts/Hack101CTF/img16.png){: width="800" height="500" }

Mục tiêu là ta cần đăng nhập với quyền admin và có thể tiếp tục khai thác sâu hơn.


![Intercepted Request](assets/img/material_posts/Hack101CTF/img17.png){: width="800" height="500" }

Ta thử test với ký tự ' vào các field nhập vào và nhận được lỗi trả về, nghi ngờ đoạn này có thể khai thác bằng SQL injection. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img18.png){: width="800" height="500" }


Đầu tiên ta cần vượt qua được thử thách của login, ta cần bypass để có thể truy cập để có quyền admin. username=HEHEH, password=123 và phản hồi là **Unknown user** ở đây có thể hiểu rằng ta đã không điền đúng username.

Hình dung cơ chế xác thực của login cơ bản như sau: 

```SQL
SELECT * FROM users WHERE username= /Untrusted data/ AND password = /untrusted data/
```

![Intercepted Request](assets/img/material_posts/Hack101CTF/img19.png){: width="800" height="500" }

Ta thử test black-box, ta thử username = HEHE' OR 1 -- A và passwd = ' or 1, ta có thể thấy được output ra đã có khác biệt **Invalid password**, nghi ngờ ta đã bypass được username tuy nhiên vẫn bị sai ở password (xác định được cơ chế kiểm tra password phía sau và sẽ khác biệt so với cơ chế mà ta hình dung - có thể chúng sẽ độc lập vì username không thể thao túng được phần password). Ở đây ta có thể hình dung tới việc cơ chế xác thực có thể chỉ lấy username sau đó đem kết quả truy vấn (password) so sánh với password người dùng nhập vào. Vậy thì ta cần kiếm cách để thao túng đầu ra ở đây. Câu truy vấn có thể là 

```SQL
SELECT password FROM admins WHERE username = /Untrusted Data/
```

Ta tiếp tục thử với payload sau:
```SQL
' UNION SELECT '1' -- A
```

Payload này được truyền vào username, nhằm thao túng câu query tại username. Ta thực hiện thao túng đầu ra của câu lệnh truy vấn. Sau đó ta nhập password = 1 thì ta đã có thể truy cập được trang web, vậy thì như đã nói cơ chế mà ta hình dung đã đúng, việc ta thao túng được kết quả trả về đã giúp ta bypass được việc xác thực mật khẩu.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img20.png){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/Hack101CTF/img21.png){: width="800" height="500" }

Và ta đã có flag đầu tiên. Cũng chính là trang page 3 thiếu quyền mà ban đầu ta đã tìm kiếm.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img22.png){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/Hack101CTF/img23.png){: width="800" height="500" }

Ta thử những lỗ hổng cũ trước đó ở bài v1 thì dường như đã không thực hiện được và đã bị ngăn chặn. 

### Flag 3

![Intercepted Request](assets/img/material_posts/Hack101CTF/img24.png){: width="800" height="500" }

Ở sau phần loged in khi đăng nhập thành công trang web đã gợi ý cho chúng ta về việc tìm kiếm 1 tài khoản thực tế/có tồn tại. Ở đây ta có thể thực hiện việc Blind SQL để thực hiện việc tìm kiếm ra account thực tế. 

#### Viết payload 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img25.png){: width="800" height="500" }

Đầu tiên ta kết nối connection với server của bài lab và kiểm tra các thông tin đầu ra để viết logic phân tích.
Logic thực thi sẽ là ta thực hiện việc bypass trước đó để làm yếu tố đúng sai, sau đó ta thực hiện duyệt từng ký tự để tìm kiếm ra kết quả đúng.

```python
def test_connection(url): 
    username = ""
    passwd = ""
    form_data = {
        "username": "test", 
        "password": "HEHE"
    }

    for i in range(1, 101):
        character_for_this_index = False
        for guess_character in 'abcdefghijklmnopqrstuvwxyz0123456789':
            payload = f"'UNION SELECT 'HEHE' FROM admins WHERE SUBSTRING(username,{i}, 1) = '{guess_character}' -- A"
            form_data["username"] = payload
            response = requests.post(url, data=form_data)
            if response.status_code == 200 and "Logged in" in response.text:
                print(f"Found character at position {i}: {guess_character}")
                username += guess_character
                character_for_this_index = True
                break
        if not character_for_this_index:
            print(f"\n Last result: {username}")
            break

```
Payload này là kỹ thuật khai thác Blind SQL - Error Base. Ta lấy việc login thành công làm logic đúng, nếu như mệnh đề WHERE phía sau đúng (khi ta dò đúng ký tự), nếu sai thì tiếp tục dò, dò cho tới khi hết ký tự.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img26.png){: width="800" height="500" }

Đây là kết quả của việc chạy code trên ta đã dò ra được tài khoản của admin.


![Intercepted Request](assets/img/material_posts/Hack101CTF/img27.png){: width="800" height="500" }

Sau khi nhập tài khoản trên thì ta đã có flag cuối cùng.

## Postbook - EASY
###  FLAG 1  & FLAG 2
Đối với flag đầu tiên ta đánh vào lỗ hổng IDOR khi mới vào trang web. Ta thử Sign in và đầu tiên tiếp cận là việc trang web quản lý các bài blog bằng các giá trị của id. Ta thử chuyển sang các bài blog khác dựa trên id nhằm tìm ra được các bài blog ẩn tuy nhiên chưa giới hạn quyền mà ta có thể truy cập được. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img28.png){: width="800" height="500" }

Và ta đã tìm kiếm được 1 bài ẩn với id=2, đây là bài viết ẩn của admin dành riêng cho Diary. Và ta đã đọc được bài viết này. Có được flag đầu tiên.

Nắm bắt được việc thiếu kiểm soát quyền của người dùng ta thử sửa một bài viết public từ admin thông qua việc sửa đi id bài viết id = 1 để có thể sửa nội dung bài post. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img29.png){: width="800" height="500" }

Và sau khi sửa ta đã có flag thứ 2.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img30.png){: width="800" height="500" }

### Flag 3

![Intercepted Request](assets/img/material_posts/Hack101CTF/img31.png){: width="800" height="500" }

Ta thử test thử khả năng xảy ra lỗi XSS thì ta thấy được, đoạn dữ liệu mà ta truyền vào đã bị xem là chuỗi và không xảy ra bất cứ hiện tượng gì. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img32.png){: width="800" height="500" }

Để kiếm được flag này ta cần phụ thuộc vào hint và hint gợi ý cho ta việc có 1 tài khoản user nào đó có password rất dễ đoán. 
> HINT: The person with username "user" has a very easy password...

Và ta thử việc test mật khẩu bằng từ điển và dừng lại ngay ở việc password chính là "password".

### Flag 4 
Ở bước này tôi gần như bí ý tưởng khi mà không biết được những khả năng có thể khai thác tiếp trên trang web thì ta lại có hint cho 1 phép tính "189 * 5", phép tính này trả về cho ta giá trị 945, và ta có thể sẽ dựa vào nó để có flag tiếp theo. Ta thử với id 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img33.png){: width="800" height="500" }

Thì có thể thấy việc trước đó ta chỉ dò xung quanh giá trị 1-15 là chưa đủ và ta cần đào sâu hơn để có thể thấy được những post như thế này. Khi ta đã tiếp tục xây dựng 

### Flag 5

Ta đi kiếm được nguyên lý hoạt động của cookie, và ta thấy phân tích ban đầu thì có thể thấy được chuỗi cookie tồn tại dưới dạng mã hash MD5.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img34.png){: width="800" height="500" }

Sau khi tôi tìm hiểu về cách mà cookie được tạo ra thì có thể thấy được cookie được tạo cơ bản từ giá trị id và đi hash MD5. Từ đó ta có thể dễ dàng thay đổi mã hash với id =1 để có thể truy cập với dạng của id = 1. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img35.png){: width="800" height="500" }

Ở phần này ta sử dụng extension Modheader để có thể thay đổi giá trị id và ta đã có được flag. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img36.png){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/Hack101CTF/img37.png){: width="800" height="500" }

### Flag 6 
Ta kiếm quy luật tính giá trị của việc xóa 1 bài viết, có thể nút delete được gán với 1 lệnh href thực hiện xóa bài post dựa trên id là đoạn hash md5 và bài viết có id = 1 đã được hash sang giá trị mà ta đã chứng minh trong ảnh, và cách làm này giống như việc định danh cookie của người dùng. 

![Intercepted Request](assets/img/material_posts/Hack101CTF/img38.png){: width="800" height="500" }

Nắm bắt được quy luật đó ta thử login ở 1 acc user thông thường, sử dụng chế độ Intercept để bắt và sửa id của post muốn sửa. Ta thử đổi thành giá trị hash của 1 --> tức là ta nhắm tới xóa bài viết của admin.

![Intercepted Request](assets/img/material_posts/Hack101CTF/img39.png){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/Hack101CTF/img40.png){: width="800" height="500" }

Sau đó ta đã có Flag.

### Flag 7
Đối với flag này đến từ việc kiểm tra xác thực người dùng sai, thay vì kiểm tra người đăng bài dựa trên session/cookie thì ở đây form sẽ tự động gán giá trị id của user và gửi ngay trong form

![Intercepted Request](assets/img/material_posts/Hack101CTF/img41.png){: width="800" height="500" }

Có thể thấy được id người dùng đã được bỏ thẳng vào form và ta thử đổi sang id của người dùng khác và submit thì đã có flag cuối, và bài post đã thay đổi **"Author: admin"**

![Intercepted Request](assets/img/material_posts/Hack101CTF/img42.png){: width="800" height="500" }
