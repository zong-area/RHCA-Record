# ch5

建立、安裝、編輯、移除檔案，並且管理主機與權限

關於檔案的模組:

| 模組名稱 | 說明 |
| --- | --- |
| blockinfile | 插入、更新、移除自定義區塊內多行的文字內容 |
| copy | 從本地檔案複製到遠端被管裡主機身上，他類似於file 模組，他也一樣可以設定權限跟selinux |
| fetch | 這個模組類似於copy 模組，但是是指相反的意思上。copy 模組是指把本地檔案複製到遠端被管裡主機，但fetch  是將遠端被管裡主機身上的檔案，拉到本地主機 |
| file | 設定參數與權限、檔案的rwx、selinux、軟連結、硬連結、跟目錄。此外這個模組還可以建立或刪除檔案、軟連結、硬連結、跟目錄 |
| lineinfile | 確認檔案內有沒有指定的行，並且使用正規表達式，決定為他新增或替換內容 |

使用檔案模組建立自動化範例
在這邊範例使用file 模組，先建立檔案後，在行修改檔案的Selinux

```yaml
[student@hosta ansible]$ cat file_create.yml
---
- name: file module use case
  hosts: hostb.lab.example.com
  become: true

  tasks:
    - name: create file in node 
      ansible.builtin.file:
        state: touch
        owner: root
        path: /home/exampl_file.txt
    
    - name: show file state
      ansible.builtin.shell: 'ls -alFZ /home/'
      register: show

    - name: display file state
      ansible.builtin.debug:
        var: show

    - name: change file selinux content
      ansible.builtin.file:
        path: /home/exampl_file.txt
        setype: samba_share_t

    - name: show file state
      ansible.builtin.shell: 'ls -alFZ /home/'
      register: show_change

    - name: display file state
      ansible.builtin.debug:
        var: show_change

```

輸出結果：

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout file_create.yml 

PLAY [file module use case] ****************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [create file in node] *****************************************************
changed: [hostb.lab.example.com]

TASK [show file state] *********************************************************
changed: [hostb.lab.example.com]

TASK [display file state] ******************************************************
ok: [hostb.lab.example.com] => {
    "show": {
        "changed": true,
        "cmd": "ls -alFZ /home/",
        "delta": "0:00:00.002153",
        "end": "2024-04-10 23:10:41.267430",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2024-04-10 23:10:41.265277",
        "stderr": "",
        "stderr_lines": [],
        "stdout_lines": [
            "total 4",
            "drwxr-xr-x.  5 root    root    system_u:object_r:home_root_t:s0           71 Apr 10 23:10 ./",
            "dr-xr-xr-x. 18 root    root    system_u:object_r:root_t:s0               236 Apr  5 03:44 ../",
            "drwx------.  6 devops  devops  unconfined_u:object_r:user_home_dir_t:s0  138 Apr  5 11:36 devops/",
            "-rw-r--r--.  1 root    root    unconfined_u:object_r:home_root_t:s0        0 Apr 10 23:10 exampl_file.txt",
            "drwx------. 15 kiosk   kiosk   unconfined_u:object_r:user_home_dir_t:s0 4096 Apr  5 11:29 kiosk/",
            "drwx------.  3 student student unconfined_u:object_r:user_home_dir_t:s0   78 Mar 26 11:19 student/"
        ]
    }
}

TASK [change file selinux content] *********************************************
changed: [hostb.lab.example.com]

TASK [show file state] *********************************************************
changed: [hostb.lab.example.com]

TASK [display file state] ******************************************************
ok: [hostb.lab.example.com] => {
    "show_change": {
        "changed": true,
        "cmd": "ls -alFZ /home/",
        "delta": "0:00:00.002058",
        "end": "2024-04-10 23:10:41.865645",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2024-04-10 23:10:41.863587",
        "stderr": "",
        "stderr_lines": [],
        "stdout_lines": [
            "total 4",
            "drwxr-xr-x.  5 root    root    system_u:object_r:home_root_t:s0           71 Apr 10 23:10 ./",
            "dr-xr-xr-x. 18 root    root    system_u:object_r:root_t:s0               236 Apr  5 03:44 ../",
            "drwx------.  6 devops  devops  unconfined_u:object_r:user_home_dir_t:s0  138 Apr  5 11:36 devops/",
            "-rw-r--r--.  1 root    root    unconfined_u:object_r:samba_share_t:s0      0 Apr 10 23:10 exampl_file.txt",
            "drwx------. 15 kiosk   kiosk   unconfined_u:object_r:user_home_dir_t:s0 4096 Apr  5 11:29 kiosk/",
            "drwx------.  3 student student unconfined_u:object_r:user_home_dir_t:s0   78 Mar 26 11:19 student/"
        ]
    }
}

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

在遠端被管裡主機上複製或編輯檔案
這兩者的應用上lineinfile 偏向為行的內容進行加入或替換，blockinfile 為一整個區塊、範圍的加入

```yaml
[student@hosta ansible]$ cat line_module.yml 
---
- name: line module example
  hosts: hostb.lab.example.com
  become: true

  tasks:
    - name: add content line in file 
      ansible.builtin.lineinfile:
        path: /home/exampl_file.txt
        line: 127.0.0.1 localhost

    - name: use blockinline
      ansible.builtin.blockinfile:
        path: /home/exampl_file.txt
        block: |
          iface eth0 inet static    
              address 192.0.2.23  
              netmask 255.255.255.0 
```

輸出結果：

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout line_module.yml 

PLAY [line module example] *****************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [add content line in file] ************************************************
changed: [hostb.lab.example.com]

