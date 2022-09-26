# Install Rados Gateway - Ceph Ansible

## Cấu hình Ansible để triển khai Rados Gateway
- Cấu hình trong Inventory:
```ini
[mons]
ceph-ms-mon ansible_host=192.168.50.37 ansible_port=22

[mgrs]
ceph-ms-mon ansible_host=192.168.50.37 ansible_port=22

[osds]
ceph-ms-mon ansible_host=192.168.50.37 ansible_port=22
ceph-ms-osd01 ansible_host=192.168.50.23 ansible_port=22
ceph-ms-osd02 ansible_host=192.168.50.33 ansible_port=22
ceph-ms-osd03 ansible_host=192.168.50.34 ansible_port=22

### Thêm group và host dưới đây làm rados gateway
[rgws]
ceph-ms-mon ansible_host=192.168.50.37 ansible_port=22
###
```

- Bổ sung biến chọn đường gọi HTTP tới rados Gateway, biến này có thể bổ sung tại `group_vars` hoặc `extra_vars`:
```yml 
radosgw_interface: "eth0"
```

