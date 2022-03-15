# Cài đặt Ceph cluster bằng Ceph-Ansible

## Chuẩn bị
- 4 node: ansible-deployment, ceph1, ceph2, ceph3
- HĐH: Centos7
- Interfaces: eth0 (192.168.10.0/24),eth1, eth2, eth3( 192.168.40.0/24)
- Mỗi node có 3 ổ: vda( dùng cài os), vdb, vdc

## Thành phần

- **mon**: ceph1, ceph2, ceph3
- **mgr**: ceph1
- **osd**: ceph1, ceph2, ceph3
- **grafana-server**: ceph1

## Thực hiện trên node ansible-deployment

- Bước 1: Cài đặt Ansible
```sh
sudo yum install epel-release

sudo yum install python-pip

pip install --upgrade pip
 
pip install ansible==2.8

pip install netaddr
``` 

- Bước 2: thực hiện clone repo ceph-ansible trên github
```sh
git clone https://github.com/ceph/ceph-ansible.git
```
- Bước 3: Chọn branch 
```sh
cd ceph-ansible

git checkout stable-4.0
```

- Bước 4: Tạo file inventory trong thư mụng ceph-ansible
```ini
cat << EOF > /root/ceph-ansible/ceph_inventory
[mons]
ceph1 ansible_host=192.168.10.40
ceph2 ansible_host=192.168.10.41
ceph3 ansible_host=192.168.10.42

[osds]
ceph1 ansible_host=192.168.10.40
ceph2 ansible_host=192.168.10.41
ceph3 ansible_host=192.168.10.42

[mgrs]
ceph1 ansible_host=192.168.10.40

[grafana-server]
ceph1 ansible_host=192.168.10.40
EOF
```
- Bước 5: Copy file playbook và file biến để sử dụng
```sh
cp site.yml.sample site.yml
cp group_vars/all.yml.sample group_vars/all.yml
```

- Bước 6: Khai báo biến tùy chọn cho playbook
```yml
vi /root/ceph-ansible/group_vars/all

ceph_origin: repository
ceph_repository: community
ceph_repository_type: cdn
ceph_stable_release: nautilus
monitor_interface: eth0
public_network: 192.168.10.0/24
cluster_network: 192.168.10.0/24

devices:
  - '/dev/vdb'
  - '/dev/vdc'
dashboard_enabled: yes
configure_firewall: yes
openstack_config: yes
```

- Bước 7: Copy key ssh đến các node trong file inventory
```sh
ssh-keygen

ssh-copy-id root@192.168.10.40
ssh-copy-id root@192.168.10.41
ssh-copy-id root@192.168.10.42
```

- Bước 8: Ping kiểm tra
```sh
ansible -i ceph_inventory all -m ping
```

- Bước 9: Thực hiện deploy cluster
```sh
ansible-playbook -i ceph_inventory site.yml -u root
```