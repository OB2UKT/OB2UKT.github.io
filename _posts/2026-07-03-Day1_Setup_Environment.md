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
- Host trên KVM/QEMU: máy Attacker (Kali-Linux), Windows Server(RDP). 
- Máy Host sẽ đóng vai là máy SOC Analyst Laptop. 
- Host trên Docker: Elastic & Kibana + Fleet Server + Ubuntu Server (SSH) + osTicket Server + C2 server.

Tuy nhiên đối với cách làm này sẽ mắc phải vấn đề giữa 2 cách ảo hóa KVM và Docker, chúng sẽ thực hiện tạo ra các switch mạng ảo khác nhau. Điều nay đi ngược hoàn toàn so với mô hình ban đầu đó là các server đều nằm chung 1 mạng. Để giải quyết việc này ta sẽ điều chỉnh iptables (mở tường lửa) trên máy host để đứng ra để thực hiện chuyển tiếp các gói tin giữa 2 vùng mạng, và biến nó hoạt động tương tự như chúng đang nằm trong một mạng.

### Lưu ý triển khai
Bởi vì Elasticsearch sử dụng mmpafs để lưu các index của nó tuy nhiên nhân linux mặc định giới hạn số lượng bộ nhớ mmap quá thấp nên có thể làm crash chính vì thế ta cần set số này ở mức cao.
![Intercepted Request](assets/img/material_posts/post_1/config_mmap.jpg){: width="800" height="500" }
_Hình 2: Cấu hình cài đặt mmap trên máy host_

### Elasticsearch Set-up
Để cấu hình Elastic ta thực hiện cấu hình trên docker-compose.yml như sau: 
```yml

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
_Hình 3: Đây là hình ảnh sau giao diện của Kibana khi ta đăng nhập thành công._


### Cấu hình Firewall & Iptables
Như đã đề cập trước đó ta sẽ có kiến trúc và phân vùng mạng sẽ bao gồm: 
- SOC Management Zone: Kibana & Elasticsearch and osTicket Server, Fleet Server (The Bridge -  cầu nối mở cổng 8220 để nhận log từ các máy khác). 
- Target Zone: Windows Server and Ubuntu Server.
- Internet (Untrusted Zone): SOC Analyst Laptop, C2 Server, Attacker Laptop.

Vấn đề host trên nhiều nền tảng khác nhau như Docker, KVM đã gây ra vấn đề lớn về việc không cùng nằm chung 1 mạng bởi vì Docker sử dụng container ảo hóa với card mạng **br-bd90203b279c - 127.20.0.0/16** và KVM sử dụng card mạng **virbr0 - 192.168.122.0/24**, nếu như kiếm cách giao tiếp giữa 2 vùng mạng ảo hóa này sẽ gặp rất nhiêu khó khăn và lỗi vì bản chất chúng đều có cơ chế Network Isolation và tự động DROP/REJECT các traffic đi tới. 

Giải pháp ở đây thay vì ta tìm cách kết nối giữa 2 vùng mạng thì ta sẽ dùng máy host chính đứng ra làm trung gian cho sự giao tiếp giữa 2 phân vùng mạng ảo hóa, về bản chất khi ta khởi tạo 1 port mở trên docker thì cũng chính máy host sẽ lắng nghe port đó (VD: docker có port mở 172.20.0.1:8220 --> trên máy host cũng mở port 8220 để lắng nghe mọi địa chỉ ip). Thay vì gửi trực tiếp các traffic từ máy ảo (KVM) đến đích (docker) ta có thể đi vòng sang con đường máy host bằng cách gửi trực tiếp cho IP Gateway của nó chính là 192.168.122.1:8220. Khi gói tin được gửi đến host nó sẽ tự động đi vào PREROUTING của Iptable để thực hiện cơ chế DNAT (Destination Network Address Translation) của docker, tiến hành nhận gói tin và xóa địa chỉ đích của gói tin 192.168.122.1 --> 172.20.0.1:8220 và cuối cùng quy trình phản hồi cũng tương tự.

#### Cấu hình UFW (Firewall)

```bash
# Mở traffic từ mạng ảo KVM virbr0 
sudo ufw allow in on virbr0