TASK [use blockinline] *********************************************************
changed: [hostb.lab.example.com]

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  

```

確認

```yaml
[student@hosta ansible]$ ansible hostb.lab.example.com -i inventory -m command -a 'cat /home/exampl_file.txt'
hostb.lab.example.com | CHANGED | rc=0 >>
127.0.0.1 localhost
# BEGIN ANSIBLE MANAGED BLOCK
iface eth0 inet static    
    address 192.0.2.23  
    netmask 255.255.255.0 
# END ANSIBLE MANAGED BLOCK
```

另一個跟lineinfile 相關的的是ansible.builtin.replace 模組，也是透過正規表達式，對指定的行進行內容的更替

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout replace.yml 

PLAY [replace module example] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [replace content line in file] ********************************************
changed: [hostb.lab.example.com]

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

發現127.0.0.1 那一行被更替了

```yaml
[student@hosta ansible]$ ansible hostb.lab.example.com -i inventory -m command -a 'cat /home/exampl_file.txt'
hostb.lab.example.com | CHANGED | rc=0 >>
192.168.122.11 hostb
# BEGIN ANSIBLE MANAGED BLOCK
iface eth0 inet static    
    address 192.0.2.23  
    netmask 255.255.255.0 
# END ANSIBLE MANAGED BLOCK
```

刪除檔案

在state 的部分，從present 改為absent

```yaml
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

確認檔案
stat 模組可以用來確認檔案的狀態，如確認/path/to/something  的檔案，是否存在，如果不存在就顯示'islnk isn't defined /path/to/something' 的訊息

```yaml
- name: Get stats of the FS object
  ansible.builtin.stat:
    path: /path/to/something
  register: sym

- name: Print a debug message
  ansible.builtin.debug:
    msg: "islnk isn't defined (path doesn't exist)"
  when: sym.stat.islnk is not defined
```

透過Jinja2 來佈屬自定義模板
ansible 使用Jinja2 來當作他的系統模板，他是一種非常適合用來當作檔案模板的一種格式
他寫法如下，兩個{{  }}然後文字跟雙引號之間要有空白鍵

{{ content }}

註解的寫法
{# content #}

控制結構(即使用迴圈)的寫法
{% content %}

範例如下，透過用變數替代的方式，編輯/etc/hosts  的內容

```yaml
{# edit /etc/hosts line #}
{{ ansible_facts['default_ipv4']['address'] }} {{ ansible_facts['hostname'] }}
```

建立Jinja2 的模板
他可以吃資料、變數、或正規表達式的內容，而變數也可以直接帶入劇本中所使用的變數內容
那麼Jinja2 的檔案格式為.j2，透過template 模組，進行佈屬

```yaml
  tasks:
    - name: config j2 template
      ansible.builtin.template:
        src: for_j2.j2 
        dest: /root/all_host.txt

```

那麼我們要如何分辨設定的修改，究竟是Ansible 修改的亦或者是我們人用手自己修改的
這時只要觀察設定檔中有出現ANSIBLE MANAGED  的字樣，那就是知道這個模板、設定擋是ansible 修改的
那把這個字樣{{ ansible_managed }} ，包覆在jinja2 的檔案中用於宣告

使用迴圈
如以下範例，開頭寫法用for 結尾為endfor，他讀取users 變數裡面所有的內容將其顯示出來

```yaml
{% for user in users %}
      {{ user }}
{% endfor %}
```

或是透過群組，讀取inventory 擋裡面，群組名稱為myhosts 的主機 設定為myhost 變數，最後將myhost 的變數內容顯示出來

```yaml
{% for myhost in groups['myhosts'] %}
{{ myhost }}
{% endfor %}
```

例如說 j2 擋我需要我inventory 主機所有的主機名稱跟ip位置，透過模板把它放在/root/all_host.txt 這裡

```yaml
[student@hosta ansible]$ cat for_j2.j2 
{# ansible_manage #}

{% for host in groups.all %}
 {{ hostvars[host].ansible_facts.fqdn }} {{ hostvars[host].ansible_facts.default_ipv4.address }}
{% endfor %}
```

```yaml
[student@hosta ansible]$ cat j2_use_templat.yml 
---
- name: j2 use template
  hosts: all
  become: true

  tasks:
    - name: config j2 template
      ansible.builtin.template:
        src: for_j2.j2 
        dest: /root/all_host.txt
```

檢查

```yaml
[student@hosta ansible]$ ansible all -i inventory -m command -a "cat /root/all_host.txt"
hosta.lab.example.com | CHANGED | rc=0 >>

 servera.lab.example.com 192.168.122.10
 serverb.lab.example.com 192.168.122.11
 hostc.lab.example.com 192.168.122.12
 hostd.lab.example.com 192.168.122.13
 
hostb.lab.example.com | CHANGED | rc=0 >>

 servera.lab.example.com 192.168.122.10
 serverb.lab.example.com 192.168.122.11
 hostc.lab.example.com 192.168.122.12
 hostd.lab.example.com 192.168.122.13
```

輸出格式
為了將輸出的內容變的好看一些，你可能會需要修改輸出格式

```yaml
{{ output | to_json }}
{{ output | to_yaml }}
```

那為了要方便閱讀，你會需要將yaml 或json 轉換為人類直覺看的懂的樣子

```yaml
{{ output | to_nice_json }}
{{ output | to_nice_yaml }}
```

而to_nice_yaml 也可以被寫成to_json(indent=4, sort_keys=True)

```yaml
{{ output | to_json(indent=4, sort_keys=True) }}
```