# ch4

透過定義迴圈，可以減化同一個模組要作的事情
例如我想要啟用郵件伺服器與郵件加密的軟體，這時一般直接service 模組寫法寫兩次

```markdown
- name: Postfix is running
  service:
    name: postfix
    state: started
- name: Dovecot is running
  service:
    name: dovecot
    state: started
```

但這樣效率實在過於低下，透過變數替代的方式，再service 模組的那一階層，加上loop: 的字樣，底下標示要帶入的變數
上方名稱的變數寫法，loop 的變數引用 使用的名稱為item

```markdown
- name: Postfix and Dovecot are running
  service:
    name: "{{ item }}"
    state: started
  loop:
    - postfix
    - dovecot
```

另外一種變數替代的方法，則是寫在劇本頂部

```markdown
vars:
  mail_service:
    - posifix
	- dovecot
  - name: Postfix and Dovecot are running
    service:
      name: "{{ item }}"
      state: started  
    loop: "{{ mail_services }}"
```

階層式的迴圈
可以將變數寫在模組底下設定 區域變數

```markdown
- name: create users and set password
  ansible.builtin.user:
    name: "{{ item.name }}"
    password: "{{ item.password }}"
    state: present
  loop:
     - name: jerry
       password: redhat
     - name: marry
       password: redhat
```

透過loop 使用註冊變數
透過註冊變數 將他的輸出結果村如註冊變數中 方便你去觀察或記錄
所以在這個地方會將將shell模組引用loop 中的變數 並將結果存入註冊變數echo_results中 在透過debug模組將內容顯示出來

```markdown
[student@workstation loopdemo]$ cat loop_register.yml
---
- name: Loop Register Test
  gather_facts: no
  hosts: localhost

  tasks:
    - name: Looping Echo Task
      ansible.builtin.shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results

    - name: Show echo_results variable
      ansible.builtin.debug:
        var: echo_results        

```

執行結果後《會出現兩個輸出（因為變數帶兩次）  This is my item: one與  This is my item: two

```markdown
.........
"stderr": "",
"stderr_lines": [],
"stdout": "This is my item: one",
"stdout_lines":
"This is my item: one"      
]
與
.......
"stderr": "",
"stderr_lines": [],
"stdout": "This is my item: two",
"stdout_lines": [
"This is my item: two"
```

Condition
判斷式的應用 ansible 變數'註冊變數'ansible facts可以被用來幫你作出區分或決定那些管理的主機要做哪些事情
這時我們使用的是when 他一樣與loop相同階層是在模組那一層及
例如如果這台主機的硬碟空間不足5G我就不安裝應用程式

```markdown
tasks:
  - name: install package
    ansible.builtin.dnf:
      name: httpd
      state: present
     when: ansible_facts.disk. >= "500"
```

when 可以用的規則表達

| 意思 | 表示方式 |
| --- | --- |
| 等號 | max_memory == 512 |
| 小於 | min_memory < 128 |
| 大於 | min_memory > 128 |
| 小於等於 | min_memory <= 256 |
| 大於等於 | min_memory >= 512 |
| 不等於 | min_memory != 512 |
| 變數存在 | min_memory is defined |
| 變數不存在 | min_memory is not defined |
| 可以使用 | memory_available |
| 不可以使用 | not memory_available |
| 變數內容是否等於變數內容 | ansible_facts['distribution'] in supported_distros |

範例
定義supported_distros 的變數為RedHat 或Fedora ,並且當ansible_distribution是的內容符合supported_distros的內容時,就安裝網頁伺服器
或
當被管理主機中有主機是RedHat 或Fedora 時,就安裝網頁伺服器

```yaml
---
- name: Demonstrate the "in" keyword
  hosts: all
  gather_facts: yes
  vars:
     supported_distros:
	     - RedHat
	     - Fedora
  tasks:
    - name: Install httpd using yum, where supported
      ansible.builtin.dnf:
        name: http
	      state: present
      when: ansible_facts.distribution in supported_distros
```

學習使用多重判斷式
當主機系統是RedHat 或Fedora 時

```yaml
when: ansible_facts['distribution'] == "RedHat" or ansible_facts['distribution']
== "Fedora"
```

當主機版本號是9.0 並且kernel版本為5.14.0-70.13.1.el9_0.x86_64

```yaml
when: ansible_facts['distribution_version'] == "9.0" and ansible_facts['kernel']
== "5.14.0-70.13.1.el9_0.x86_64"
```

或是將or 與and合併在一起

```yaml
when: >
    ( ansible_distribution == "RedHat" and
      ansible_distribution_major_version == "8" )
    or
    ( ansible_distribution == "Fedora" and
    ansible_distribution_major_version == "30" )
```

