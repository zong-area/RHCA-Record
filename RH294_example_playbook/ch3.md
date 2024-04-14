# ch3

變數的使用盡量使用（﹍）底線來標示，不要使用空白鍵、（.）點、或$ 錢號

變數的種類有
群組變數、主機變數、play 的變數、facts
而群組變數跟主機變數都可以設定在inventory中
或者將群組變數，設定在當前project 底下的group_vars 目錄中
主機變數也可以設定在host_vars 目錄底下

**NOTE
且記group_vars 與host_vars 變數的目錄是規定好的，不是你可以亂取名稱

play 的變數，直接在上方定義變數名稱，或將變數寫入檔案中，並在劇本的開頭引入

```markdown
- hosts: all
  vars:
    user: joe
    home: /home/joe
```

透過變數檔引入

```markdown
[student@hosta ansible]$ cat users.yml
users: joe
home: /home/joe	
```

將群組變數定義在group_vars/ 之中
未來在劇本中呼叫package時，dev 群組的主機會安裝httpd，test 群組的主機會安裝nginx

```markdown
[student@hosta ansible]$ cat group_vars/dev 
package: httpd
[student@hosta ansible]$ cat group_vars/test 
package: nginx
```

主機變數 建立在inventry  之中

```markdown
[student@hosta ansible]$ cat inventory 

[dev]
hosta.lab.example.com
hostb.lab.example.com

[test]
hostc.lab.example.com
hostd.lab.example.com

[manage:children]
dev
test

[manage:vars]
user=joe
```

主機變數目標即針對該主機，可以定義在inventor 以下爲定義在host_vars 裡

```markdown
[student@hosta ansible]$ cat host_vars/hosta.lab.example.com 
user: joe
[student@hosta ansible]$ cat host_vars/hostb.lab.example.com 
user: jerry
[student@hosta ansible]$ cat host_vars/hostc.lab.example.com 
user: marry
[student@hosta ansible]$ cat host_vars/hostd.lab.example.com 
user: hanson
```

另外，在指令上直接指定變數爲最高優先度，將蓋過其他所有地方的組態，此為globol 環境，擁有最高的優先度

如：ansible-navigator run install.yml  -e "package=mariadb-server"，此時劇本中所有的package 變數，將代入mariadb-server 套件，不會事原先預設的內容

```markdown
[student@hosta ansible]$ ansible-navigator run install.yml  -e "package=mariadb-server"
```

字典檔變數的使用，類似過去矩陣的寫法

```markdown
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```

如果我要呼叫bjones 的話，我要的寫法應該爲用. 或[''] 來表示階層

```markdown
users.bjones.home_dir 
或
users['bjones']['first_name'] 
```

註冊變數

這個非常重要，他會把你的執行結果儲存起來，之後在搭配debug 模組，將其執行結果打出來給你看到
寫法如下：
寫法是register： 他與模組在同一階層， 而變數名稱隨便寫，這裏示範用result
而debug 模組使用var 參數後面接變數名稱，這裏變數名稱要與register 那裏寫的一樣

```markdown
[student@hosta ansible]$ cat install.yml 
---
- name: install package for all
  hosts: hostb.lab.example.com
  become: true

  tasks:
    - name: install virt package
      ansible.builtin.dnf:
        name: httpd
        state: present
      register: result

    - name: display install messages
      ansible.builtin.debug:
        var: result
```

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout install.yml -C

PLAY [install package for all] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [install virt package] ****************************************************
changed: [hostb.lab.example.com]

TASK [display install messages] ************************************************
ok: [hostb.lab.example.com] => {
    "result": {
        "changed": true,
        "failed": false,
        "msg": "Check mode: No changes made, but would have if not in check mode",
        "rc": 0,
        "results": [
            "Installed: apr-util-bdb-1.6.1-9.el8.x86_64",
            "Installed: mod_http2-1.15.7-9.module_el8+920+e7b4a22a.3.x86_64",
            "Installed: centos-logos-httpd-85.8-2.el8.noarch",
            "Installed: httpd-2.4.37-64.module_el8+965+1ad5c49d.x86_64",
            "Installed: apr-util-openssl-1.6.1-9.el8.x86_64",
            "Installed: httpd-tools-2.4.37-64.module_el8+965+1ad5c49d.x86_64",
            "Installed: apr-1.6.3-12.el8.x86_64",
            "Installed: apr-util-1.6.1-9.el8.x86_64",
            "Installed: httpd-filesystem-2.4.37-64.module_el8+965+1ad5c49d.noarch"
        ]
    }
}

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

使用ansible-vault 隱藏敏感資訊 

