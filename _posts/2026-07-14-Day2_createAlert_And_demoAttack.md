---
title: Day 2 - Trải nghiệm tạo tính năng và tiến hành demo tấn công thực tiễn
date: 2026-07-14 14:30:00 +0700
categories: [Lab Setup, SOC Analyst, Attack & Defense]
tags: [Attack & Defense, SOC Analyst]
math: true
mermaid: true
---

## Kịch bản tấn công thực tế 
Ở phần này ta sẽ chia việc tấn công thành nhiều giai đoạn và sẽ giải thích chi tiết ở từng giai đoạn. Mục đích của phần này nhằm demo lại một cuộc tấn công trong thực tế nhắm vào Windows Server (đã bật sẵn RDP và cấu hình bảo mật yếu) mà chúng ta đã dựng trước đó ta sẽ tiến hành bruteforce sau đó trích tiến hành khai thác và đánh cắp thông tin.

### Initial Access

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Initial_Access.jpg){: width="800" height="500" }
_Hình 1: Mô tả giai đoạn Initial Access._

Ở giai đoạn đầu tiên trong cuộc tấn công ta sẽ cố gắng brute-force password RDP của Windows Server. Đi vào chi tiết ta sẽ sử dụng Crowbar để thực hiện brute-force password dựa trên từ điển password có sẵn. 

Nguyên nhân bị tấn công đến từ việc thiết lập mật khẩu của Windows Server quá mặc định và dễ đoán. Ở trên máy Kali linux ta tiến hành thu thập chuỗi từ điển password phổ thông và tiến hành tấn công tìm password của RDP Windows Server. Ta tiến hành sử dụng NetExec để tiến hành brute-force và tìm ra mật khẩu của tài khoản RDP Windown Server. 

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/result_bruteforce_RDP.jpg){: width="800" height="500" }
_Hình 2: Mô tả kết quả sau khi thực hiện brute-force mật khẩu RDP._


Sau khi đã có được tài khoản ta tiến hành truy cập vào Windows Server thông qua công cụ xfreerdp với lệnh sau cụ thể sau:

```bash
xfreerdp /u:Administrator /p:password@123 /v:192.168.122.184:3389
```

### Discovery

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Discovery_phase.jpg){: width="800" height="500" }
_Hình 3: Tiến hành khai thác thông tin trên máy nạn nhân sau khi đã kết nối RDP._

Sau khi đã có thể kết nối vào Windows Server ta hoàn toàn có thể thao tác với quyền Administrator, có thể thấy được khi attacker đã kiểm soát được Administrator (High Intergrity) thì mức độ impact của cuộc tấn công đã được đẩy cao và gây thiệt hại đáng kể.

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/discovery_after_connect_via_RDP.jpg){: width="800" height="500" }
_Hình 4: Tiến hành khai thác thông tin trên máy nạn nhân sau khi đã kết nối RDP._

Trong thực tế từ bước này khi đã có thể chiếm quyền được Administrator ta có thể dễ dàng tìm các máy khác trong cùng mạng nội bộ từ đó nhảy và lây lan để duy trì cũng như lẩn trốn nếu sau đó bị truy vết hoặc tìm ra. Việc lây lan và lẫn trốn sẽ giúp ta duy trì khả năng xâm nhập lâu nhất có thể nếu làm tốt bước Discovery và tìm được nhiều lỗ hổng, cấu hình kiến trúc từ bên trong sẽ tạo ra tiền đề và phương hướng để có những cuộc tấn công sau này.

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Defender_Evasion.jpg){: width="800" height="500" }
_Hình 5: Tiến hành tắt toàn bộ Defender để có thể dễ dàng triển khai tấn công mà không bị cản trở._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Turn_off_Defender.jpg){: width="800" height="500" }

Để tránh tình trạng bị cản trở bởi các công cụ Defender trong quá trình thực thi thiết lập kết nối tới C2 Server ta sẽ thực hiện tắt ngay sau khi đã có toàn quyền kiểm soát hệ thống.

