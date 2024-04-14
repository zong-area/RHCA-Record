# ch1

1. Ansible 簡介

舊版Ansible-playbook 與Ansible-navigator
管理員自動化管理工具ansible 是一種用來加速實現自動化的一種管理工具，他可用於應付多個不同類型的主機/節點(如：Windows, UNIX,Linux,或是網路設備），進行作業
並且可以透過執行模組，來對各別主機進行操作，如軟體更新、更改組態、呼叫API（基本上使用的語言爲python，對windows 主機則使用powershell)
呼叫的方式，Linux與UNIX 皆是使用ssh進行溝通，Windows 則是使用WinRM
而ansible 所跑的劇本，則是一種名叫YAML 的語言。而YAML 是一種人類易於讀懂得的語言，方便人們進行編輯與維護

其中最重要的是核心零件爲Ansible-core，提供主要平常使用的模組，給ansible 使用，模組名稱爲ansible.builtin.module_name
另外還有社群提供的模組或角色，可以用來擴充自己ansible 可用的功能

對比於舊版本ansible-playbook,新的工具ansible-navigator，爲ansible平臺2 的工具，它包含了ansible-playbook、ansible-inventory、ansible-config的功能。
因為隨著發展，ansible 所包含的模組越來越多，超乎想像。對於管理或支援上，有了不小的困擾。也因為有些想要使用特定版本或比當前ansible-core 打包更新/更舊的版本（可能較穩定或相依性的問題）因此透過ansible-navigator，可以更容易將社群和額外核心的功能/模組加入其中，將其帶入到自己的工作環境或production 環境中。

安裝

另外如果要使用ansible-playbook 的指令，你會需要安裝ansoble-core 的套件，但如果你已經安裝ansible-navigator，大預設也會安裝ansible-core
參考社群版本安裝方式

1. 需要使用容器管理工具，Docker 或podman 則一即可
2. 確認安裝 python3-pip 套件

```markdown
[root@hosta ~]# dnf install python3-pip
Last metadata expiration check: 0:45:02 ago on Fri 05 Apr 2024 03:08:45 AM EDT.
Package python3-pip-9.0.3-22.el8.noarch is already installed.
Dependencies resolved.
======================================================================================================
 Package                        Architecture      Version                  Repository            Size
======================================================================================================
Upgrading:
 platform-python                x86_64            3.6.8-59.el8             baseos                87 k
 platform-python-pip            noarch            9.0.3-24.el8             baseos               1.7 M
 python3-libs                   x86_64            3.6.8-59.el8             baseos               8.4 M
 python3-pip                    noarch            9.0.3-24.el8             appstream             20 k

Transaction Summary
======================================================================================================
Upgrade  4 Packages
```

1. 使用pip 安裝ansible-navigator

```markdown
[student@hosta ~]$  python3 -m pip install ansible-navigator --user
WARNING: Running pip install with root privileges is generally not a good idea. Try `__main__.py install --user` instead.
Collecting ansible-navigator
.........
Requirement already satisfied: pycparser in /usr/lib/python3.6/site-packages (from cffi>=1->onigurumacffi<2,>=1.1.0->ansible-navigator)
Installing collected packages: pexpect, pyparsing, packaging, ansible-runner, dataclasses, zipp, importlib-resources, onigurumacffi, ansible-navigator
Successfully installed ansible-navigator-1.1.0 ansible-runner-2.3.2 dataclasses-0.8 importlib-resources-5.4.0 onigurumacffi-1.1.0 packaging-21.3 pexpect-4.9.0 pyparsing-3.1.2 zipp-3.6.0
```

1. 設定環境變數

```markdown
[student@hosta ~]$  echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.profile
[student@hosta ~]$ source ~/.profile
```

5.且記，此處一定要用ansible 執行時的身份登入，不可以用su 進行切換，否則podman  會無法下載容器映像檔

並且使用podman images 檢查映像檔是否下載下來

```markdown
[student@hosta ~]$  ansible-navigator 
----------------------------------------------------------------------------------
Execution environment image and pull policy overview
----------------------------------------------------------------------------------
Execution environment image name:  quay.io/ansible/ansible-navigator-demo-ee:0.6.0
Execution environment image tag:   0.6.0
Execution environment pull policy: tag
Execution environment pull needed: True
----------------------------------------------------------------------------------
Updating the execution environment
...
Writing manifest to image destination
Storing signatures
4405e824c556ff708afbd0de144373cac250135d892e178941fe7c690e78814c

[student@hosta ~]$ podman images
REPOSITORY                                 TAG         IMAGE ID      CREATED       SIZE
ghcr.io/ansible/creator-ee                 v0.22.0     4405e824c556  2 months ago  869 MB
quay.io/ansible/ansible-navigator-demo-ee  0.6.0       e65e4777caa3  2 years ago   1.35 GB

```

故障排除

如果python3.6 會出現SyntaxError: future feature annotations is not defined 的錯誤

```markdown
 File "/home/student/.local/lib/python3.6/site-packages/ansible_navigator/runner/api.py", line 17, in <module>
    from ansible_runner import Runner  # type: ignore
  File "/home/student/.local/lib/python3.6/site-packages/ansible_runner/__init__.py", line 3, in <module>
    from .interface import run, run_async, \
  File "/home/student/.local/lib/python3.6/site-packages/ansible_runner/interface.py", line 33, in <module>
    from ansible_runner.streaming import Transmitter, Worker, Processor
  File "/home/student/.local/lib/python3.6/site-packages/ansible_runner/streaming.py", line 1
    from __future__ import annotations  # allow newer type syntax until 3.10 is our minimum
    ^
SyntaxError: future feature annotations is not defined
```

使用update-alternatives --config python3 選擇要更換的python3 版本

```markdown
[root@hosta ~]# python3 --version
Python 3.6.8
[student@hosta ~]$ sudo update-alternatives --config python3

There is 2 program that provides 'python3'.

  Selection    Command
-----------------------------------------------
*+ 1           /usr/bin/python3.6
   2           /usr/bin/python3.9

Enter to keep the current selection[+], or type selection number: 2
[student@hosta ~]$ python3 --version
Python 3.9.18
[student@hosta ~]$ 

```