# ch6

透過參數來選擇主機

例如以下，我們在inventory  中透過群組的方式，對主機進行分類
[hosta.lab.example.com](http://hosta.lab.example.com/) [與hostb.lab.example.com](http://xn--hostb-oo6n.lab.example.com/) 為dev 群組，所以在呼叫dev [時就會呼叫到hosta.lab.example.com](http://xn--hosta-6o6hk7he7al87bjxxy2b.lab.example.com/) [與hostb.lab.example.com](http://xn--hostb-oo6n.lab.example.com/) 這兩台主機

```yaml

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
```

那所以我在指定群組時，我們可以使用IP 位置、hostname、群組、特殊字元來進行過濾或篩選
例如: 

針對IP

```yaml
---
- name: file module remove file example
  hosts: 192.168.122.11
```

針對主機名稱

```yaml
---
- name: file module remove file example
  hosts: hostb.lab.example.com
```

針對群組

```yaml
---
- name: file module remove file example
  hosts: dev
```

特殊字元

除了hostc 以外,hostname [結尾為.lab.example.com](http://xn--bgtq09begj.lab.example.com/) 都涵蓋在內

```yaml
---
- name: ensure all node have manage
  hosts: '!hostc.lab.example.com,host*.lab.example.com'
```

只有hosta與hostc  

```yaml
---
- name: file module remove file example
  hosts: hosta, hostc
```

Include 與import 兩種導入劇本
目的在於，把多個工作分成多個劇本，你最後再由一個劇本，去一一呼叫這些零碎的劇本即可
使用import_tasks時，在執行playbook時會靜態導入工作，當使用include_tasks時，則是工作需要時再動態套用
如果工作中如果只用來處理單一的 task 檔，這時可以使用靜態導入的方式較好
"import_*"  會在解析劇本時，預先一併處理導入的劇本(即進行乾的跑的時候，會把import 的yml 帶入執行)
"include_*" 只會在執行劇本時，才會處理匯入進來的劇本(即進行乾的跑的時候，不會把include的yml 帶入執行)

import 寫法如下：

```yaml
import_playbook:
import_tasks:
```

include 寫法如下：

```yaml
include_tasks:
```

兩者的寫法的差異如下

```yaml
tasks:
- import_tasks: common_tasks.yml
# or
- include_tasks: common_tasks.yml

```

在handlers底下使用的方式

```yaml
handlers:
- include_tasks: more_handlers.yml
# or
- import_tasks: more_handlers.yml
```

import_playbook 範例: 

```yaml
- name: Include a play after another play
  ansible.builtin.import_playbook: otherplays.yaml
```

include_tasks 範例:

```yaml
- hosts: all
  tasks:
    - ansible.builtin.debug:
        msg: task1

    - name: Include task list in play
      ansible.builtin.include_tasks:
        file: stuff.yaml
```

或是將變數寫在導入的劇本下方

```yaml
- name: Set variables on an imported playbook
  ansible.builtin.import_playbook: otherplays.yml
  vars:
    service: httpd
```

```yaml
#otherplays.yml
  tasks:
    - name: importtasks
      ansibe.builtin.dnf:
        name: "{{ service }}"
        state: present 
    - name: start {{ service }}
	  ansible.builtin.service:
	    name: "{{ service }}"
		state: started	

```

其中使用import 注意的是

1. loop 不能被import 使用
2. import 呼叫的變數，不可以使用inventory 檔、host_vars、group_vars 目錄裡面的變數
3. 不可以使用handlers 去呼叫import時的tasks(如下錯誤範例)，因為用import 時，會執行的是import 進來的工作跟覆蓋原本劇本的handlers
如

```yaml
    - name: Include task list in play
      ansible.builtin.include_tasks:
        file: stuff.yaml
	    notify: stuff
handlers:
  - name: stuff
    ansible.builtin.service
 ........	

```

使用include 注意的是

1. include 進來的工作
2. include 進來的工作不會出現在執行大劇本時的輸出中
3. 你無法使用notify 去呼叫include 進來的劇本裡面的 handler
4. 你無法使用--start-at-task (指定從哪一個工作開始跑)，去指定先執行include 進來的哪一個工作

參考:
[https://docs.ansible.com/ansible/2.9/user_guide/playbooks_reuse.html#playbooks-reuse](https://docs.ansible.com/ansible/2.9/user_guide/playbooks_reuse.html#playbooks-reuse)[https://docs.ansible.com/ansible/2.9/user_guide/playbooks_reuse_includes.html](https://docs.ansible.com/ansible/2.9/user_guide/playbooks_reuse_includes.html)