使用loop迴圈搭配when條件句
透過loop制定變數內容搭配when的規則去作出判斷,而有使用loop他的變數就一定用item來替代
例如底下我使用被管理主機的facts 的內容,去判斷他的掛載的空間大小是否還大於300MB,如果足夠就安裝mariadb-server

```yaml
- name: install mariadb-server if enough space on root
  ansible.builtin.dnf:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
```

那當然我們其實也可以使用註冊變數來當作判斷式,最常用的就是result.rc
我們都知道程式和指令執行成功的話,就代表0 非0就是不成功
所以我們可以使用result.rc == 0 來幫助我們判斷上一個指令的結果,是否成功的依據

```yaml
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      ansible.builtin.command: /usr/bin/systemctl start httpd
      register: result
      
    - name: Restart Apache HTTPD based on Postfix status
      ansible.builtin.dnf:
        name: httpd
        state: present
      when: result.rc == 0
```

Handlers
handlers主要是用來減少我們去執行重複/多餘的行為,他主要是將整個劇本最後要做的事情,在所有tasks執行完後,再執行的tasks
例如說我們可能因為修改了網頁伺服器的設定檔,那加入要使設定檔生效,我會需要重開重讀伺服器
那我不可能,每改一個設定我就重開伺服器,這會使得劇本變得複雜且多於
所以我使用handler,改完整個伺服器設定後,再重開伺服器即可

那handlers使用的單字為notifly,階層根模組在同一層
然後注意notifly 那裡寫的變數（restart apache）與handlers 的名稱要相同,不可以不同
然後無論在這個劇本中 handler 被呼叫幾次，它在最後只會跑一次

```yaml
  tasks:
  - name: copy demo.example.conf configuration template
    ansible.builtin.template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart apache

handlers:
  - name: restart apache    
    service:
      name: httpd
      state: restarted
```

略過失敗
在執行劇本的時候,如果劇本中間有任何的錯誤,會導致劇本中斷,那我們希望即使這個tasks失敗,或是這個tasks有可能會失敗,我們也要繼續執行下去,此時我們會使用ignore_errors: yes 參數
ignore_errors: yes 用來宣告即使發生失敗,也還是繼續跑下去
那我們也可以透過error的組合,進行管理
定義在tasks之上就表示我整個劇本執行出了錯誤,也繼續跑到底,放在模組名稱底下,就只代表即使這個模組任務失敗,也還是繼續執行下一個tasks

```yaml
   - name: Latest version of notapkg is installed
     ansible.builtin.dnf:
       name: libvirt
       state: latest
     ignore_errors: yes

```

強制執行handlers
我們明白劇本中間有任何的錯誤,會導致劇本中斷,那可能導致我的伺服器,不完整的開啟或設定
因此宣告force_handlers: yes,表示這個劇本中間即使發生任何error ,我都要執行handlers

```yaml
---
- hosts: all
  force_handlers: yes
  tasks:
```

定義失敗的條件
在有些時候,如果我們執行了一個命令稿,但我們不知道他執行是否成功,除了可以用when 加入錯誤是的特徵進行判斷外
此時我們會使用fail模組. fail模組可以幫助我們定義何為失敗
這邊特別談到fail 模組目的是，判定上一個工作失敗，不要跑

```yaml

- name: Example using fail and when together
  ansible.builtin.fail:
    msg: The system may not be provisioned according to the CMDB status.
  when: cmdb_status != "to-be-staged"

```

Block rescue always
block可以用來控制底下那一個區塊的工作,並搭配when 條件句組合使用組合使用

```yaml

 tasks:
   - name: Install, configure, and start Apache
     block:
       - name: Install httpd and memcached
         ansible.builtin.yum:
           name:
           - httpd
           - memcached
           state: present

       - name: Apply the foo config template
         ansible.builtin.template:
           src: templates/src.j2
           dest: /etc/foo.conf

       - name: Start service bar and enable it
         ansible.builtin.service:
           name: bar
           state: started
           enabled: True
     when: ansible_facts['distribution'] == 'CentOS'
     become: true
     become_user: root
     ignore_errors: true
```

其他常使用搭配rescue always來進行組合
這三者的意思是如果我第一個Block區塊的工作失敗了,那我就執行rescue 在跑always
如果我第一個block區塊工作成功 那我就跑always區塊的工作

```yaml
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute, due to the above task failing, :-('
     rescue:
       - name: Print when errors
         ansible.builtin.debug:
           msg: 'I caught an error'

       - name: Force a failure in middle of recovery! >:-)
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I also never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes"
```