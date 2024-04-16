# ch7

role 與ansible collection
在我們佈署、轉寫劇本到後來，發現我們會一直撰寫相同的程式碼、工作，當然你可以透過調整執行的順序來減緩
但是長期下來將會使你的劇本變的籠長、可讀性下降，將使你的劇本變得不好閱讀維持
而Roles 可以透過已知結構去自動呼叫、載入變數、檔案、工作、handler 或其他ansible 的元件
將要執行的工作分擔給每個role以後，為來就可以很簡單的去執行這些重複的工作

Roles 的基本架構如下
common 為主要角色名稱
handlers 存handler 要執行的內容
defaults 為主要變數存放的地方（高於vars 目錄）
tasks    為主要執行的工作內容
templates 負責存放jinja2 模板檔
files 	  負責存放要呼叫或執行或複製的檔案	
vars	  存放變數的地方

```markdown
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

```

而紅帽有提供預設的role 在安裝來源中可以使用

```markdown
[student@hosta ansible]$ sudo yum install rhel-system-roles.noarch
Last metadata expiration check: 3:29:13 ago on Sun 14 Apr 2024 00:27:54 EDT.
Dependencies resolved.
============================================================================================================
 Package                       Architecture       Version                       Repository             Size
============================================================================================================
Installing:
 rhel-system-roles             noarch             1.23.0-2.21.el8               appstream             4.4 M

Transaction Summary
============================================================================================================
Install  1 Package

Total download size: 4.4 M

```

定義變數的方法
role 的變數設定在vars/main.yml，變數設定寫法爲{{ variable }}

使用role 的方式有三種，1.在劇本中使用role、2.在工作中使用include_role、3.在工作中使用import_role

在劇本中使用role 的方法爲下
透過這個方法，只要roles/role_name 目錄底下所有的檔案(tasks、handler、variable、template) 都會被playbook 給吃進來

```markdown

---
- hosts: webservers
  roles:
    - role_name
```

2.在工作中使用include_role

```markdown

---
- hosts: webservers
  tasks:
    - name: Include the foo_app_instance role
      include_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
```

3.在工作中使用import_role

```markdown
---
- hosts: webservers
  tasks:
    - name: Import the foo_app_instance role
      import_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
```

參考：[https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)

建立角色
透過使用ansible-galaxy init 指令 在roles 目錄之中建立一個自定義的role
定義roles 的內容，將其每一個工作分散寫在每一個劇本中

```markdown
[student@hosta roles]$ ansible-galaxy init apache
- Role apache was created successfully
```

主要的工作目錄中，編寫要執行的工作

```markdown
[student@hosta ansible]$ cat roles/apache/tasks/main.yml 
---
# tasks file for apache
- name: install webserver
  ansible.builtin.dnf:
    name: "{{ web_package }}"
    state: present

- name: enable web server
  ansible.builtin.service:
    name: "{{ web_package }}"
    enabled: true
    state: started

- name: firewall create
  ansible.posix.firewalld:
    service: http
    state: enabled
```

在handlers 目錄之中，撰寫最後要執行的工作

```markdown
[student@hosta ansible]$ cat roles/apache/handlers/main.yml 
---
# handlers file for apache

- name: restart httpd
  ansible.builtin.service:
    name: httpd
    state: restarted
```

然後再templates 目錄中放入模板檔

```markdown
[student@hosta ansible]$ cat roles/apache/templates/index.html.j2
This is apache role in {{ ansibe_facts.fqdn }}

This file is {{ owner }} write and config

```

然後變數放在default 之中

```markdown
[student@hosta ansible]$ cat roles/apache/defaults/main.yml 
---
# defaults file for apache
owner: jerry
```

而變數當然可以直接寫劇本裡提供給roles 使用，此時這裏變數的優先度，大於roles 裡面定義的

```markdown
---
- name: use create role
  hosts: webservers
  vars:
    - owner: marry
  roles:
    - role: apache

```

或

```markdown
---
- name: use create role
  hosts: webservers

  roles:
    - role: apache
      owner: marry

```

透過外部來源導入佈署的roles
除了自己撰寫role 以外，還可以用外面、社群提供撰寫的role 來導入到自己的project 之中，那就是ansible-galaxy
ansible-galaxy 除了提供角色以外還有提供collection 給你使用
到https://galaxy.ansible.com 去的尋找或用關鍵字找尋，自己可能需要的role
那假如，我們發現到一個喜歡的role，那麼最直接的寫法為 ansible-galaxy role install -o role_name ，-p 指定安裝的目錄

```markdown
[student@hosta ansible]$ ansible-galaxy role install bsmeding.nginx_docker  -p roles/
```

或是把他寫在yml檔裡面，ansible-galaxy install 加上-r 指定要透過哪一個檔案來告訴ansible galaxy 我要安裝哪些roles

