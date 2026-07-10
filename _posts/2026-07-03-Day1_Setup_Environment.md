---
title: Day 1 - Xây dựng mô hình hệ thống thực tế
date: 2026-07-03 14:30:00 +0700
categories: [Lab Setup, SOC, BlueTeam]
tags: [BlueTeam, SOC]
math: true
mermaid: true
---

## 1. Tổng quan kiến trúc của mô hình.

![Intercepted Request](assets/img/material_posts/post_1/structure_lab.jpg){: width="800" height="500" }
_Hình 1: Hình ảnh sơ đồ hệ thống_

### 1.1 Thành phần
Hệ thống sẽ phân loại server dựa vào chức năng cụ thể như sau:
- Server mục tiêu: Windows Server 2022, Ubuntu Server.
- Server quản lý Agent trên các máy chủ mục tiêu: Fleet Server.
- Server lưu trữ, phân tích, trực quan hóa log: Elastic & Kibana.
- Server quản lý ticket, cảnh báo phát sinh: osTicket Server.
- Server tấn công: C2 Server.

Các server sẽ được cấu hình chung một mạng và việc phân tích log sẽ dựa vào Web GUI.

### 1.2 Mục tiêu
Triển khai và xây dựng một mô hình đơn giản có đầy đủ thành phần vai trò, chức năng. Sau đó triển khai các cuộc tấn công nhằm khai thác lên hệ thống và cuối cùng sẽ là phân tích log và tìm ra các dấu hiệu của cuộc tấn công, nắm bắt và tìm cách ngăn chặn, đề xuất giải pháp.

## Tìm hiểu công nghệ sử dụng.
### ELK Stack (Elasticsearch & Logstash & Kibana)
#### Elasticsearch
Bản chất là cơ sở dữ liệu để lưu trữ toàn bộ các log (VD: Syslog, Firewall, Windows Event,...). Điểm nổi trội của nó so với các database khác đến từ khả năng tìm kiếm của nó, sử dụng ngôn ngữ truy vấn ESQL (Elastic search query language), việc tìm kiếm linh hoạt khi có thể sử API và json restful mang đến khả năng có thể truy xuất/ tương tác với CSDL trên các ứng dụng khác nhau (Ở đây có thể nhắc đến ví dụ điển hình là Web Console Kibana).

#### Logstash
Bản chất là phần xử lý dữ liệu. Sau khi thu thập các log từ các nguồn (VD: Elastic Agents & Beats) thì sẽ tiến hành biến đổi và phân tích dữ liệu trước khi nạp vào Elasticsearch. Khi xử lý dữ liệu như vậy đã giúp giảm chi phí lưu trữ và chỉ giữ lại những thông tin quan trọng.

#### Kibana
Bản chất là một công cụ giúp ảo hóa và trình bày việc kết quả dữ liệu truy xuất dưới dạng tổng quan nhất thông qua một Web GUI. Ở trên trang web này người dùng có thể thực hiện các thao tác kéo-thả để điều chỉnh và tương tác. Ngoài ra Kibana có tính năng Discover giúp người dùng có thể thực hiện truy vấn trực tiếp vào Elasticsearch (nhờ ESQ) và có sử dụng Machine Learning để tự động tìm kiếm các mẫu bất thường và thiết lập hệ thống cảnh báo dựa trên dữ liệu log.

#### Elastic Agents & Beats
Bản chất cả 2 công cụ trên là các phần được cài đặt trực tiếp trên máy mục tiêu cần được bảo vệ, chức năng chính như là một máy nghe lén nằm trên chính máy của nạn nhân, đọc những thay đổi và hành động của máy sau đó trả log về phía Logstash (hoặc đẩy thẳng tới Elasticsearch, tùy vào cấu hình).

Có 6 mấy loại Beats, tùy vào chức năng và mong muốn sử dụng:
- File Beat --> Log
- Metric Beat --> Metric
- Packet Beat --> Network Data
- Winlog Beat --> Windows Events
- Audit Beat --> Audit Data
- Heartbeat Beat --> Uptime

Nếu như muốn lấy những log riêng biệt thì có thể lựa chọn Beats ứng với thông tin muốn lấy. Ngoài ra Elastic Agents sẽ lấy toàn bộ log trên hệ thống (điều này nên cân nhắc).

#### So sánh ELK Stack với Splunk 
Về mặt bản chất thì cả 2 công cụ đều có điểm chung:
- Hỗ trợ thu thập, tìm kiếm và trực quan hóa dữ liệu từ nguồn khác nhau.
- Đều cho phép người dùng sử dụng dashboard, truy vấn dữ liệu và thiết lập cảnh báo khi có sự cố bảo mật.

Điểm khác giữa 2 công cụ: 

| Tiêu chí | ELK Stack | Splunk |
| :--- | :--- | :--- |
| Mô hình | Mã nguồn mở, có tính linh hoạt cao | Phần mềm thương mại, đóng gói sẵn |
| Triển khai | Tự cấu hình, quản lý và bảo trì | Cung cấp trải nghiệm có sẵn dễ dàng sử dụng |
| Chi Phí | Chi phí thấp | Chi phí bản quyền cao |
| Hệ sinh thái | Cộng đồng hỗ trợ lớn, tích hợp tùy biến mạnh mẽ | Hỗ trợ cho doanh nghiệp |

Chính vì thế ELK Stack sẽ phù hợp hơn với dự án hiện tại khi ta có thể linh hoạt điều chỉnh và bản thân là open source nên dễ dàng tiếp cận hơn với người mới.

