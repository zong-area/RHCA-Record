# ch2

ansible 實際操作
ansible 主要操作的需要爲ansible.cfg 設定檔與inventory 庫存檔兩者

1,inventory
用於定義群組，而群組底下則套用哪些主機，如dev 群組底下含有hosta、hostb 兩臺主機
當我呼叫dev 群組時，操作與改動的行為皆只會針對hosta與hostb

```markdown
[root@control ansible]# vim inventory
[dev]
hosta.lab.example.com
hostb.lab.example.com

[test]
hostc.lab.example.com
hostd.lab.example.com
```

ansible.cfg 設定檔，設定檔預設最先讀取的設定檔爲當前執行目錄的ansible.cfg
ansible 每個專案的目錄都是獨立的環境，包含所有運行/作業需所需要的具體內容。paybook 可以在該項目的目錄裏找到
主要提供於當前project 的環境設定，基本設定如下
remote_user 爲ssh到被管理主機上時，進去的使用者身份
ask_pass  詢問到對方主機上時，要不要輸入密碼，設爲false
privilege_escalation 欄位，標示到被管理主機上時，提權的參數

```markdown
[student@control web]$ vim ansible.cfg
[defaults]
inventory      = ./inventory  
remote_user   =  NOPASS_USER
ask_pass = false

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

```

可以使用ansible --version 進行檢查，當前目錄是吃到當前設定檔中

```markdown
[student@control web]$ ansible --version 
ansible [core 2.13.3]
  config file = /home/student/web/ansible.cfg **********************************
  configured module search path = ['/home/student/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /home/student/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
```

執行第一個playbook

撰寫第一個劇本，將透過使用dnf 這個模組，來為指定的群組/主機，安裝套件
開頭起始爲三個---，第二行 - name: 描寫這個劇本是用來做什麼？ ，第三行hosts 標示我這個劇本的目標要對那些被管理的主機，使用all 爲代表紀載在inventory 裡的所有主機，都是我的執行對象
become:  決定要不要提權的意思
接著對齊往下空一行，撰寫第一個任務(task)，此處撰寫的內容，爲實際上要用到哪些模組根作哪些行爲
模組的名稱爲ansible.builtin.MODULE_NAME，第三段爲模組名稱。而此處我所使用的是dnf 模組，因為要安裝軟體
而第一個縮排，name 顯示我要安裝的軟體名稱爲libvirt 虛擬化套件
第二個參數state present 的意思爲從無到有，如果我判定這個節點上，沒有這個軟體，那我就安裝

- * NOTE
此處如果是習慣舊版本的方式，也可以只單純輸入dnf 即可不需要前面的ansible.builtin 的字樣，但有些時候，可能因為寫不清楚，或是ansible 無法很正確的判斷模組名稱，因此建議寫完整字樣

```markdown
[student@hosta ansible]$ cat install.yml 
---
- name: install package for all
  hosts: all
  become: true

  tasks:
    - name: install virt package
      ansible.builtin.dnf:
        name: libvirt
        state: present
```

執行時的指令，將改為使用ansible-navigator run ，-m stdout 的意思是將執行結果的標準輸出顯示出來，最後加上劇本
其中看到hosta、hostb 本體爲ok 但是hostc、hostd 卻是爲change
這是因為hosta、hostb 已經安裝了這個libvirt 的套件，因此不改變他的狀態
但是hostc、hostd 本身沒有安裝過libvirt 所以在這兩個主機上，安裝libvirt 的套件，因為有做更動，所以他的狀態是change

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout install.yml 

PLAY [install package for all] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [hosta.lab.example.com]
ok: [hostb.lab.example.com]
ok: [hostc.lab.example.com]
 ok: [hostd.lab.example.com]

TASK [install virt package] ****************************************************
ok: [hosta.lab.example.com]
ok: [hostb.lab.example.com]
changed: [hostd.lab.example.com] 
changed: [hostc.lab.example.com]

PLAY RECAP *********************************************************************
hosta.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostb.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostc.lab.example.com      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostd.lab.example.com      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
 
```

輸出內容
當然其中你也可以使用-v 來增加劇本的輸出結果，可用來進行故障排除之用
常見爲-v、-vv、-vvv、-vvvv 依照v 的數量決定他出來的訊息，當然也可能因為輸出結果過多而導致你無法判斷，因此建議適量-v 即可

拼字檢查
可用於檢查，你的劇本有沒有拼錯字，如下
但無法檢測你的劇本有沒有錯，或能不跑，因此個人習慣建議使用-C 進行所謂的乾的跑 --dry-run

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout install.yml --syntax-check
playbook: /home/student/ansible/install.yml
[student@hosta ansible]$
```

乾的跑 尾巴加上-C

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout install.yml -C

