# ch9

管理員工作自動化

基本上我們如果要管理被管理主機身上軟體,那我們會使用dnf模組來幫助我們來管理
例如我們要安裝軟體name 表示名稱'state標示present,表示從無到有

```markdown
    - name: install package
      ansible.builtin.dnf:
        name: '*'
        state: present
```

```markdown
    - name: remove package
      ansible.builtin.dnf:
        name: '*'
        state: absent
```

還有更新,name使用'*',宣告該管理主機身上所有的套件,state宣告latest標示到最新版

```markdown
    - name: update package
      ansible.builtin.dnf:
        name: '*'
        state: latest
```

移除軟體name 表示名稱'state標示absent,表示如果有該軟體在主機身上,那就把他移除

```markdown
[student@hosta ansible]$ cat remove_file.yml 
---
- name: file module remove file example
  hosts: hostb.lab.example.com
  become: true

  tasks:
    - name: remove file 
      ansible.builtin.file:
        path: /home/exampl_file.txt
        state: absent
```

透過迴圈的寫法,同時安裝三個軟體

```markdown
  tasks:
    - name: install package
      ansible.builtin.dnf:
        name:
          - php
          - mariadb-server
          - httpd
        state: present
```

透過facts將被管理主機身上安裝的軟體資訊抓回來

如最近的ssh 使用到 xz某些特定版本的函式庫有重大漏洞 

```markdown
[student@hosta ansible]$ cat host_package_info.yml 
---
- name: package info from node 
  hosts: all
  become: true

  tasks:
    - name: capture manage node pacakage info
      ansible.builtin.package_facts:
        manager: auto

    - name: display package info
      ansible.builtin.debug:
        var: ansible_facts.packages.xz.version
      ignore_errors: yes
      
    - name: Check whether a package called foobar is installed
      ansible.builtin.debug:
        msg: "The host xz package version is {{ ansible_facts.packages.xz }} "
      when: "'xz' in ansible_facts.packages "
```

執行結果:

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout host_package_info.yml -C

PLAY [package info from node] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [hosta.lab.example.com]
ok: [hostb.lab.example.com]
ok: [hostd.lab.example.com]
ok: [hostc.lab.example.com]

TASK [capture manage node pacakage info] ***************************************
ok: [hostd.lab.example.com]
ok: [hostc.lab.example.com]
ok: [hostb.lab.example.com]
ok: [hosta.lab.example.com]

TASK [display package info] ****************************************************
ok: [hosta.lab.example.com] => {
    "ansible_facts.packages.xz.version": "VARIABLE IS NOT DEFINED!"
}
ok: [hostb.lab.example.com] => {
    "ansible_facts.packages.xz.version": "VARIABLE IS NOT DEFINED!"
}
ok: [hostc.lab.example.com] => {
    "ansible_facts.packages.xz.version": "VARIABLE IS NOT DEFINED!"
}
ok: [hostd.lab.example.com] => {
    "ansible_facts.packages.xz.version": "VARIABLE IS NOT DEFINED!"
}

TASK [Check whether a package called foobar is installed] **********************
ok: [hosta.lab.example.com] => {
    "msg": "The host xz package version is [{'name': 'xz', 'version': '5.2.4', 'release': '4.el8', 'epoch': None, 'arch': 'x86_64', 'source': 'rpm'}] "
}
ok: [hostb.lab.example.com] => {
    "msg": "The host xz package version is [{'name': 'xz', 'version': '5.2.4', 'release': '4.el8', 'epoch': None, 'arch': 'x86_64', 'source': 'rpm'}] "
}
ok: [hostc.lab.example.com] => {
    "msg": "The host xz package version is [{'name': 'xz', 'version': '5.2.4', 'release': '4.el8', 'epoch': None, 'arch': 'x86_64', 'source': 'rpm'}] "
}
ok: [hostd.lab.example.com] => {
    "msg": "The host xz package version is [{'name': 'xz', 'version': '5.2.4', 'release': '4.el8', 'epoch': None, 'arch': 'x86_64', 'source': 'rpm'}] "
}

PLAY RECAP *********************************************************************
hosta.lab.example.com      : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostb.lab.example.com      : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostc.lab.example.com      : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostd.lab.example.com      : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

那畢竟我們不是dnf模組可以對應所有的作業系統,如ubuntu就是使用apt來管理軟體,還有windows所使用的軟體套件就不是dnf
因此這種時候我們會使用package模組,他會偵測該遠端主機身上是那個作業系統版本,並根據是和該作業系統的方式安裝軟體

```markdown
[student@hosta ansible]$ cat package_module.yml 
---
- name: pacakage example
  hosts: all
  become: true

  tasks:
    - name: Install ntpdate
      ansible.builtin.package:
        name: httpd
        state: present 
```