# Mở cổng Control Plane (Fleet Server - 8220) và Data Plane (Elasticsearch - 9200)
sudo ufw allow 8220/tcp
sudo ufw allow 9200/tcp

# Bật tính năng Forwarding, cho phép gói tin chui vào mạng Docker.
sudo sed -i 's/DEFAULT_FORWARD_POLICY="DROP"/DEFAULT_FORWARD_POLICY="ACCEPT"/g' /etc/default/ufw
sudo ufw reload

# Hoặc đơn giản hơn nhưng không thực tiễn chỉ phụ vụ trong lab, chúng ta có thể tắt tạm UFW (Không khuyến cáo trong thực tế).
sudo ufw disable
```

#### Cấu hình iptable

```bash
# Đặt rule ở mức ưu tiên cao nhất để cho phép trafic từ KVM được xử lý.
sudo iptables -I DOCKER-USER 1 -i virbr0 -j ACCEPT
sudo iptables -I DOCKER-USER 1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Sau khi cấu hình thành công ta đã đáp ứng được việc các agent nằm trên các máy host có thể gửi trực tiếp log trực tiếp về Elastic Search (port 9200), ngoài ra các agent này phải bị kiểm soát, quản lý thông qua kết nối với Fleet Server (port 8220).

### Fleet Server set-up
Fleet Server (Control Plane) được thiết lập để quản lý toàn bộ các Elastic Agent được cài đặt trên các máy host. Bản chất nó vẫn là một elastic-agent thông thường, tuy nhiên nó sẽ không đọc logs và đổ về cho Kibana mà ngược lại nó lắng nghe ở port 8220 để quản lý, nhận kết nối từ các Agent khác đổ về. Fleet Server có quyền hành cao dùng để cấp phát API key, quản lý trạng thái của các agent, cấu hình các agent. 

Fleet Server ra đời nhằm giải quyết nhu cầu: 
- Về quản lý cấu hình tập trung với quy mô lớn (sửa đồng bộ và áp dụng chính sách cấu hình lên toàn bộ agent quản lý).
- Giảm tải và đứng ra gánh vác vai trò thay cho Elasticsearch/Kibana, thay vì để các dịch vụ này đón các connection khổng lồ từ nhiều agent.
- Bảo mật hơn, tránh lateral movement. Fleet server chỉ thực hiện cấp API key độc lập, duy nhất và giới hạn quyền tối thiểu, nếu như agent bị khai thác và lấy được API key này cũng không thể lây lan hay tấn công các agent khác.

Đối với chính sách mới nhất của Elasitc Search ở đây yêu cầu ta phải dùng theo chuẩn HTTPS để đảm bảo an toàn tuy nhiên dưới góc độ là một bài lab cá nhân và chạy hoàn toàn local thì tôi đánh giá hướng đi này khá phức tạp (mặc dù điều này thực tế). Ngoài ra việc sử dụng HTTPS sẽ tăng thêm chi phí và dễ gặp phải một số thách thức khi ta chạy lab ở mạng nội bộ thì phải dùng Self-signed Certificate (Chứng chỉ tự ký) thay vì sử dụng các chứng chỉ public dẫn đến các trường hợp ngắt kết nối tự động/lỗi (x509: certificate signed by unknown authority). Chính vì thế trong phạm vị lab này ta sẽ sử dụng HTTP và thực hiện một số thủ thuật để đánh lừa và bypass cơ chế này của Elastic Search (hành động này không khuyến cáo chỉ phục vụ cho mục đích bài lab).

Đầu tiên ta sẽ thực hiện cấu hình Fleet Server như thông thường và ta xác định bước verify phía client-side.

![Intercepted Request](assets/img/material_posts/post_1/config_fleetServer.jpg){: width="800" height="500" }
_Hình 4: Cấu hình chi tiết của Fleet Server trên Kibana._

Ở bước này ta sẽ sử dụng Burpsuite (Intercept mode) để can thiệp và sửa nội dung gói tin trước khi nó được truyền tới và xử lý ở back-end.

