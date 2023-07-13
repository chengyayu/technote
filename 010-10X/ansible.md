# Ansible

## 1.配置文件读取顺序

ansible.cfg 文件是 Ansible 中最主要的配置文件，ansible 寻找配置文件按照如下的优先级进行：

1. 由环境变量 ANSIBLE_CONFIG 指定的文件
2. ./ansible.cfg (ansible.cfg in the current directory)
3. ~/.ansible.cfg (.ansible.cfg in your home directory)
4. /etc/ansible/ansible.cfg

最简单的 ansible.cfg 配置示例：
```
[defaults]
hostfile = hosts
remote_user = root
remote_port = 22
host_key_checking = False
```

说明：
hostfile 文件指定了当前文件夹下的 hosts 文件。hosts 文件中会配置需要管理的机器 host
配置 SSH 免密登录的文章可以参考之前的文章.
remote_user 配置默认操作的用户，如果没有配置，默认会使用当前用户
host_key_checking: 禁用 SSH key host checking