### Execution & Exfiltration Senitive Data

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Execution.jpg){: width="800" height="500" }
_Hình 6: Mô tả quy trình thực thi agent trước khi tấn công._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/C2_Exfilltration.jpg){: width="800" height="500" }
_Hình 7: Quy trình đánh cắp dữ liệu ._

Dựa trên những tiền đề trước đó ở bước này ta thực hiện bước cài đặt agent lên máy nạn nhân và tiến hành chạy nó. Bởi vì bối cảnh hiện tại ta đã dựa vào việc khai thác thành công Remote Desktop trước đó nên tại bước này ta chỉ cần host file (.exe) và tải về để thực hiện chạy file cài đặt agent.


Để thực hiện được điều này ta cần tạo thành công agent qua các bước sau: 

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_1.jpg){: width="800" height="500" }
_Hình 8: Cấu hình thể loại file thực thi, lựa chọn mục tiêu tấn công của payload._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_2.jpg){: width="800" height="500" }
_Hình 9: Lựa chọn những lệnh có thể sử dụng sau khi đã cài đặt thành công._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_3.jpg){: width="800" height="500" }
_Hình 10: Cấu hình việc callback về C2 server sửa Header._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_4.jpg){: width="800" height="500" }
_Hình 11: Tiến hành build payload._

Sau khi tạo thành công payload cũng chính là lúc ta thực hiện áp dụng nó để tấn công victim

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/save_host_payload_5.jpg){: width="800" height="500" }
_Hình 12: Host file cài đặt trên chính server Mythic._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Download_payload_execute_onVictim_6.jpg){: width="800" height="500" }
_Hình 13: Cài đặt Agent lên máy Victim._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Result_callback_from_victim_7.jpg){: width="800" height="500" }
_Hình 14: Sau khi chạy và cài đặt thành công agent sẽ có một callback từ máy victim._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Exfilltration_SensitiveData_8.jpg){: width="800" height="500" }
_Hình 15: Tiến hành đánh cắp file Sensitive trên máy nạn nhân._


## Triển khai biện pháp ngăn ngừa và phòng tránh
Ngoài việc triển khai các biện pháp ngăn chặn tấn công đối với các mục tiêu bên trong mạng ta cần đề cao khả năng phòng chống từ trước dựa trên những dấu hiệu thu thập từ logs, điều này giúp ta chủ động ngăn ngừa từ sớm tránh để những hậu quả nghiệm trọng tác động vào mục tiêu, phát hiện những biến thể của kiểu tấn công và phòng thủ chủ động.

### Tạo Alert 
Alert là một tính năng được sử dụng để đảm bảo luồng dữ liệu, giám sát luồng log để đảm bảo trạng thái hoạt động của các agent (Filebeat, winlog, sysmon, ...) trên máy chủ, nếu như chúng ngưng kết nối và không cập nhật log về ELK sẽ thông báo ngay cho các vị trí chịu trách nhiệm xử lý.

Đảm bảo tình trạng tài nguyên, cảnh báo khi CPU của máy chủ web vượt quá ngưỡng quy định hoặc ổ cứng của Elasticsearch Cluster sắp đầy. Ngoài ra sẽ cảnh báo khi thời gian phản hồi của một API bị chậm hoặc tỷ lệ lỗi HTTP 500 tăng đột biến. Alert log thiên hướng về cảnh báo tình trạng hệ thống, những thay đổi bất thường trong logs (đo lường thay đổi đo về lượng) để đưa ra cảnh báo/thông báo.

Ta có thể áp dụng để tiến hành đếm được những bất thường trong security, những hành vi bất thường dựa trên ngưỡng định sẵn. Tuy nhiên chỉ nhằm mục đích tham khảo và lấy số liệu thống kê chứ không được sử dụng phục vụ mục đích security chính bởi vì một số nhược điểm về độ chuẩn xác và dễ bị false positive. Ta sẽ sử dụng htính năng khác để đảm nhiệm vai trò security được phân tích ở bên dưới. 

