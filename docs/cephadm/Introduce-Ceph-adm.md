# Cephadm

## Introduction

Cephadm đóng vai trò triển khai và quản lý một cụm Ceph, bằng cách kết nối đến các daemon quản lý thông qua SSH. Daemon quản lý có thể `add`, `remove` và `update` các Ceph Containers.
Cephadm không phụ thuộc vào các công cụ tự động hóa cấu hình như Ansible, Rook và Salt.

Cephadm quản lý cả vòng đời của một cụm Ceph. Vòng đời này bắt đâu khi khởi động tiến trình `bootstrapping`, khi Cephadm khởi tạo một cụm ceph nhỏ (trên một node) thì cụm này chỉ bao gồm một thành phần monitor và một thành phần manager. Cephadm sẽ sử dụng giao diện orchestration để mở rộng cụm Ceph, bổ sung thêm các host, daemon và service của Ceph. Việc quản lý vòng đời của cụm Ceph có thể được thực hiện thông qua câu lệnh (CLI) hoặc giao diện (GUI).

Cephadm mới được phát triển từ bản Ceph Octopus (v15.2.0) và không hỗ trợ các phiên bản Ceph cũ hơn.

## Stability

Cephadm là công cụ triển khai mới, đang trong quá trình phát triển vậy nên cần cẩn thận một vài chức năng vẫn đang trong quá trình nâng cấp.
Một vài thành phần có thể thay đổi sắp tới:
- RGW
Cephadm sẽ phát triển các tính năng sau đây:
- Ingress
- Cephadm exporter daemon
- Cephfs-mirror

## Requirement