針對rpm體系的主機,我們會有所謂的yum 軟體安裝來源repository
我們今天要確認被管理主機上有正確的安裝/可用的來源,要使用yum_repository
name檔案名稱'description 解說'file檔案要放置的目錄'enable決定這個安裝來源要不要啟用

```yaml
     - name: Add AppStream repositories into the same file (2/2)
       ansible.builtin.yum_repository:
         name: AppStream
         description: AppStream YUM repo
         file: /etc/yum.repos.d/redhat-AppStream.repo
         baseurl: https://mirror01.idc.hinet.net/CentOS/8-stream/AppStream
         enabled: yes
         gpgkey: https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official
         gpgcheck: true

```

結果：

```markdown
[AppStream]
baseurl = https://mirror01.idc.hinet.net/CentOS/8-stream/AppStream
enabled = 1
gpgcheck = 1
name = AppStream YUM repo
```

import gpgkey
匯入gpg金鑰驗證安裝來源

```markdown

- name: Import a key from a file
  ansible.builtin.rpm_key:
   key: https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official
   state:present
```

管理使用者與身份驗證
建立使用者,將devops加入manager與root群組

```markdown
   - name: create user fro manager
      ansible.builtin.user:
        name: devops
        shell: /bin/bash
        groups: root,manager
        append: yes
```

