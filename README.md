# k8s-ansible
k8s的contoller-manager,scheduler,apiserver核心组件是通过kubelet启动的静态pod来实现的，ansible会把几个组件的安装yaml文件放在/etc/
kubernetes/manifest下面，kubelet启动后会自动安装
## 1. 修改inventory/hosts.ini,我是在virtualbox下安装centos7，用两台机器来安装，
注意ip地址，我的虚拟机装了两块网卡，一个是nat模式，网卡为enp0s3, ip为10.0.2.15，一个是bridge模式，网卡为enp0s8，ip为192.168.56.102
nat模式下虚拟机之间网络无法访问，一定要选择bridge的ip，etcd，apiserver，flannel，calico等组件的iface网口一定要设置成enp0s8！
master 192.168.56.102
node1 192.168.56.103

```sh
[etcds]
master

[masters]
master

[nodes]
node1

[kube-cluster:children]
nodes
```

## 2. 配置roles/cluster-default/defaults/main.yml
如1所述，凡是涉及到iface的地方，都需要改成enp0s8
```sh
# etcd extra variables.
etcd_iface: "enp0s8"
etcd_domain_name: test.etcd.com
```

```sh
# Supported: calico, flannel.
cni_enable: true
container_network: calico
cni_iface: "enp0s8"
```

## 3. 安装cluster-cert
```sh
ansible-playbook -i inventory/hosts.ini cluster-etcd.yml 
```

## 4. 安装cluster-download
因为下载比较费时，单独做了一个task

```sh
ansible-playbook -i inventory/hosts.ini cluster-download.yml 
```