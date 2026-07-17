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

Để tránh tình trạng bị cản trở bởi các công cụ Defender trong quá trình thực thi thiết lập kết nối tới C2 Server ta sẽ thực hiện tắt ngay sau khi đã có toàn quyền kiểm soát hệ thống.

### Execution
Dựa trên những tiền đề trước đó ở bước này ta thực hiện bước cài đặt agent lên máy nạn nhân và tiến hành chạy nó. Bởi vì bối cảnh hiện tại ta đã dựa vào việc khai thác thành công Remote Desktop trước đó nên tại bước này ta chỉ cần host file (.exe) và tải về để thực hiện chạy file cài đặt agent.

Để thực hiện được điều này ta cần tạo thành công agent qua các bước sau: 
![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_1.jpg){: width="800" height="500" }
_Hình 6: Cấu hình thể loại file thực thi, lựa chọn mục tiêu tấn công của payload._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_2.jpg){: width="800" height="500" }
_Hình 7: Lựa chọn những lệnh có thể sử dụng sau khi đã cài đặt thành công._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_3.jpg){: width="800" height="500" }
_Hình 8: Cấu hình việc callback về C2 server sửa Header._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Gen_payload_4.jpg){: width="800" height="500" }
_Hình 9: Tiến hành build payload._

Sau khi tạo thành công payload cũng chính là lúc ta thực hiện áp dụng nó để tấn công victim

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/save_host_payload_5.jpg){: width="800" height="500" }
_Hình 6: Host file cài đặt trên chính server Mythic._


![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Download_payload_execute_onVictim_6.jpg){: width="800" height="500" }
_Hình 7: Cài đặt Agent lên máy Victim._


![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Result_callback_from_victim_7.jpg){: width="800" height="500" }
_Hình 8: Sau khi chạy và cài đặt thành công agent sẽ có một callback từ máy victim._

![Intercepted Request](assets/img/material_posts/post_3/Stragery_Attack/Exfilltration_SensitiveData_8.jpg){: width="800" height="500" }
_Hình 9: Tiến hành đánh cắp file Sensitive trên máy nạn nhân._






### C2 & Exfilltration

## Triển khai phân tích và ngăn ngừa

## Giải pháp 

## Bài học rút ra 