PLAY [install package for all] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]
ok: [hosta.lab.example.com]
ok: [hostd.lab.example.com]
ok: [hostc.lab.example.com]

TASK [install virt package] ****************************************************
ok: [hostd.lab.example.com]
ok: [hostc.lab.example.com]
ok: [hosta.lab.example.com]
ok: [hostb.lab.example.com]

PLAY RECAP *********************************************************************
hosta.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostb.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostc.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hostd.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Multiple Plays
多齣戲在同一劇本
在同一個劇本裡面，可以有多齣戲(play)。意思就是你可以在一個劇本裡面寫兩齣戲，如可以第一個戲的內容/行爲針對dev 群組，安裝網頁伺服器並啟動網頁伺服器。第二齣戲則針對test 群組安裝開發人員工具

```yaml
---
- name: multi play running
  hosts: dev
  become: true

  tasks:
    - name: dev group install apache
      ansible.builti.dnf:
        name: httpd
        state: present

    - name: httpd service start
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
        
- name: two play
  hosts: test
  become: true

  tasks:
    - name: install Developer tools
      ansible.builtin.dnf:
        name: '@Development tools'
        state: present

```

通用/常用模組
File 類型相關：

| 模組 | 功用 |
| --- | --- |
| ansible.builtin.copy  | 用於複製到管理主機 |
| ansible.builtin.File | 設定權限與建立檔案、目錄、軟連結之用 |
| ansible.builtin.lineinfile  | 確認檔案中是否有標示的關鍵字或 |
| ansible.builtin.synchronize  | 用於檔案同步之用 |

軟體類別：

| 模組 | 功用 |
| --- | --- |
| ansible.builtin.package | 管理軟體或是自動安裝、移除在管理的主機上，且會自動偵測該平臺的版本，選擇適用於該平臺的軟體管理工具進行管理 |
| ansible.builtin.dnf | 使用dnf 進行管理 |
| ansible.builtin.apt | 使用apt 進行管理(即.deb) |
| ansible.builtin.pip | 使用pip 進行管理 |

系統類別

| 模組 | 功用 |
| --- | --- |
| ansible.posix.firewalld | 用於防火牆開放哪些服務哪些port |
| ansible.builtin.reboot | 重啟主機 |
| ansible.builtin.service | 管理systemd 的服務 |
| ansible.builtin.user | 建立或刪除使用者 |

模組說明
若想要知道當前系統有哪些模組或模組的使用說明（可視為模組的man page） 使用ansible-navigator doc -l 可以列出

```markdown
[student@hosta ~]$ ansible-navigator doc -l 
ansible.builtin.add_host                                Add a host (and alternatively a group) to the ansible-...
ansible.builtin.apt                                     Manages apt-packages                                  
ansible.builtin.apt_key                                 Add or remove an apt key                              
ansible.builtin.apt_repository                          Add and remove APT repositories                       
ansible.builtin.assemble                                Assemble configuration files from fragments           
ansible.builtin.assert                                  Asserts given expressions are true                    
ansible.builtin.async_status                            Obtain status of asynchronous task                    
ansible.builtin.blockinfile                             Insert/update/remove a text block surrounded by marker...
ansible.builtin.command                                 Execute commands on targets                           
ansible.builtin.copy                                    Copy files to remote locations                        
ansible.builtin.cron                                    Manage cron.d and crontab entries                     
ansible.builtin.deb822_repository                       Add and remove deb822 formatted repositories          
ansible.builtin.debconf                                 Configure a .deb package                              
ansible.builtin.debug                                   Print statements during execution                     
ansible.builtin.dnf                                     Manages packages with the `dnf' package manager     
```

對於想看的模組，只需要將模組名稱加再ansible-navigator doc 的後方，即會進入說明界面

```markdown
Name: dnf (module)                                                                                          
  0?---                                                                                                    ?
  1?doc:                                                                                                    
  2?  attributes:                                                                                           
  3?    action:                                                                                             
  4?      description: Indicates this has a corresponding action plugin so some parts                       
  5?        of the options can be executed on the controller                                                
  6?      details: In the case of dnf, it has 2 action plugins that use it under the hood,                  
  7?        M(ansible.builtin.yum) and M(ansible.builtin.package).                                          
  8?      support: partial                                                                                  
  9?    async:                                                                                              
 10?      description: Supports being used with the C(async) keyword                                        
 11?      support: none                                                                                     
 12?    bypass_host_loop:                                                                                   
 13?      description:                                                                                      
 14?      - Forces a 'global' task that does not execute per host, this bypasses per host                   
 15?        templating and serial, throttle and other loop considerations                                   
 16?      - Conditionals will work as if C(run_once) is being used, variables used will                     
 17?        be from the first available host                                                                
 18?      - This action will not work normally outside of lockstep strategies
```