```markdown
[student@hosta ansible]$ ansible-galaxy  install -r requirement.yml  -p roles/
```

而寫在yml 檔中，src、name、version
src: 表示我這個角色的來源在何方？
name: 抓下來的角色名稱是什麼
version: 這個角色的版本
scm: 這個角色要用什麼方式抓下來？　用git?、hg? 預設是用git

```markdown
# from galaxy
- name: yatesr.timezone

# from locally cloned git repository (git+file:// requires full paths)
- src: git+file:///home/bennojoy/nginx

# from GitHub
- src: https://github.com/bennojoy/nginx

# from GitHub, overriding the name and specifying a specific tag
- name: nginx_role
  src: https://github.com/bennojoy/nginx
  version: main

# from GitHub, specifying a specific commit hash
- src: https://github.com/bennojoy/nginx
  version: "ee8aa41"

# from a webserver, where the role is packaged in a tar.gz
- name: http-role-gz
  src: https://some.webserver.example.com/files/main.tar.gz

# from a webserver, where the role is packaged in a tar.bz2
- name: http-role-bz2
  src: https://some.webserver.example.com/files/main.tar.bz2

# from a webserver, where the role is packaged in a tar.xz (Python 3.x only)
- name: http-role-xz
  src: https://some.webserver.example.com/files/main.tar.xz

# from Bitbucket
- src: git+https://bitbucket.org/willthames/git-ansible-galaxy
  version: v1.4

# from Bitbucket, alternative syntax and caveats
- src: https://bitbucket.org/willthames/hg-ansible-galaxy
  scm: hg

# from GitLab or other git-based scm, using git+ssh
- src: git@gitlab.company.com:mygroup/ansible-core.git
  scm: git
  version: "0.1"  # quoted, so YAML doesn't parse this as a floating-point value

```

透過指令界面搜尋需要的角色

```markdown
[student@hosta ansible]$ ansible-galaxy search 'redis'

Found 232 roles matching your search:

 Name                                               Description
 ----                                               -----------
 aaronpederson.redis                                Redis is an open source, BSD licensed, advanced key-val>
 abarrak.redis_ansible_role                         Ansible role to install and configure redis instances b>
 aboveops.ct_redis                                  Ansible role for creating Redis container
 adfinis-sygroup.redis                              Ansible role for Redis
 adriano-di-giovanni.redis                          Ansible role for Redis
 AerisCloud.redis                                   Installs redis on a server
 alainvanhoof.alpine_redis                          Redis for Alpine Linux
 alexdzyoba.redis                                   Provision Redis for Debian as in official Docker image
 alikins.php_redis                                  PhpRedis support for Linux
 alikins.redis                                      Redis for Linux
 almaops.ct_redis                                   Ansible role deploys Redis container
 AlphaHydrae.redis-dev                              Installs Redis for development.
 andrewrothstein.redis                              builds Redis from src and installs
 Ar11rA.redis                                       Highly configurable role to install Redis and Redis Sen>
 AttestationLegale.redis                            Redis management
```

找到順眼的role就用info 進去看看他的說明

```markdown
[student@hosta ansible]$ ansible-galaxy info adriano-di-giovanni.redis
 
Role: adriano-di-giovanni.redis
        description: Ansible role for Redis
        commit: 0301cb97d967527750f09193c7c4ef847c70ddb6
        commit_message: changes:
* do not default redis_maxmemory_percentage to 70
* add `LimitNOFILE` to systemd script
        created: 2023-05-08T20:19:49.031352Z
        download_count: 46
        github_branch: master
        github_repo: ansible-role-redis
        github_user: adriano-di-giovanni
        id: 662
        imported: 2019-02-01T07:21:23.372334-05:00
        modified: 2023-10-29T18:44:34.842630Z
        path: ('/home/student/.ansible/roles', '/usr/share/ansible/roles', '/etc/ansible/roles')
        upstream_id: 13295
        username: adriano-di-giovanni

```

參考：
[https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles)

透過Collection 取得roles 與模組的功能
Collection 與Roles 差異觀點
自己再使用上Collection 觀念要視為"模組"的看法與使用方式，Roles 爲執行這個role 時，會呼叫哪些工作與該執行的內容（template 要不要到遠端主機、handlers 要執行....)有哪些
而ansible collection 則讓你模組與ansible的脫離核心的依賴(如不再依賴ansible-core)，這樣一來使得你不再依賴整個ansible-core 的更新，將你需要或想要的模組，直接導入、引用至你的project  中
而要支援ansible collection 則需要到ansible v2.9版以後才可以使用ansible collection

更可以說collection 是一個自定義的模組集合包

要使用collection ，我們必須要在ansible.cfg 設定檔中，加入collection 的路徑