#### Tạo Alert cho SSH Brute-force
Đầu tiên để có thể tạo được alert để bắt chuẩn xác những sự thay đổi bất thường, hay những hành vi bất thường ta cần nắm bắt được yếu tố, đặc điểm của các hành vi trên. Đối với Brute-force là một dạng tấn công với mục đích là tấn công vào phần xác thực tài khoản của mục tiêu khi cố tình thử toàn bộ mật khẩu có thể có để tìm ra mật khẩu chuẩn nhất để có thể đăng nhập tài khoản nạn nhân mà không có sự cho phép. Vậy thì trong lúc thử sẽ có những đặc điểm nhận dạng như: 
- Cùng địa chỉ **ip** cố tình nhập sai password nhiều lần trong một khoảng thời gian ngắn --> xuất hiện nhiều log như **fail attempts**.
- Tinh vi hơn có thể là nhiều địa chỉ ip đến từ nhiều nơi cùng cố thử mật khẩu của một tài khoản đang bị nhắm tới. --> Tài khoản của một **user** bị thử sai password nhiều lần trong khoảng thời gian ngắn.

Dựa trên những đặc điểm/tiêu chí trên ta thực hiện truy vấn để tìm ra những log, manh mối nghi ngờ là SSH Brute-force.

![Intercepted Request](assets/img/material_posts/post_3/Create_Alert/Query_logs_SSH_Bruteforce.jpg){: width="800" height="500" }
_Hình 16: Mô tả các logs đáng nghi liên quan tới SSH Brute-Force._

Từ những logs trên ta tiến hành tạo Alert để thông báo mỗi khi tình trạng này xảy ra và chủ động phòng ngừa/ngăn chặn trước những nguy cơ, hiểm họa tiềm ẩn trong tương lai.

![Intercepted Request](assets/img/material_posts/post_3/Create_Alert/Create_Alert_1.jpg){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/post_3/Create_Alert/Create_Alert_2.jpg){: width="800" height="500" }
_Hình 17: Chi tiết cấu hình của Alert._

Ở phần nay ngoài việc đặt ngưỡng (threshold) cho việc kích hoạt alert ta cấu hình thêm việc tạo và gửi ticket đến osTicket Server mà trước đó đã thực hiện cấu hình và cài đặt. Những lợi ích mà điều này mang lại đã được trình bày chi tiết ở blog trước.

#### Tạo Alert cho RDP Bruteforce
Tiếp theo tương tự với phần trước khi ngăn chặn SSH bruteforce ta cần tìm ra được những đặc điểm nhận dạng của dạng tấn công này và từ đó khai thác chúng để đưa tìm ra được những logs đáng ngờ. Khác với Ubuntu, Windows đã có sẵn những EventID để ta có thể dựa vào đó và xác định được những logs Failed Authentication. 

![Intercepted Request](assets/img/material_posts/post_3/Create_Alert/information_about_failed_authentication.jpg){: width="800" height="500" }
_Hình 18: Chi tiết thông tin về Event ID 4625._

Tương tự với phần trên khi đã xác định được các yếu tố/đặc điểm của kiểu tấn công ta sẽ tạo alert dựa trên điều đó.

![Intercepted Request](assets/img/material_posts/post_3/Create_Alert/Query_logs_RDP_Bruteforce.jpg){: width="800" height="500" }
_Hình 19: Chi tiết alert RDP Bruteforce._

