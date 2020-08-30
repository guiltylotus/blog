---
layout: post
title: "Cloud Storage System"
category: system-design-questions
author: lotus
short-description: Thiết kế hệ thống S3 của Amazon
---

-----
# Câu hỏi
Thiết kế hệ thống lưu file tương tự như S3 của Amazon

- Yêu cầu cơ bản:
    - Người dùng có thể upload và download file từ hệ thống
    - Là 1 hệ thống có số lượng file và ảnh lớn

- Giả thiết bổ sung:
    - Tỷ lệ đọc/viết của hệ thống là 1:1
    - Hệ thống có 500tr người dùng và có 100tr người dùng thường xuyên hoạt động
    - Trung bình có 1tr requests mỗi phút

# Phân tích câu hỏi

# High-level design

<div class="text-center">
    <img src="{{ site.baseurl }}/assets/caching/cahcing.png" alt="client_servers"
        title="Client and servers" style="width: 100%"/>
</div>