![Intercepted Request](assets/img/material_posts/post_1/change_config_fleetServer_1.jpg){: width="800" height="500" }
_Hình 5: Đây là hình ảnh mô tả giai đoạn sửa nội dung gói tin (method PUT)._

Sau bước này ta sửa về http và foward đi thì ta đã hoàn toàn có thể sửa đổi được ip như mong muốn.

![Intercepted Request](assets/img/material_posts/post_1/change_config_fleetServer_success.jpg){: width="800" height="500" }
_Hình 6: Kết quả thực tế._


### Cấu hình Windows Server 2022 
Ở phiên bản này ta lựa chọn Windows Server 2022, tuy không là phiên bản quá mới nhưng đối với phiên bản này hội tụ đủ những yếu tố về bảo mật, tích hợp được agent, được phiên bản ELK version 8.15.5 support tốt, ngoài ra còn đáp ứng tiêu chí tiết kiệm RAM. Đối với phần này ta sẽ tiến hành thực hiện cài đặt cấu hình sysmon, cài đặt agent lên máy và thực hiện kết nối agent tới Fleet server cũng như gửi logs về Elasticsearch.

#### Cài đặt Sysmon 
Sysmon (System Monitor) là công cụ thuộc bộ tiện ích Sysinternals của Microsoft. Khi được cài đặt trên hệ thống Windows, nó hoạt động dưới dạng một Windows service và một device driver ở tầng Kernel. Nhiệm vụ chính là theo dõi, giám sát các hoạt động ở mức hệ thống (system activity) và ghi lại toàn bộ thông tin chi tiết này vào Windows Event Log. Ta cần cài đặt thêm Sysmon để phục vụ tốt hơn cho việc phân tích logs, bởi vì Windows Event Log thông thường có một số nhược điểm sau:
- Thiếu ngữ cảnh của tiến trình về mã băm, mặc dù Event ID 4688 (Process Creation) mặc định của Windows không tự tính toán mã băm của tệp thực thi. Việc xác minh tính toàn vẹn của một file nghi ngờ chỉ dựa vào log mặc định là không khả thi.
- Đứt gãy chuỗi ngữ cảnh mạng, đối với Event ID 5156 (Windows Filtering Platform) mặc định có sự ghi nhận các kết nối mạng tuy nhiên vô tình tạo ra một lượng rác lớn và khó để đối chiếu/truy vấn trực tiếp mạng hoặc tiến trình cụ thể đã bắt nguồn từ đâu.
- Mù trước các kỹ thuật tấn công filess-malware.

Tiếp theo ta đến với các bước cài đặt Sysmon. 

![Intercepted Request](assets/img/material_posts/post_1/sysmon_v1521_in4.jpg){: width="800" height="500" }
_Hình 7: Mô tả chi tiết gói Sysmon._

Sau đó ta tiến hành giải nén và thực hiện các bước cài đặt tuần tự.

![Intercepted Request](assets/img/material_posts/post_1/install_Sysmon.jpg){: width="800" height="500" }
_Hình 8: Chạy lệnh cài đặt Sysmon._

Ở đây ta sử dụng bộ rule được thiết lập sẵn (sysmonconfig.xml) từ github sysmon-modular, đây là tập hợp các rules được viết sẵn, khi Sysmon đọc đực file này thì tiến hành xử lý chính xác những đối tượng cần bắt và bỏ qua dựa trên những hoạt động trên hệ thống Windows. 

Bộ rules có sẵn giúp giải quyết được bài toán về số logs lượng lớn và cách phân tích logs tối ưu để giảm nhiễu, được cập nhật thường xuyên, tránh false-positves và outdate.

![Intercepted Request](assets/img/material_posts/post_1/config_sysmon_windowServer2022.jpg){: width="800" height="500" }
_Hình 9: Thiết lập Sysmon chạy với bộ rule đã có sẵn._


![Intercepted Request](assets/img/material_posts/post_1/result_after_config_sysmon.jpg){: width="800" height="500" }
_Hình 10: Sau khi thiết lập thành công Sysmon._