主要用途是希望我的變數用加密/亂數的方式加以隱藏某些我不希望其他人直接看到的內容
如：webkeys、password..與其他類型的敏感資料不應該以純文字的方式儲存在playbook中，ansible Valut 對資料進行加密與解密使資料不可讀，只有在讀取資料時，輸入正確的密碼才可以閱讀到正確的資訊
ansible-vault 基本上分為幾種 建立(create)、編輯(edit)、加密(encrypt)、解密(decrypt)、瀏覽(view) 這幾種
還有一種爲rekey 用來重置/重新設定密碼檔的密碼

```markdown
[student@hosta plays]$ ansible-vault --help
usage: ansible-vault [-h] [--version] [-v] {create,decrypt,edit,view,encrypt,encrypt_string,rekey} ...

encryption/decryption utility for Ansible data files

positional arguments:
  {create,decrypt,edit,view,encrypt,encrypt_string,rekey}
    create              Create new vault encrypted file
    decrypt             Decrypt vault encrypted file
    edit                Edit vault encrypted file
    view                View vault encrypted file
    encrypt             Encrypt YAML file
    encrypt_string      Encrypt a string
    rekey               Re-key a vault encrypted file
```

首先使用ansible-vault create 建立一個加密的檔案

```markdown
[student@hosta ansible]$ ansible-vault create secret.yml
New Vault password:  12345
Confirm New Vault password: 12345
```

加密完畢後，直接用cat 是無效，無法閱讀的

```markdown
[student@hosta ansible]$ cat secret.yml 
$ANSIBLE_VAULT;1.1;AES256
66323266356239376136366562626365623566626435653961343861653132353761383263373234
6636613761323564323237333961303736623263356664370a653636343364383534393634633033
64323133386432343534303562393537373665646535616436666530376461313434666138633035
3966656437376536660a636637383537303066653861393761386566396336363663336463383930
30393930396238643361653761653234336130353437663939663662313531323334623431393938
6461623130666336356334396262383433336261616631323062
```

必須使用ansible-vault view secret.yml 並且輸入正確的密碼才可以閱讀

```markdown
[student@hosta ansible]$ ansible-vault view secret.yml
Vault password: 
username: ansible-secret
password: 56789
```

或是將密碼存成一個檔案(vault-pass)，之後藉由指令呼叫即可，減少程式執行中斷的機會

而此檔案也要記得權限爲0600

```markdown
[student@hosta ansible]$ ansible-vault create --vault-password-file=vault-pass secret.yml
```

重設密碼

```markdown
[student@hosta ansible]$ ansible-vault rekey secret.yml
Vault password: 
New Vault password: new_password
Confirm New Vault password: new_password
Rekey successful
```

管理ansible facts
ansible facts 他的指的意思是，被管理主機身上硬體或軟體的狀態，如ip 位置、hostname、FQDN、cpu、記憶體、可用的記憶體、可用的硬碟空間、掛載多少硬碟.....等等
那也可以故意去自定義facts 到被管理主機上，如此一來，再用撈取facts 的時候，就可以直接撈到這些自定義的資訊
要讀取ansible facts 需要使用到的模組爲ansible_builtin.setup ，透過這個模組，可以抓出被管理主機身上預設的facts

```markdown
[student@hosta ansible]$ cat display_facts.yml 
---
- name: display manage node facts
  hosts: all
  become: true

  tasks:
    - name: node facts from use debug module 
      ansible.builtin.debug:
        var: ansible_facts

```

```markdown

[student@hosta ansible]$ ansible-navigator run -m stdout display_facts.yml 

PLAY [display manage node facts] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]
ok: [hosta.lab.example.com]
ok: [hostc.lab.example.com]
ok: [hostd.lab.example.com]
TASK [node facts from use debug module] ****************************************
ok: [hostc.lab.example.com] => {
    "ansible_facts": {
        "all_ipv4_addresses": [
            "172.24.0.12",
            "192.168.122.12",
            "172.25.250.12"
        ],
        "all_ipv6_addresses": [
            "fe80::23fb:6428:4ee2:e017",
            "fe80::ad49:564c:a789:977d",
            "fe80::5054:ff:fea1:2318"
        ],
        "ansible_local": {},
        "apparmor": {
            "status": "disabled"
        },
        "architecture": "x86_64",
        "bios_date": "04/01/2014",
        "bios_vendor": "SeaBIOS",
        "bios_version": "1.16.0-1.module_el8.7.0+1140+ff0772f9",
        "board_asset_tag": "NA",
        "board_name": "RHEL-AV",
        "board_serial": "NA",
        "board_vendor": "Red Hat",
        "board_version": "RHEL-8.6.0 PC (Q35 + ICH9, 2009)",

.......
        "memtotal_mb": 15720,
        "module_setup": true,
        "mounts": [
            {
                "block_available": 6165630,
                "block_size": 4096,
                "block_total": 7584135,
                "block_used": 1418505,
                "device": "/dev/mapper/cs-root",
                "fstype": "xfs",
                "inode_available": 15050410,
                "inode_total": 15175680,
                "inode_used": 125270,
                "mount": "/",
                "options": "rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota",
                "size_available": 25254420480,
                "size_total": 31064616960,
                "uuid": "07b23dd1-e038-4324-a254-3f407f909c46"
            },
.......
        "system": "Linux",
        "system_capabilities": [],
        "system_capabilities_enforced": "False",
        "system_vendor": "Red Hat",
        "uptime_seconds": 613,
        "user_dir": "/root",
        "user_gecos": "root",
        "user_gid": 0,
        "user_id": "root",
        "user_shell": "/bin/bash",
        "user_uid": 0,
        "userspace_architecture": "x86_64",
        "userspace_bits": "64",
        "virtualization_role": "guest",
        "virtualization_tech_guest": [
            "kvm"
        ],
        "virtualization_tech_host": [],
        "virtualization_type": "kvm"
    }
}

```