#### Lợi ích của ELK Stack
- Mang lại khả năng quản lý log tập trung, giúp hỗ trợ rà soát và truy vết dữ liệu khi có sự cố an ninh xảy ra.
- Khả năng linh hoạt trong thu thập dữ liệu: Có khả kiểm soát việc chọn loại dữ liệu thu thập, tối ưu chi phí lưu trữ (nhờ Beats/Elastic Agent).
- Khả năng trực quan hóa: Có dashboard để trực quan dữ liệu, giúp quản lý và phân cấp mức độ ưu tiên hiển thị các thông tin qua trọng. 
- Có tính mở rộng cao, dễ dàng đáp ứng được các nhu cầu trong môi trường mạng lớn
- Hệ sinh thái phong phú, có nhiều tích hợp sẵn có giúp tối ưu việc cài đặt và mang lại trải nghiệm tốt.

## Triển khai thực tiễn
### Cách thức triển khai
Mô hình sẽ có thể triển khai trơn tru nếu như có thể host trên cloud tuy nhiên để tối ưu chi phí và chỉ dùng một máy điều hành tất cả thì ở đây ta sẽ chuyển mô hình dưới dạng host như sau:
- Máy Attacker (Kali-Linux), Windows Server(RDP) sẽ được host trên KVM/QEMU.
- Máy SOC Analyst Laptop sẽ chính là máy host. 
- Elastic & Kibana + Fleet Server + Ubuntu Server (SSH) + osTicket Server + C2 server sẽ được host Docker.

Tuy nhiên đối với cách làm này sẽ mắc phải vấn đề giữa 2 cách ảo hóa KVM và Docker, chúng sẽ thực hiện tạo ra các switch mạng ảo khác nhau. Điều nay đi ngược hoàn toàn so với mô hình ban đầu đó là các server đều nằm chung 1 mạng. Để giải quyết việc này ta sẽ điều chỉnh iptables (mở tường lửa) trên máy host để đứng ra để thực hiện chuyển tiếp các gói tin giữa 2 vùng mạng, và biến nó hoạt động tương tự như chúng đang nằm trong một mạng.

### Lưu ý triển khai
Bởi vì Elasticsearch sử dụng mmpafs để lưu các index của nó tuy nhiên nhân linux mặc định giới hạn số lượng bộ nhớ mmap quá thấp nên có thể làm crash chính vì thế ta cần set số này ở mức cao.
![Intercepted Request](assets/img/material_posts/post_1/config_mmap.jpg){: width="800" height="500" }
_Hình 2: Cấu hình cài đặt mmap trên máy host_

### Elasticsearch Set-up
Để cấu hình Elastic ta thực hiện cấu hình trên docker-compose.yml như sau: 
```yml

version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    container_name: elasticsearch
    environment:
      - node.name=es-node-01
      - cluster.name=soc-lab-cluster
      - discovery.type=single-node # Ép chạy chế độ 1 node, không tìm kiếm cluster để nhẹ máy
      - ELASTIC_PASSWORD=SOC_Password_123! # Set cứng mật khẩu cho user 'elastic'
      - xpack.security.enabled=true # Bắt buộc cho Fleet/Agent
      - xpack.security.http.ssl.enabled=false # Tắt TLS để dễ kết nối từ KVM và giảm tải CPU
      - ES_JAVA_OPTS=-Xms2g -Xmx2g # Giới hạn cứng JVM Heap size là 2GB
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - soc_private_net
    deploy:
      resources:
        limits:
          memory: 3G # chống OOM host

volumes:
  es_data:
    driver: local

networks:
  soc_private_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16 # Định hình dải mạng Private

```
Sau khi cấu hình xong file ta chỉ cần chạy lệnh "docker compose up -d" để hoàn tất việc cài đặt Elasticsearch.

### Kibana set-up
```yml
  kibana:
    image: docker.elastic.co/kibana/kibana:8.15.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - SERVER_NAME=soc-kibana
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200 # Khai báo gọi thẳng tên container ES qua mạng nội bộ
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=SOC_Password_123!  # ở đây set tạm thời tránh để credential dạng cleartext trong file.
    networks:
      - soc_private_net
    depends_on:
      - elasticsearch # Đảm bảo ES khởi động xong thì Kibana mới được chạy
    deploy:
      resources:
        limits:
          memory: 1G # Giới hạn tài nguyên sử dụng ram.
```
Kibana cũng là một phần của ELK Stack chính vì thế để tiện lợi ta thực hiện pull Kibana version 8.15, tuy nhiên cần lưu ý rằng đối với Kibana từ bản 8+ sẽ áp dụng nguyên tắc đặc quyền tối thiểu (hay Principle of Least Privilege). Bởi vì Kibana còn nhiều tác vụ chạy ngầm và ghi dữ liệu vào index hệ thống nên nếu sử Superuser (tài khoản elastic) để làm việc này rủi ro leo thang đặc quyền sẽ cao nếu như Kibana bị tấn công. 

Chính vì thế ở phần này ta phải dùng tài khoản chuyên dụng "kibana_system" (đây là tài khoản đã có sẵn nhưng chưa được cài mật khẩu) ta sẽ đặt mật khẩu cho tài khoản thông qua lệnh sau: 
```bash
curl -X POST -u elastic:SOC_Password_123! "http://localhost:9200/_security/user/kibana_system/_password" -H "Content-Type: application/json" -d '{"password":"SOC_Password_123!"}' 
```
Và từ đó ta sẽ login với tài khoản elastic để làm việc như thông thường.

![Intercepted Request](assets/img/material_posts/post_1/KibanaScreen.jpg){: width="800" height="500" }
_Hình 3:Đây là hình ảnh sau giao diện của Kibana khi ta đăng nhập thành công._



## Thực nghiệm tấn công 
## Phân tích cách thức tấn công
## Ngăn chặn và giải pháp
## Bài học rút ra 

