# Deploy-OPS-Ansible
Deploy hệ thống Openstack bằng Ansible

## Khả năng thực hiện
- Deploy hệ thống Openstack Train HA gồm 3 node controller và nhiều node compute
- Deploy hệ thống cho môi trường lab gồm 1 node controlle và nhiều node compute

- Có thể sử dụng 4 interfaces network
## Topology

![ima](ima/OPS-Ansible-Topo.png)

## Các thành phần trong hệ thống
1. Node controller:
 - Chrony
 - Memcache
 - RabbitMQ
 - Mariadb
 - HAproxy
 - Corosync
 - Pacemaker
 - Các project OPS: Keystone, Glance, Placement, Nova, Neutron, Horizon,...
 
2. Node Compute:
 - Chrony
 - Nova-compute
 - Neutron-openvswitch-agent
 
## Yêu cầu cơ bản:
1. Node deployment

- Enable EPEL
```sh
sudo yum install epel-release
```
- Cài đặt pip
```sh
sudo yum install python-pip
```
- Update phiên bản mới nhất cho pip
```sh
 pip install --upgrade pip
```
- Cài đặt Ansible phiên bản 2.8
```sh
pip install ansible==2.8
```
- Cài đặt ipaddr (  Jinja2 filter )
```sh
pip install netaddr
```
- Cài đặt module hỗ trợ việc generate passwords
```sh
pip install flask-uuid-utils

pip install netifaces

pip install oslo.utils
```
- Thực hiện copy ssh-key đến các node cần deploy
```sh
ssh-keygen
ssh-copy-id root@con1
ssh-copy-id root@con2
ssh-copy-id root@con3
ssh-copy-id root@com1
....
```

2. Node Remote
- Các Interfaces cần sử dụng đã có IP 
- Cài đặt MySQL-python (đã có trong playbook)

## Thực thi 
- B1: Vào thư mục chứa playbook
```sh
cd Deploy-OPS-Ansible/
```
- B2: Khai báo các node trong file inventory như [tại đây](https://github.com/vinhducnguyen1708/Deploy-OPS-Ansible/blob/master/multinodeHA) bao gồm:
	
	- tên node muốn đặt tên
	- Khai báo địa chỉ IP để SSH đến **ansible_ssh_host=**

- B3: Thực hiện khai báo các thông số trong file khai báo biến [tại đây](https://github.com/vinhducnguyen1708/Deploy-OPS-Ansible/blob/master/customise.yml). Các thông số cần được khai báo:
  
    - giá trị **IP_VIP**( với mô hình HA thì là ip tùy chọn để truy nhập vào hệ thống OPS, đối với mô hình 1con1com thì đặt IP là IP của controller), **IP_VIP_NETMASK**, **REGION**
    
    - Khai báo các Network Interfaces sử dụng: **MGNT_IF**, **DATA_IF**, **FLAT_IF**

    - Enable các dịch vụ muốn cài ( tùy vào hô hình)
        - Ví dụ: Thực hiện triển khai 1 node controller, 1 hay nhiều node compute thì không cần tới HAproxy và pacemake, nên ta khai báo cho giá trị **enable_haproxy: "no", enable_pacemaker: "no"** hoặc chưa cần dùng đến cinder để tạo volume cho máy ảo khai báo **enable_cinder: "no"**
    
    - Và nếu sử dụng playbook tạo VMs khi cài đặt xong OPS thì khai báo đầy đủ tất cả ở phần **### Create VMs** trong file `/group_vars/all.yml`

- B4: Thực hiện generate ra file `passwords.yml`
```sh
rm -rf passwords.yml

cp passwords.yml.bak passwords.yml

python genpassword.py
```
- B5: Ping kiểm tra đã kết nối tới các host
```sh
ansible -i multinodeHA all -m ping
```
- B6: Thực hiện Deploy hệ thống
```sh
ansible-playbook -i multinodeHA Deploy_OPS_main.yml -e@customise.yml -e@passwords.yml -e my_action=deploy
```
hoặc có thể chạy từng role chỉ định bằng cách thêm các tags ở cuối lệnh
```sh
ansible-playbook -i multinodeHA Deploy_OPS_main.yml -t install_rabbitmq -e@customise.yml -e@passwords.yml -e my_action=deploy
```
- B7: Trên node controller thực hiện lệnh add compute
```sh
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```
- B8: Thực hiện tạo máy ảo sau khi cài xong OPS
    
    ***! LƯU Ý:*** Đã khai báo đầy đủ ở phần **### Create VMs** ( file group_vars/all.yml)

```sh
ansible-playbook -i multinodeHA Create_VMs,yml -e@customise.yml -e@passwords.yml
```


**DONE!**