常用facts

| 意思 | 寫法 |
| --- | --- |
| 短的主機名稱 | ansible_facts['hostname'] |
| 完整主機名稱 | ansible_facts['fqdn'] |
| IP 位置 | ansible_facts['default_ipv4']['address'] |
| 列出使用的網卡名稱 | ansible_facts['interfaces'] |
| 主機的vda 硬碟大小 | ansible_facts['devices']['vda']['partitions']['vda1']['size'] |
| 主機的DNS Server	 | ansible_facts['dns']['nameservers'] |
| 主機目前正在執行的kernel | ansible_facts['kernel'] |

當然你也可以用. 來呈現

| [’’] 標示方法 | . 標示方法 |
| --- | --- |
| ansible_facts['hostname']  | ansible_facts.hostname |
| ansible_facts['fqdn'] | ansible_facts.fqdn |
| ansible_facts['default_ipv4']['address'] | ansible_facts.default_ipv4.address |
| ansible_facts['interfaces']					 | ansible_facts.interfaces |
| ansible_facts['dns']['nameservers']			 | ansible_facts.dns.nameservers |
| ansible_facts['kernel'] | ansible_facts.kernel |

呈現結果如下

```yaml
---
- hosts: all
  tasks:
    - name: Prints various Ansible facts
      ansible.builtin.debug:
      msg: >
        The default IPv4 address of {{ ansible_facts.fqdn }}
        is {{ ansible_facts.default_ipv4.address }}
```

執行結果:

```markdown
TASK [Prints various Ansible facts] ****************************************
ok: [hosta.lab.example.com] => {
"msg": "The default IPv4 address of hosta.lab.example.com is 172.25.250.10\n"
}

```

如果你不希望劇本執行時，去自動撈取facts，要在劇本的開頭gather_facts: no
如:

```markdown
---
- name: This play does not automatically gather any facts
  hosts: all
  gather_facts: no
```

建立自定義facts
可以自定義facts 再被管理的節點上，而你可以用此方式，取得被管理主機上的facts 變數
那麼你可以使用ansible.builtin.setup ，取得取得被管理主機上的自己的facts 內容
建立fatcs 建立的位置再/etc/ansible/facts.d  目錄底下，副擋名為.fact
如:

```markdown
[packages]
web_package = httpd
db_package = mariadb-server
[users]
user1 = joe
user2 = jane
```

```yaml
- name: Prints various Ansible facts 
  debug:
    msg: >
        The package to install on {{ ansible_facts['fqdn'] }}
        is {{ ansible_facts['ansible_local']['custom']['packages']['web_package'] }}
     
```

結果:

```markdown
TASK [Prints various Ansible facts] ****************************************
ok: [hosta.lab.example.com] => {
"msg": "The package to install on hostb.example.com is httpd"
}

```

魔法變數 magic variable
魔法變數它是一的自動更動的變數，而使用魔法變數可以被用來取得被管理主機上一些資訊
那魔法變數有
hostvars
這裡面的變數內容是放被管裡的主機名稱，
group_names
指的inventory 裡面的群組名稱
groups
指的是inventory 裡面的群組或主機
inventory_hostname
指的inventory 裡面的主機名稱，他並不是被管裡主機身上的facts 抓回來的主機名稱

使用方法可以項是這樣，可以透過hostvars 來指定是哪一個主機身上的facts 的資訊

```markdown
- name: Print list of network interfaces for demo2
  ansible.builtin.debug:
    var: hostvars['demo2.example.com']['ansible_facts']['interfaces']
```