```markdown
collections_path= ./collections

[student@hosta ansible]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = devops
ask_sudo_pass = False
ask_pass      = False
roles_path = ./roles
collections_path = ./collections:/usr/share/ansible/collections:/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles/
```

透過ansible-galaxy collection install 安裝collection

```markdown
ansible-galaxy collection install fedora.linux_system_roles
```

加上-p 來指定安裝路徑

```markdown
[student@hosta ansible]$ ansible-galaxy collection install fedora.linux_system_roles -p ./collections
Starting galaxy collection install process
[WARNING]: The specified collections path '/home/student/ansible/collections' is not part of the configured
Ansible collections paths '/home/student/.ansible/collections:/usr/share/ansible/collections'. The
installed collection will not be picked up in an Ansible run, unless within a playbook-adjacent collections
directory.
[WARNING]: Collection at '/usr/share/ansible/collections/ansible_collections/ovirt/ovirt' does not have a
MANIFEST.json file, nor has it galaxy.yml: cannot detect version.
Process install dependency map
 Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/fedora-linux_system_roles-1.76.2.tar.gz to /home/student/.ansible/tmp/ansible-local-69787c2m1edl7/tmpmlbr3u3w/fedora-linux_system_roles-1.76.2-6rzzsj10
Installing 'fedora.linux_system_roles:1.76.2' to '/home/student/ansible/collections/ansible_collections/fedora/linux_system_roles'
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/ansible-posix-1.5.4.tar.gz to /home/student/.ansible/tmp/ansible-local-69787c2m1edl7/tmpmlbr3u3w/ansible-posix-1.5.4-g1bpqfrv
fedora.linux_system_roles:1.76.2 was installed successfully
Installing 'ansible.posix:1.5.4' to '/home/student/ansible/collections/ansible_collections/ansible/posix'
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/containers-podman-1.12.1.tar.gz to /home/student/.ansible/tmp/ansible-local-69787c2m1edl7/tmpmlbr3u3w/containers-podman-1.12.1-k9_epkkv
ansible.posix:1.5.4 was installed successfully
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-general-8.5.0.tar.gz to /home/student/.ansible/tmp/ansible-local-69787c2m1edl7/tmpmlbr3u3w/community-general-8.5.0-ib4y3dy2
Installing 'containers.podman:1.12.1' to '/home/student/ansible/collections/ansible_collections/containers/podman'
containers.podman:1.12.1 was installed successfully
Installing 'community.general:8.5.0' to '/home/student/ansible/collections/ansible_collections/community/general'
community.general:8.5.0 was installed successfully

```

或是使用打包出來的collection，直接匯入到collection  目錄之中

```markdown
[student@hosta ansible]$ ansible-galaxy collection install ./freeipa-ansible_freeipa-1.12.1.tar.gz -p collections/ 
Starting galaxy collection install process
[WARNING]: The specified collections path '/home/student/ansible/collections' is not part of the configured
Ansible collections paths '/home/student/.ansible/collections:/usr/share/ansible/collections'. The
installed collection will not be picked up in an Ansible run, unless within a playbook-adjacent collections
directory.
[WARNING]: Collection at '/usr/share/ansible/collections/ansible_collections/ovirt/ovirt' does not have a
MANIFEST.json file, nor has it galaxy.yml: cannot detect version.
Process install dependency map
Starting collection install process
Installing 'freeipa.ansible_freeipa:1.12.1' to '/home/student/ansible/collections/ansible_collections/freeipa/ansible_freeipa'
freeipa.ansible_freeipa:1.12.1 was installed successfully

```

透過yml 檔指定安裝包

```markdown
---
collections:
  # Install a collection from Ansible Galaxy.
  - name: community.general
    version: ">=8.5"
    source: https://galaxy.ansible.com
```

或

```markdown
---
collections:
  - name: /tmp/my_namespace-my_collection-1.0.0.tar.gz
    type: file
```

執行結果

```markdown
[student@hosta ansible]$ ansible-galaxy collection install -r collection.yml -p collections/
Starting galaxy collection install process
[WARNING]: The specified collections path '/home/student/ansible/collections' is not part of the configured
Ansible collections paths '/home/student/.ansible/collections:/usr/share/ansible/collections'. The
installed collection will not be picked up in an Ansible run, unless within a playbook-adjacent collections
directory.
[WARNING]: Collection at '/usr/share/ansible/collections/ansible_collections/ovirt/ovirt' does not have a
MANIFEST.json file, nor has it galaxy.yml: cannot detect version.
Process install dependency map
community.general:8.5.0 was installed successfully
```

從git repository 中安裝collection

```markdown
# Install a collection in a repository using the latest commit on the branch 'devel'
ansible-galaxy collection install git+https://github.com/organization/repo_name.git,devel

# Install a collection from a private GitHub repository
ansible-galaxy collection install git@github.com:organization/repo_name.git

# Install a collection from a local git repository
ansible-galaxy collection install git+file:///home/user/path/to/repo_name.git
```