#### Cài đặt Agent
Các agent này được cài trên máy host nhằm mục đích lắng nghe logs trên máy host và tiến hành gửi về cho Elasticsearch để làm tài nguyên phân tích. Chính vì thế ở phần này ta sẽ có 2 mục tiêu chính là thiết lập kết nối về Elasticsearch (port 9200) để gửi logs và kết nối tới Fleet Server (port 8220) để thực hiện quản lý chính agent.

Để làm được điều này ta tiến hành cài đặt elastic và cài đặt với các tham số như địa chỉ fleet-servr, token, policy, port. Khi chạy thành công ta sẽ thấy được **"Elastic Agent has been successfully installed"**.

```bash
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.15.0-windows-x86_64.zip -OutFile elastic-agent-8.15.0-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.15.0-windows-x86_64.zip
cd elastic-agent-8.15.0-windows-x86_64
.\elastic-agent.exe install `
  --fleet-server-es=http://192.168.122.1:9200 `
  --fleet-server-service-token=TOKEN_HIDDEN_WHEN_DISPLAY `
  --fleet-server-policy=fleet-server-policy `
  --fleet-server-port=8220
```

#### Cài đặt Integration
Sau việc cấu hình agent trên máy host, ta cần cấu hình thêm integration để đáp ứng các nhu cầu sau: 
- Chọn, chỉ thị được kênh dữ liệu, điểm quan trọng cần phân tích (trong bài lab này là Windows Defender & Sysmon)
- Chuẩn hóa dữ liệu tại chỗ và ép kiểu, bởi vì tên gọi trên mỗi hệ điều hành là khác nhau chính vì thế, integration sẽ giúp chuẩn hóa (đổi tên, ép biến thể theo chuẩn ECS duy nhất), điều này giúp dễ truy vết và đảm bảo đồng nhất khi truy vấn logs, săn lùng dấu vết trên nhiều hệ điều hành nền tảng khác nhau.
- Xử lý, lọc và tối ưu hóa băng thông. Ta hoàn toàn có thể lọc logs ngay tại phía host, ta xử lý và giới hạn những logs nguy hiểm (logs quan trọng) sẽ thu thập và loại bỏ những logs nhiễu (logs mang tính chất thông báo), điều này giảm đáng kể chi phí lưu trữ và truyền tải.

Ta thực hiện cấu hình 2 integration chính cho agent trên host windows

![Intercepted Request](assets/img/material_posts/post_1/config_integration_windows.jpg){: width="800" height="500" }
_Hình 11: Loại Integration ta chọn để thiết lập trên host Windows._

Chi tiết cấu hình lần lượt như sau: 

![Intercepted Request](assets/img/material_posts/post_1/config_windowsDefender_integration_1.jpg){: width="800" height="500" }
_Hình 12: Chi tiết cấu hình integration để thu thập logs từ Windows Defender._

![Intercepted Request](assets/img/material_posts/post_1/config_windowsDefender_integration_2.jpg){: width="800" height="500" }
_Hình 13: Liệt kê các EventID sẽ được nhận đối với logs từ Windows Defender._


![Intercepted Request](assets/img/material_posts/post_1/config_windowsEventLogs_integration.jpg){: width="800" height="500" }
_Hình 14: Chi tiết cấu hình integration Sysmon._

Sau khi cài đặt xong 2 integration trên ta có thể thấy được kết quả sau, từ 2 integration trên ta đã thực hiện thu thập và phân tích chi tiết tập trung vào logs của sysmon và integration.

![Intercepted Request](assets/img/material_posts/post_1/result_after_instal_Integration.jpg){: width="800" height="500" }
_Hình 15: Kết quả integration của host Windows._

Ta thử tắt chế độ Real-time protection của Windows Defender để kiểm tra khả năng bắt logs của agent.

![Intercepted Request](assets/img/material_posts/post_1/alert_turnoff_windows_defender.jpg){: width="800" height="500" }
_Hình 16: Khả năng bắt logs khi phát hiện sự thay đổi mang tính nguy hiểm cao._

### Cấu hình Ubuntu Server 24.04
Tiếp theo ta sẽ lựa chọn Ubuntu 24.04 để làm Ubuntu Server mục tiêu, cũng giống như tiêu chí của Windows ta sẽ lựa chọn phiên bản được ELK hỗ trợ và tối ưu RAM nhất có thể nên ta sẽ lựa chọn phiên bản vừa đủ vì Ubuntu Server không chạy giao diện mà hoàn toàn là CLI nên ta cấu hình máy ảo chạy với 1GB RAM.

#### Cài đặt Agent - Ubuntu
Tương tự với việc cài đặt agent trên host của Windows. Ta chỉ cần chạy lệnh đã có sẵn của Fleet Server cấp là có thể tải agent về máy và thực hiện kết nối tới Fleet server và gửi logs về Elasticsearch. Agent nằm trên Ubuntu server chủ yếu tập trung theo dõi những thay đổi nằm trong /var/log/auth.log để báo cáo về Elasticsearch về những trạng thái truy cập bất thường, brute-force, authentication failure.

### Cấu hình C2 Server - Mythic Framework

![Intercepted Request](assets/img/material_posts/post_1/mythic_logo.jpg){: width="800" height="500" }
_Hình 17: Hình ảnh về Mythic Framework._

Mythic là một framework **Command and Control (C2)** mã nguồn mở thiết kế phục vụ cho chiến dịch RedTeam và giả lập tấn công, Mythic nổi bật với kiến trúc nhiều module linh hoạt và nhiều kỹ thuật tấn công tinh vi trong thực tế, việc áp dụng framework này vào bài lab hiện tại với mong muốn đem lại trải nghiệm thực tế nhất. Ở mô hình này để có thể Exfilltration sensitive data từ máy mục tiêu (Windows Server 2022) thì ta sẽ lựa chọn agent Apollo.

Một số thứ điều thú vị về Mythic: 
- Khả năng giao tiếp linh hoạt và tàng hình, nhằm tránh hệ thống tường lửa phát hiện Mythic cho phép điều hướng C2 qua nhiều giao thức HTTP/HTTPS/DNS/WebSockets/SMB, ngoài ra còn cho phép tùy chỉnh header và cấu trúc gói tin để mô phỏng giống như 1 lưu lượng hợp lệ.
- Mythic khi được host lên sẽ cho phép nhiêù người dùng truy cập vào để tương tác và khai thác máy xâm nhập, có thể chia sẽ và theo dõi tiến độ chiến dịch. 
- Mythic tích hợp sẵn các kỹ thuật MITRE ATT&CK, mọi hành động và lệnh thực thi trong Mythic đều được gắn thẻ mapping theo khung này. Giúp cho Blue Team dễ dàng đối chiếu và báo cáo sau chiến dịch.


Để có thể cài đặt được Mythic thì yêu cầu ban đầu của chúng ta cơ bản cần phải có Docker và công cụ make. Chi tiết quá trình cấu hình như sau:

![Intercepted Request](assets/img/material_posts/post_1/c2_config_make_result.jpg){: width="800" height="500" }
_Hình 18: Chi tiết log cài đặt Mythic._

![Intercepted Request](assets/img/material_posts/post_1/install_mythic_3.jpg){: width="800" height="500" }
_Hình 19: Cài đặt Mythic._

![Intercepted Request](assets/img/material_posts/post_1/install_C2_agent_apollo_4.jpg){: width="800" height="500" }
_Hình 20: Cài đặt Apollo Agent._

![Intercepted Request](assets/img/material_posts/post_1/install_c2Profile_5.jpg){: width="800" height="500" }
_Hình 21: Cài đặt C2 profile._

![Intercepted Request](assets/img/material_posts/post_1/Result_after_config_package_6.jpg){: width="800" height="500" }
_Hình 22: Giao diện sau khi cài đặt thành công._


### Cấu hình osTicket

![Intercepted Request](assets/img/material_posts/post_1/osticket-logo_dark_logo.jpg){: width="800" height="500" }
_Hình 23: Logo osTicket._

Trong bối cảnh số lượng logs lớn, mặc dù ELK Stack đã cho phép ta cài đặt alert để thông báo những bất thường khi có logs bất thường xuất hiện. Tuy nhiên vấn đề kiểm soát và xử lý các alert này lại tiếp tục đặt ra bài toán lớn cần được giải quyết, một hành vi bất thường cần được báo cáo chuẩn xác tại thời điểm xảy ra để phục vụ cho đội Incident Response và kiểm soát được vòng đời của alert đã trải qua những giai đoạn nào đối với SOC Analyst . Nếu cập nhật trạng thái của alert qua mỗi khâu xử lý một cách thủ công (qua email hay nền tảng liên lạc cá nhân) sẽ mang lại nhiều bất cập và kèm theo những rủi ro, sai sót ngoài ra còn thiếu đồng bộ và thất thoát. Gây ảnh hướng tới hiệu suất xử lý và rối quy trình. 

Chính vì thế osTicket Server là giải pháp được đưa ra để khắc phục những hạn chế và mang lại giá trị như:
- Phân loại log dựa trên trình tự thời gian và mức độ impact, giúp xác định thứ tự ưu tiên xử lý.
- Cập nhật trạng thái, giúp quản lý vòng đời của alert (Vd: New -> In progress -> Resolved -> Closed) ngoài.
- Phân quyền, phân chia nhiệm vụ và trách nhiệm cho chính ticket và tránh xử lý trùng lập công việc và thiếu minh bạch.
- Lưu trữ các thao tác điều tra, log liên quan và phương án khắc phục, những ghi chú xử lý và thao tác giải quyết sẽ được lưu lại để phục vụ cho hậu kỳ và xây dựng cẩm nang xử lý các trường hợp tương tự trong tương lai.

Ở phần này ta thực hiện cài đặt 1 máy ảo Windows Server 2022 chạy dịch vụ XAMPP và cài đặt osTicket. Chi tiết quá trình cài đặt như sau:

![Intercepted Request](assets/img/material_posts/post_1/config_osTicket_installer.jpg){: width="800" height="500" }

![Intercepted Request](assets/img/material_posts/post_1/config_osTicket_installer_2.jpg){: width="800" height="500" }
_Hình 24: Chi tiết cấu hình osTicket._

![Intercepted Request](assets/img/material_posts/post_1/result_after_install_osticket.jpg){: width="800" height="500" }
_Hình 25: Kết quả sau khi cài đặt osTicket._

#### osTicket Collab With ELK Integration
Ở ELK (phiên bản trả phí) chúng ta được phép kết nối đến các Connector như Gmail, Teams,... các platform khác để có thể gửi đi alert trên các nền tảng đó. Ở trong bài lab này ta có thể sử dụng free-trial để có thể thực hiện tạo kết nối tới osTicket mà ta đã host sẵn ở bước trên để tiến hành thu thập và quản lý các ticket.

Trong phần này ta sẽ chú trọng đến việc cài đặt và thiết lập kết nối từ Kibana đến osTicket để có thể gửi cảnh báo đến . Ta sẽ thực hiện các bước cài đặt như sau:

![Intercepted Request](assets/img/material_posts/post_1/Initital_osTicket_alert_on_kibana.jpg){: width="800" height="500" }
_Hình 26: Cấu hình đểtạo kết nối tới osTicket._

Để có thể kiểm tra khả năng tạo ticket và gửi tới osTicket từ Kibana ta thử gửi đi 1 gói tin dạng .xml dựa trên một form đã chuẩn bị sẵn.

![Intercepted Request](assets/img/material_posts/post_1/Test_Alert2osTicket.jpg){: width="800" height="500" }
_Hình 27: Thực hiện tạo gói tin để gửi tới osTicket._

![Intercepted Request](assets/img/material_posts/post_1/take_ticket_from_Kibana.jpg){: width="800" height="500" }
_Hình 28: Gói tin được nhận và chuyển tới phần quản lý của osTicket._



