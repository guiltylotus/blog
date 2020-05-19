---
layout: post
title: "Load Balancer"
category: system-design
author: lotus
short-description: Tại sao lại là Load Balancer? Thuật toán gì ẩn chứa bên trong? Load Balancer có thể đặt ở layer nào?
---

-----

Load Balancer đóng vai trò rất quan trọng trong thiết kế hệ thống. Vậy:
- Load Balancer là gì? Tại sao các hệ thống lớn cần có Load Balancer?
- Các thuật toán nào được sử dụng cho Load Balancer?
- Load Balancer có thể được đặt ở những layer nào?

### I. Load Balancer là gì?

Với một mô hình client-server đơn giản - chỉ gồm 1 server duy nhất xử lý tất tần tật mọi thứ và đạt được *throughput* là 200 requests/s (1 server không thể đặt được vô hạn request trên 1 giây vì nó phụ thuộc vào tài nguyên mà server đó sở hữu: *CPU, RAM, I/O*). *Throughput* bằng 200 requests/s điều này có nghĩa là nếu có 201 clients gửi requests đến server cùng lúc thì client thứ 201 sẽ phải đợi hơn 1 giây để nhận được response từ server. Điều này rất không hay ho vì chỉ chậm vào giây thôi cũng có thể làm bạn mất đi khách hàng tiềm năng của mình. 

<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_server.jpg" alt="client_server"
        title="Client and server" style="width: 40%"/>
</div>


<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_servers.jpg" alt="client_servers"
        title="Client and servers" style="width: 40%"/>
</div>


Những lợi ích mà Load Balancer mang lại:
- **Hidden server maintenance**: Trải nghiệm trên website của người dùng hoàn toàn không bị ảnh hưởng khi phía server đang thực hiện update version (bằng cách sử dụng *rolling update* )
- **Efficient failure management**: Khi một server trọng cụm bị lỗi, Load Balancer sẽ tách server lỗi ra khỏi cụm để đảm bả0 những requests mới đến được phục vụ bình thường. 
- **Automated scaling**: Việc thêm mới hay gỡ bỏ 1 server ra khỏi cụm hoàn toàn tự động bởi Load Balancer và client hoàn toàn không bị ảnh hưởng.
- **Effective resource management**: Load Balancer sẽ chia đều requests ra các servers để đảm bảo chúng hoạt động hết công suất. Việc bỏ tiền ra mua 5 thằng servers mà 4 thằng làm chăm chỉ còn 1 thằng ngồi không thì thật phí tiền.


### II. Các thuật toán được áp dụng cho Load Balancer
#### 1. Round-Robin

Vào cái thời Load Balancer chưa phổ biến và còn quá đắt đỏ, người ta dùng DNS để chia đều requests đến các servers trong cụm. 

**Round-Robin DNS**

```
example.com 192.168.2.16
example.com 192.168.2.17
example.com 192.168.2.18
```
```
request-1 -> 192.168.2.16
request-2 -> 192.168.2.17
request-3 -> 192.168.2.18
request-4 -> 192.168.2.16

```

Nhược điểm: 
- Do các server mạnh yếu khác nhau, nên việc sử dụng Round-Robin có thể làm 1 server bị quá tải trong khi server khác lại quá nhàn rỗi. Ví dụ, 192.168.2.16 có thông số Core i9 - RAM 16GB - ổ đĩa SSD - xử lý được 8 reuqest trên 1 giây, trong khi 192.168.2.18 CPU Pentium - RAM 2GB - ổ đĩa HĐD xử lý được 1 request trên 1 giây.
- Việc caching trong phương pháp này không được tối ưu. Giả sử client thực hiện 2 request giống nhau trên cùng 1 máy, thì request thứ nhất sẽ được xử lý bởi server 192.168.2.16, còn request thứ hai được thực hiện trên server 192.168.2.17, với cùng request giống nhau nhưng phải query vào DB 2 lần thực sự lẵng phí.
- Việc config nằm ở DNS layer, nên việc cân bằng tải nằm ngoài sự kiểm soát của lập trình viên. Caching trên DNS layer cũng rất khó quản lý.

**Weighted-Round-Robin**
Là một phương pháp cải tiến của Round-Robin, được thực hiện trên Load Balancer. Cho phép đánh trọng số (weight) với mỗi server, những server có trọng số cao hơn sẽ được Load Balancer ưu tiên cho xử lý nhiều request hơn.


#### 2. Random Load Balancing
Chọn ngẫu nhiên một server để xử lý request. Việc này có thể được thực hiện bởi client hoặc server

- **Server-side**: Với mỗi request, Load Balancer sẽ chọn ngẫu nhiên 1 server để xử lý request.
- **Client-side**: Server sẽ cho client biết được IP của những server trong cụm. Việc lựa chọn server nào sẽ do client quyết định.

Nhược điểm:
- Vì là chọn ngẫu nhiên nên 1 server hoàn toàn có thể rơi vào tình trạng quá tải.

#### 3. X-based Hashing
*X-based hasing* ở đây bạn có thể thay thế nó bằng *IP-based Hashing*, *Routing-based Hashing*, hay 1 cái gì đó-based Hashing.

Load Balancer sẽ dựa vào IP của client hoặc url của request để tạo ra 1 mã hash. Mã hash này sẽ quyết định server nào tiếp nhận và xử lý request. Với cùng 1 client IP, dù request nhiều lần cũng đều do 1 server xử lý, điều này đã tái sử dụng được cache 1 cách hiệu quả.

Nhược điểm:
- Khi 1 server bị mất kết nối hay hỏng hóc, Load Balancer sẽ ngắt kết nối với server đó và phải invalid toàn bộ cặp mapping đang map tới server lỗi.

**Consistent Hashing**
1 phương pháp được sinh ra để tối ưu việc tái sử dụng cache. Các bạn có thể xem thêm tại [Consistent Hashing](https://www.youtube.com/watch?v=zaRkONvyGr8) được giải thích bởi anh Gaurav Sen.


### III. Load Balancer được đặt ở những layer nào trong hệ thống?

**Các bạn có thắc mắc như ví dụ ở trên, Load Balancer (website khautrangbaba.com) hoàn toàn có thể bị quả tải khi lượng request nhiều lên không?**

<div class="text-center">
    <img src="{{ site.baseurl }}/assets/load_balancer/client_lbs_server.jpg" alt="client_lbs_server"
        title="Client, Load Balancer and servers" style="width: 50%"/>
</div>


Trên thực tế mỗi service không chỉ có 1 Load Balancer. Các Load Balancer 1, 2 sẽ quản lý các Load Balancer ở layer thấp hơn, tương ứng 1.1, 1.2, 2.1, ..., để đảm bảo **High availability** cho hệ thống. Load Balancer 1 và 2 có thể giao tiếp trực tiếp với nhau để phân chia lượng request. 

Load Balancer có thể được đặt ở rất nhiều layer:
- DNS layer.
- Giữa client và server.
- Giữa server và database.
- Giữa chính những Load Balancer khác. 