### Tạo Detection rules (SIEM)
Khác với Alert - xuất hiện cảnh báo/thông báo khi vượt ngưỡng bất thường/thỏa mãn điều kiện. Rule có phần chi tiết và nhiều chức năng hơn, rule có khả năng phát hiện mối đe dọa/tấn công dựa trên dấu hiệu xâm nhập và chuỗi hành vi (khác với Alert chỉ nhận diện bằng threshold). Ngoài ra rule có thể tạo ra một Security Alert bao gồm nhiều thông tin chi tiết về đối tượng hơn Alert, điểm đánh giá mức độ nguy hiểm, gán nhãn MITRE ATT&CK, quy trình xử lý đối tượng, các bài viết liên quan về đối tượng để đội SOC Analyst/Incident Response có thể dựa vào thông tin để xử lý nhanh chóng, giúp cung cấp bối cảnh, quy trình xử lý, thông tin cần thiết để giảm thời gian phân tích logs và xử lý công việc chuyên nghiệp hơn.

Trong thực tế Detection rules (SIEM) thường được sử dụng để đảm bảo sự an toàn của tài sản số, bởi vì khả năng phân tích hành vi và khả năng cung cấp đầy đủ bối cảnh và quy trình phân tích và xử lý cụ thể giúp cấu hình hệ thống chuyên nghiệp và xử lý nhanh chóng, đảm bảo sự an toàn của hệ thống. Một điểm đáng tiền của Dectection rule so với Alert đó là khả năng đào sâu vào hành vi và chi tiết logs thay vì dựa vào threshold như Alert (Discovery) chính vì thế nó có thể tránh khỏi false positive do attacker cố tình nhằm đánh lạc hướng hoặc gây động với nhiều mục đích khác nhau.

Cách thiết lập rule ở đây ta sử dụng cơ bản bằng threshold tương tự với phần alert trước đó, ta liên kết với osTicket để gửi thông báo/cảnh báo về mỗi khi phát hiện tấn công. 

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/detail_rules_SSH_bruteforce.jpg){: width="800" height="500" }
_Hình 20: Chi tiết cấu hình rule SSH bruteforce._

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/detail_rules_bruteforce_RDP.jpg){: width="800" height="500" }
_Hình 21: Chi tiết cấu hình rule RDP bruteforce._

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/Detail_Rule_Detect_C2_Agent.jpg){: width="800" height="500" }
_Hình 22: Chi tiết cấu hình rule detect C2 Agent._

Đối với việc phát hiện C2 Agent ta sẽ dựa vào một số yếu tố đặc điểm của nó như sau: 
- Tạo tiến trình thực thi mới --> Event ID = 1, việc tạo tiến trình mới để có thể thực thi các lệnh từ xa là đặc điểm dễ dàng thấy được của agent.

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/information_C2Agent.jpg){: width="800" height="500" }
_Hình 23: Thông tin chi tiết về Event ID 1._

- Sự khác biệt giữa tên file gốc và file thực thi trên máy nạn nhân, ở đây tệp tin agent được giả mạo là "svhost-no1hiz9.exe" nhưng thực chất tên gốc của nó là apollo.exe là một Agent của Mythic. 

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/detail_c2_agent_logs.jpg){: width="800" height="500" }
_Hình 24: Chi tiết log của C2 Agent hiển thị bởi Kibana._

Sau khi thiết lập xong rule ta tiến hành tấn công (SSH Bruteforce) lại một lần nữa để kiểm tra hoạt động của rule ta vừa mới thiết lập.

![Intercepted Request](assets/img/material_posts/post_3/Create_Rules/Ticket_receive.jpg){: width="800" height="500" }
_Hình 25: Thông báo được gửi tới osTicket với nội dung của hành vi đe dọa hệ thống._


### Cốt lõi lỗ hổng và cách phòng tránh
#### SSH Bruteforce
Cốt lõi vấn đề dẫn đến SSH Brute force trong bài lab gặp phải đến từ việc cấu hình sai 


#### RDP Bruteforce


## Kết quả và bài học
Bài blog đã hoàn thành mục đích khi xây dựng và triển khai một mô hình mạng lưới gồm nhiều thành phần thu nhỏ một mô hình hệ thống trong thực tế. Đã triển khai tấn công và đưa ra những phương hướng khắc phục hậu quả. Đã mô phỏng hoàn toàn một mô hình xử lý sự cố khi tiến hành.