那其中關於使用者的密碼,參考How do I generate encrypted passwords for the user module 章節
建議使用{{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"
因此將使用者建立的編寫方式改為

```yaml
  vars_files:
    - user_list.yml
  tasks:
   - name: create user fro manager
      ansible.builtin.user:
        name: devops
        shell: /bin/bash
        groups: root,manager
        append: yes
        password: "{{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"

```

透過user模組取得user模組的ssh key 

```yaml
- name: Create a 2048-bit SSH key for user jsmith in ~jsmith/.ssh/id_rsa
  ansible.builtin.user:
    name: devops
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa
```

建立群組

```yaml
    - name: create group
      ansible.builtin.group:
        name: manager
        state: present
```

known_hosts 模組

他用來管理新增/移除 ssh key,即代表known_hosts
但若如果有大量的主機金鑰進行管理,建議使用template模組

```yaml
- name: Set authorized key taken from file
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
```

確保使用者,不需要輸入密碼
使用lineinfile把提權且不需要的賬號/群組加入/etc/sudoers之中

```yaml
- name: Validate the sudoers file before saving
  ansible.builtin.lineinfile:
    path: /etc/sudoers
    state: present
    regexp: '^%ADMIN ALL='
    line: '%ADMIN ALL=(ALL) NOPASSWD: ALL'
```

管理開機程序與排程
使用at 模組在20分鍾後,用root身分更新httpd伺服器

```yaml
- name: Schedule a command to execute in 20 minutes as root
  ansible.posix.at:
    command: yum update httpd -y
    count: 20
    units: minutes
```

或設定定期排程 使用cron模組
設定每兩周一次在半夜12點進行系統更新

```yaml
- name: Creates a cron file under /etc/cron.d
  ansible.builtin.cron:
    name: yum autoupdate
    weekday: "2"
    minute: "0"
    hour: "12"
    user: root
    job: "YUMINTERACTIVE=0 /usr/sbin/yum-autoupdate"

```

systemd模組可以使用systemd模組去控制systemd管理的服務他與service模組相同

管理儲存
mount掛載模組

```yaml
- name: Mount an NFS volume
  ansible.posix.mount:
    src: 192.168.1.100:/nfs/ssd/shared_data
    path: /mnt/shared_data
    opts: rw,sync,hard
    state: mounted
    fstype: nfs
```

使用redhat.rhel_system_roles.storage 管理LVM與檔案系統

透過storage role自動切割'設定'管理你的硬碟與邏輯卷宗
使用redhat.rhel_system_roles.storage 在複數的主機上管理未作分割區的硬碟
雖然storage role可以為未格式化的硬碟或lvm建立檔案系統,但是storage role無法建立硬碟分割表

針對sdb將他格式化為xfs,名稱為barefs

```markdown
- name: Manage storage
  hosts: hostb.lab.example.com
  roles:
    - name: redhat.rhel_system_roles.storage
  vars:
    storage_volumes:
      - name: barefs
        type: disk
        disks:
          - sdb
        fs_type: xfs
```

使用redhat.rhel_system_roles.storage 管理lvm

如果要建成lvm,那就改成storage_pools 的變數

```markdown
- name: Manage storage
  hosts: hostb.lab.example.com
  roles:
    - rhel-system-roles.storage
  vars:
    storage_pools:
      - name: myvg
        disks:
          - sda
          - sdb
          - sdc
        volumes:
          - name: mylv
            size: 2G
            fs_type: ext4
            mount_point: /mnt/dat
```

硬碟分割

基本上關於被管理主機的硬體設備,他的facts參數為 ansible_facts.devices供你查看
而基本建立硬碟分割表,最簡單直接的方式就是使用command模組

```markdown
- name: create partition from command module
  ansible.builtin.command: parted mklabel gpt /dev/vdb
```

如果想要做出硬碟分割的內容,可以使用community.general.parted
針對遠端主機身上的/dev/sdb作出分割區,格式為ext4

```yaml
- name: Create a new ext4 primary partition
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: present
    fs_type: ext4
```

那要如何確定遠端主機身上有無硬碟,可以透過facts取得

```yaml
- name: display sdb infomation
  ansible.builtin.debug:
    msg: {{ ansible_facts.devices.sdb }}

```

建立好後要確認掛在可以用ansible_facts.mount 查看掛載狀況

```yaml
- name: display sdb infomation
  ansible.builtin.debug:
    msg: {{ ansible_facts.mount }}
```

管理主機網路
透過redhat.rhel_system_roles.network role來管理跟設定
這個角色可以組態網卡'橋接器'bound 網卡(有作網卡HA的),VLAN 網卡,MACVLAN
這個角色他有兩種參數network_connections 或 network_provider
network_provider 則是宣告我這個角色管理網路的方式,例如第七版以後都是使用 nmcli 的指令進行管理
network_connections 是針對網卡上的設定進行調整,如ip多少,要不要自動啟動,網卡吃不吃dhcp....
例如以下,針對nodea include 角色近來,並組態該節點的名稱為enp1s0 的乙太網卡,網卡的ip為192.0.2.1/24,閘道是192.0.2.254,狀態是啟用,並且開機時會自動啟動

```markdown
---
- name: Configure the network
  hosts: nodea.lab.example.com
  tasks:
    - name: Configure an Ethernet connection with static IP
      ansible.builtin.include_role:
        name: rhel-system-roles.network
      vars:
        network_connections:
          - name: enp1s0
            interface_name: enp1s0
            type: ethernet
            autoconnect: yes
            ip:
              address:
                     - 192.0.2.1/24
                - 2001:db8:1::1/64
              gateway4: 192.0.2.254
              gateway6: 2001:db8:1::fffe
              dns:
                     - 192.0.2.200
                - 2001:db8:1::ffbb
              dns_search:
                - example.com
            state: up

```

使用dhcp的設定

```markdown
---
- name: Configure the network
  hosts: managed-node-01.example.com
  tasks:
    - name: Configure an Ethernet connection with dynamic IP
      ansible.builtin.include_role:
        name: rhel-system-roles.network
      vars:
        network_connections:
          - name: enp1s0
            interface_name: enp1s0
            type: ethernet
            autoconnect: yes
            ip:
              dhcp4: yes
              auto6: yes
            state: up
```

參考：
[https://docs.ansible.com/ansible/latest/reference_appendices/faq.html](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html) -How do I generate encrypted passwords for the user module
[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fail_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fail_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/known_hosts_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/known_hosts_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/posix/at_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/posix/at_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)[https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html)[https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html)[https://docs.ansible.com/ansible/latest/collections/community/general/parted_module.html](https://docs.ansible.com/ansible/latest/collections/community/general/parted_module.html)[https://access.redhat.com/articles/3050101](https://access.redhat.com/articles/3050101)[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/automating_system_administration_by_using_rhel_system_roles/assembly_configuring-network-settings-by-using-rhel-system-roles_automating-system-administration-by-using-rhel-system-roles](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/automating_system_administration_by_using_rhel_system_roles/assembly_configuring-network-settings-by-using-rhel-system-roles_automating-system-administration-by-using-rhel-system-roles)[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-local-storage-using-rhel-system-roles_configuring-and-managing-logical-volumes](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-local-storage-using-rhel-system-roles_configuring-and-managing-logical-volumes)[https://www.redhat.com/en/blog/automating-storage-management-rhel-system-roles](https://www.redhat.com/en/blog/automating-storage-management-rhel-system-roles)[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-lvm-logical-volumes_configuring-and-managing-logical-volumes#an-example-playbook-to-manage-logical-volumes_managing-lvm-logical-volumes-using-rhel-system-roles](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_logical_volumes/managing-lvm-logical-volumes_configuring-and-managing-logical-volumes#an-example-playbook-to-manage-logical-volumes_managing-lvm-logical-volumes-using-rhel-system-roles)[https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux/9/html/automating_system_administration_by_using_rhel_system_roles/assembly_configuring-network-settings-by-using-rhel-system-roles_automating-system-administration-by-using-rhel-system-roles](https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux/9/html/automating_system_administration_by_using_rhel_system_roles/assembly_configuring-network-settings-by-using-rhel-system-roles_automating-system-administration-by-using-rhel-system-roles)