在ansible.cfg 中可以查看galaxy server的來源

```markdown
# Install a collection in a repository using the latest commit on the branch 'devel'
ansible-galaxy collection install git+https://github.com/organization/repo_name.git,devel

# Install a collection from a private GitHub repository
ansible-galaxy collection install git@github.com:organization/repo_name.git

# Install a collection from a local git repository
ansible-galaxy collection install git+file:///home/user/path/to/repo_name.git

在ansible.cfg 中可以查看galaxy server的來源
[galaxy]
# (path) The directory that stores cached responses from a Galaxy server.
# This is only used by the ``ansible-galaxy collection install`` and ``download`` commands.
# Cache files inside this dir will be ignored if they are world writable.
cache_dir={{ ANSIBLE_HOME ~ "/galaxy_cache" }}

# (list) patterns of files to ignore inside a Galaxy collection skeleton directory
collection_skeleton_ignore=^.git$, ^.*/.git_keep$

# (string) URL to prepend when roles don't specify the full URI, assume they are referencing this server as the source.
server=https://galaxy.ansible.com

# (list) A list of Galaxy servers to use when installing a collection.
# The value corresponds to the config ini header ``[galaxy_server.{{item}}]`` which defines the server details.
# See :ref:`galaxy_server_config` for more details on how to define a Galaxy server.
# The order of servers in this list is used to as the order in which a collection is resolved.
# Setting this config option will ignore the :ref:`galaxy_server` config option.
server_list=
```

如果要移除collection，直接刪除整個目錄就可以了

```markdown
rm -rf ~/.ansible/collections/ansible_collections/community/general
rm -rf ./venv/lib/python3.9/site-packages/ansible_collections/community/general
```

使用ansible connection 資源

以roles 的方式使用

```markdown
[student@hosta ansible]$ cat role_basic_collection.yml
---
- name: role use collection
  hosts: hostb.lab.example.com
  become: true
  vars:
    postfix_check: false

  tasks:
    - name: install posix
      ansible.builtin.dnf:
        name: postfix
        state: present

  roles:
    - fedora.linux_system_roles.postfix
```

執行結果

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout role_basic_collection.yml -C

PLAY [role use collection] *****************************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [fedora.linux_system_roles.postfix : Ensure ansible_facts required by role] ***
included: /home/student/ansible/collections/ansible_collections/fedora/linux_system_roles/roles/postfix/tasks/set_facts.yml for hostb.lab.example.com

TASK [fedora.linux_system_roles.postfix : Ensure ansible_facts used by role are present] ***
skipping: [hostb.lab.example.com]

TASK [fedora.linux_system_roles.postfix : Check if system is ostree] ***********
ok: [hostb.lab.example.com]
....

TASK [fedora.linux_system_roles.postfix : Install Postfix] *********************
ok: [hostb.lab.example.com]

TASK [fedora.linux_system_roles.postfix : Enable Postfix] **********************
changed: [hostb.lab.example.com]
TASK [install posix] ***********************************************************
ok: [hostb.lab.example.com]

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=9    changed=1    unreachable=0    failed=0    skipped=18   rescued=0    ignored=0

```

playbook 的方式使用collection

```yaml
---
- name: Reference a collection content using its FQCN
  hosts: all
  
  tasks:
    - name: Call a module using FQCN
      my_namespace.my_collection.my_module:
        option1: value

```

透過role 使用重複的工作
透過系統預設的role，即rhel-system-roles.noarch 去使用

```yaml
[student@hosta ansible]$ cat time.yml 
---
- name: default role
  hosts: hostb.lab.example.com
  become: true
  vars:
    timesync_ntp_servers:
      - hostname: 192.43.244.18
        iburst: true
  roles:
    - redhat.rhel_system_roles.timesync
```

參考:
[https://docs.ansible.com/ansible/latest/collections_guide/index.html](https://docs.ansible.com/ansible/latest/collections_guide/index.html)[https://galaxy.ansible.com/ui/repo/published/fedora/linux_system_roles/](https://galaxy.ansible.com/ui/repo/published/fedora/linux_system_roles/)[https://galaxy.ansible.com/ui/repo/published/freeipa/ansible_freeipa/](https://galaxy.ansible.com/ui/repo/published/freeipa/ansible_freeipa/)[https://docs.ansible.com/ansible/latest/collections_guide/index.html](https://docs.ansible.com/ansible/latest/collections_guide/index.html)[https://docs.ansible.com/ansible/latest/collections_guide/collections_using_playbooks.html](https://docs.ansible.com/ansible/latest/collections_guide/collections_using_playbooks.html)