# ch8

Troubleshooting Ansible

故障排除
當我們執行劇本的時候,可能會出現些錯誤
那要檢查錯誤,之前有提到使用-v'-vv'-vvv'-vvvv來顯示資訊

越多其實並非好事,適量即可，反倒應該觀察輸出的錯誤結果，細看原始狀態噴的錯誤，可能列的錯誤是什麼？並搭配其他指令，如--check 檢查你的拼字有沒有寫錯

-v

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout remove_file.yml -v
Using /home/student/ansible/ansible.cfg as config file

PLAY [file module remove file example] *****************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [remove file] *************************************************************
ok: [hostb.lab.example.com] => {"changed": false, "path": "/home/exampl_file.txt", "state": "absent"}

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

-vv

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout remove_file.yml -vv
ansible-playbook [core 2.16.2]
  config file = /home/student/ansible/ansible.cfg
  configured module search path = ['/home/runner/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.12/site-packages/ansible
  ansible collection location = /home/student/ansible/collections:/usr/share/ansible/collections:/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.12.1 (main, Dec 18 2023, 00:00:00) [GCC 13.2.1 20231205 (Red Hat 13.2.1-6)] (/usr/bin/python3)
  jinja version = 3.1.3
  libyaml = True
Using /home/student/ansible/ansible.cfg as config file
Skipping callback 'awx_display', as we already have a stdout callback.
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: remove_file.yml ******************************************************
1 plays in /home/student/ansible/remove_file.yml

PLAY [file module remove file example] *****************************************

TASK [Gathering Facts] *********************************************************
task path: /home/student/ansible/remove_file.yml:2
ok: [hostb.lab.example.com]

TASK [remove file] *************************************************************
task path: /home/student/ansible/remove_file.yml:7
ok: [hostb.lab.example.com] => {"changed": false, "path": "/home/exampl_file.txt", "state": "absent"}

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

-vvv

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout remove_file.yml -vvv
ansible-playbook [core 2.16.2]
  config file = /home/student/ansible/ansible.cfg
  configured module search path = ['/home/runner/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.12/site-packages/ansible
  ansible collection location = /home/student/ansible/collections:/usr/share/ansible/collections:/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles
  executable location = /usr/local/bin/ansible-playbook
  python version = 3.12.1 (main, Dec 18 2023, 00:00:00) [GCC 13.2.1 20231205 (Red Hat 13.2.1-6)] (/usr/bin/python3)
  jinja version = 3.1.3
  libyaml = True
Using /home/student/ansible/ansible.cfg as config file
host_list declined parsing /home/student/ansible/inventory as it did not pass its verify_file() method
script declined parsing /home/student/ansible/inventory as it did not pass its verify_file() method
auto declined parsing /home/student/ansible/inventory as it did not pass its verify_file() method
Parsed /home/student/ansible/inventory inventory source with ini plugin
Skipping callback 'awx_display', as we already have a stdout callback.
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: remove_file.yml ******************************************************
1 plays in /home/student/ansible/remove_file.yml

PLAY [file module remove file example] *****************************************

TASK [Gathering Facts] *********************************************************
task path: /home/student/ansible/remove_file.yml:2
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'echo ~devops && sleep 0'"'"''
<hostb.lab.example.com> (0, b'/home/devops\n', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'( umask 77 && mkdir -p "` echo /home/devops/.ansible/tmp `"&& mkdir "` echo /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340 `" && echo ansible-tmp-1713288074.0390089-26-251223453694340="` echo /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340 `" ) && sleep 0'"'"''
<hostb.lab.example.com> (0, b'ansible-tmp-1713288074.0390089-26-251223453694340=/home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340\n', b'')
<hostb.lab.example.com> Attempting python interpreter discovery
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'echo PLATFORM; uname; echo FOUND; command -v '"'"'"'"'"'"'"'"'python3.12'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.11'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.10'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.9'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.8'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.7'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python3.6'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'/usr/bin/python3'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'/usr/libexec/platform-python'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python2.7'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'/usr/bin/python'"'"'"'"'"'"'"'"'; command -v '"'"'"'"'"'"'"'"'python'"'"'"'"'"'"'"'"'; echo ENDFOUND && sleep 0'"'"''
<hostb.lab.example.com> (0, b'PLATFORM\nLinux\nFOUND\n/usr/bin/python3.6\n/usr/bin/python3\n/usr/libexec/platform-python\nENDFOUND\n', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'/usr/bin/python3.6 && sleep 0'"'"''
<hostb.lab.example.com> (0, b'{"platform_dist_result": ["centos", "8", ""], "osrelease_content": "NAME=\\"CentOS Stream\\"\\nVERSION=\\"8\\"\\nID=\\"centos\\"\\nID_LIKE=\\"rhel fedora\\"\\nVERSION_ID=\\"8\\"\\nPLATFORM_ID=\\"platform:el8\\"\\nPRETTY_NAME=\\"CentOS Stream 8\\"\\nANSI_COLOR=\\"0;31\\"\\nCPE_NAME=\\"cpe:/o:centos:centos:8\\"\\nHOME_URL=\\"https://centos.org/\\"\\nBUG_REPORT_URL=\\"https://bugzilla.redhat.com/\\"\\nREDHAT_SUPPORT_PRODUCT=\\"Red Hat Enterprise Linux 8\\"\\nREDHAT_SUPPORT_PRODUCT_VERSION=\\"CentOS Stream\\"\\n"}\n', b'')
Using module file /usr/local/lib/python3.12/site-packages/ansible/modules/setup.py
<hostb.lab.example.com> PUT /home/runner/.ansible/tmp/ansible-local-23lubrlmkg/tmpx9vpvgew TO /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/AnsiballZ_setup.py
<hostb.lab.example.com> SSH: EXEC sftp -b - -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' '[hostb.lab.example.com]'
<hostb.lab.example.com> (0, b'sftp> put /home/runner/.ansible/tmp/ansible-local-23lubrlmkg/tmpx9vpvgew /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/AnsiballZ_setup.py\n', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'chmod u+x /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/ /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/AnsiballZ_setup.py && sleep 0'"'"''
<hostb.lab.example.com> (0, b'', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' -tt hostb.lab.example.com '/bin/sh -c '"'"'sudo -H -S -n  -u root /bin/sh -c '"'"'"'"'"'"'"'"'echo BECOME-SUCCESS-brmijihaxtukmnlnzfsxfsgmctblrave ; /usr/libexec/platform-python /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/AnsiballZ_setup.py'"'"'"'"'"'"'"'"' && sleep 0'"'"''
Escalation succeeded
<hostb.lab.example.com> (0, b'\r\n{"ansible_facts": {"ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABgQCcl0eAz48S4YxDZL5FSdZMbVvV1zR5DX0Oa/fXWxgfK18DaBQQMMnZrYdNdq1ZeR3D6DEPp9pYAqVDHsRG8VHyd9zypc6mQIxVs4KBqliTvxmIKIsEerUP6UUd5iWwQqrXqJMkZhwebfZtm4xBey8Ecbayn5ZFdL0Wi5ZPriIZyuWNI5YSVwV/XZpImfU20pk3mWNSkwfoYTEerfsrc2WEZSBJWb3FUxXRCWyrbXkKTmhyffCmQHTPlXaDBqo+DAut6Do+aLTE9tDRXu831tA9t9AJYVBp/34w/oLLJYoXLbGBxpNDaOkD91KsywRETXxpcbUBf4QFV0WReqW+cBERqS5GpQV2N04pdIMKuqUAONeqI9J73Tx6SgMImbVzmjER6QCh6MDM4gpWmSO61UGefjDaLuOrZOc0bhfddyfLkpOC/nibah/7Y2VP9ibQBAoE465uUQvhsrUUUAqYH1AvU1T6H2wNH8kyIaPi2KG++yARbQNDXb9wfFuF0qQDRpE=", "ansible_ssh_host_key_rsa_public_keytype": "ssh-rsa", "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+GQrFmePManwZK5FU9HyajO4HtAZJEetNrpr6QcjHjfacPQcfPEZRKDycgu+KEtW6NkkDxSxGgaFZ/bRG1LwE=", "ansible_ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256", "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIM8AbNPD8Yxqq4oRghaOIXUKSj8S+iMj/sJHcUiLXuML", "ansible_ssh_host_key_ed25519_public_keytype": "ssh-ed25519", "ansible_distribution": "CentOS", "ansible_distribution_release": "Stream", "ansible_distribution_version": "8", "ansible_distribution_major_version": "8", "ansible_distribution_file_path": "/etc/centos-release", "ansible_distribution_file_variety": "CentOS", "ansible_distribution_file_parsed": true, "ansible_os_family": "RedHat", "ansible_env": {"LS_COLORS": "rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36:", "LANG": "en_US.UTF-8", "SUDO_GID": "1003", "SUDO_COMMAND": "/bin/sh -c echo BECOME-SUCCESS-brmijihaxtukmnlnzfsxfsgmctblrave ; /usr/libexec/platform-python /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/AnsiballZ_setup.py", "USER": "root", "PWD": "/home/devops", "HOME": "/root", "SUDO_USER": "devops", "SUDO_UID": "1002", "MAIL": "/var/mail/root", "SHELL": "/bin/bash", "TERM": "xterm", "SHLVL": "1", "LOGNAME": "root", "PATH": "/sbin:/bin:/usr/sbin:/usr/bin", "_": "/usr/libexec/platform-python"}, "ansible_hostnqn": "nqn.2014-08.org.nvmexpress:uuid:1af29640-d569-448e-ac31-3bef40e0f46e", "ansible_user_id": "root", "ansible_user_uid": 0, "ansible_user_gid": 0, "ansible_user_gecos": "root", "ansible_user_dir": "/root", "ansible_user_shell": "/bin/bash", "ansible_real_user_id": 0, "ansible_effective_user_id": 0, "ansible_real_group_id": 0, "ansible_effective_group_id": 0, "ansible_pkg_mgr": "dnf", "ansible_virtualization_role": "guest", "ansible_virtualization_type": "kvm", "ansible_virtualization_tech_guest": ["kvm"], "ansible_virtualization_tech_host": [], "ansible_system_capabilities_enforced": "False", "ansible_system_capabilities": [], "ansible_apparmor": {"status": "disabled"}, "ansible_loadavg": {"1m": 0.05, "5m": 0.03, "15m": 0.0}, "ansible_system": "Linux", "ansible_kernel": "4.18.0-500.el8.x86_64", "ansible_kernel_version": "#1 SMP Wed Jun 28 00:07:07 UTC 2023", "ansible_machine": "x86_64", "ansible_python_version": "3.6.8", "ansible_fqdn": "serverb.lab.example.com", "ansible_hostname": "hostb", "ansible_nodename": "hostb.lab.example.com", "ansible_domain": "lab.example.com", "ansible_userspace_bits": "64", "ansible_architecture": "x86_64", "ansible_userspace_architecture": "x86_64", "ansible_machine_id": "1af29640d569448eac313bef40e0f46e", "ansible_lsb": {}, "ansible_date_time": {"year": "2024", "month": "04", "weekday": "Tuesday", "weekday_number": "2", "weeknumber": "16", "day": "16", "hour": "13", "minute": "21", "second": "14", "epoch": "1713288074", "epoch_int": "1713288074", "date": "2024-04-16", "time": "13:21:14", "iso8601_micro": "2024-04-16T17:21:14.607699Z", "iso8601": "2024-04-16T17:21:14Z", "iso8601_basic": "20240416T132114607699", "iso8601_basic_short": "20240416T132114", "tz": "EDT", "tz_dst": "EDT", "tz_offset": "-0400"}, "ansible_fibre_channel_wwn": [], "ansible_processor": ["0", "AuthenticAMD", "AMD EPYC-Milan Processor", "1", "AuthenticAMD", "AMD EPYC-Milan Processor", "2", "AuthenticAMD", "AMD EPYC-Milan Processor", "3", "AuthenticAMD", "AMD EPYC-Milan Processor"], "ansible_processor_count": 4, "ansible_processor_cores": 1, "ansible_processor_threads_per_core": 1, "ansible_processor_vcpus": 4, "ansible_processor_nproc": 4, "ansible_memtotal_mb": 15720, "ansible_memfree_mb": 14084, "ansible_swaptotal_mb": 2099, "ansible_swapfree_mb": 2099, "ansible_memory_mb": {"real": {"total": 15720, "used": 1636, "free": 14084}, "nocache": {"free": 15005, "used": 715}, "swap": {"total": 2099, "free": 2099, "used": 0, "cached": 0}}, "ansible_bios_date": "04/01/2014", "ansible_bios_vendor": "SeaBIOS", "ansible_bios_version": "1.16.0-1.module_el8.7.0+1140+ff0772f9", "ansible_board_asset_tag": "NA", "ansible_board_name": "RHEL-AV", "ansible_board_serial": "NA", "ansible_board_vendor": "Red Hat", "ansible_board_version": "RHEL-8.6.0 PC (Q35 + ICH9, 2009)", "ansible_chassis_asset_tag": "NA", "ansible_chassis_serial": "NA", "ansible_chassis_vendor": "Red Hat", "ansible_chassis_version": "RHEL-8.6.0 PC (Q35 + ICH9, 2009)", "ansible_form_factor": "Other", "ansible_product_name": "KVM", "ansible_product_serial": "NA", "ansible_product_uuid": "61ef21ff-f53f-4a83-b02e-30ade5d31713", "ansible_product_version": "RHEL-8.6.0 PC (Q35 + ICH9, 2009)", "ansible_system_vendor": "Red Hat", "ansible_devices": {"dm-1": {"virtual": 1, "links": {"ids": ["dm-name-cs-swap", "dm-uuid-LVM-ZNOfzZfdA3X5OhOfUgJcMHnlhDzXYUwu6LpzxzaiFHq0K6GPrNHagSqkp0A3ABY7"], "uuids": ["e3f5eee0-9ad3-4d06-9111-30e0a6157d56"], "labels": [], "masters": []}, "vendor": null, "model": null, "sas_address": null, "sas_device_handle": null, "removable": "0", "support_discard": "512", "partitions": {}, "rotational": "1", "scheduler_mode": "", "sectors": "4300800", "sectorsize": "512", "size": "2.05 GB", "host": "", "holders": []}, "sr0": {"virtual": 1, "links": {"ids": ["ata-QEMU_DVD-ROM_QM00001"], "uuids": [], "labels": [], "masters": []}, "vendor": "QEMU", "model": "QEMU DVD-ROM", "sas_address": null, "sas_device_handle": null, "removable": "1", "support_discard": "0", "partitions": {}, "rotational": "1", "scheduler_mode": "mq-deadline", "sectors": "2097151", "sectorsize": "512", "size": "1024.00 MB", "host": "SATA controller: Intel Corporation 82801IR/IO/IH (ICH9R/DO/DH) 6 port SATA Controller [AHCI mode] (rev 02)", "holders": []}, "dm-0": {"virtual": 1, "links": {"ids": ["dm-name-cs-root", "dm-uuid-LVM-ZNOfzZfdA3X5OhOfUgJcMHnlhDzXYUwuOCzhzFRkqDYJHJ1240jG0dnPPloQWXbs"], "uuids": ["07b23dd1-e038-4324-a254-3f407f909c46"], "labels": [], "masters": []}, "vendor": null, "model": null, "sas_address": null, "sas_device_handle": null, "removable": "0", "support_discard": "512", "partitions": {}, "rotational": "1", "scheduler_mode": "", "sectors": "60702720", "sectorsize": "512", "size": "28.95 GB", "host": "", "holders": []}, "vda": {"virtual": 1, "links": {"ids": [], "uuids": [], "labels": [], "masters": []}, "vendor": "0x1af4", "model": null, "sas_address": null, "sas_device_handle": null, "removable": "0", "support_discard": "512", "partitions": {"vda2": {"links": {"ids": ["lvm-pv-uuid-R4lcg4-1eC0-Z1uc-CkCe-GaSE-iRST-RU3ktV"], "uuids": [], "labels": [], "masters": ["dm-0", "dm-1"]}, "start": "2099200", "sectors": "65009664", "sectorsize": 512, "size": "31.00 GB", "uuid": null, "holders": ["cs-swap", "cs-root"]}, "vda1": {"links": {"ids": [], "uuids": ["e4f0ddbd-8ab2-499b-acca-1542ec6d216b"], "labels": [], "masters": []}, "start": "2048", "sectors": "2097152", "sectorsize": 512, "size": "1.00 GB", "uuid": "e4f0ddbd-8ab2-499b-acca-1542ec6d216b", "holders": []}}, "rotational": "1", "scheduler_mode": "none", "sectors": "67108864", "sectorsize": "512", "size": "32.00 GB", "host": "SCSI storage controller: Red Hat, Inc. Virtio 1.0 block device (rev 01)", "holders": []}}, "ansible_device_links": {"ids": {"dm-1": ["dm-name-cs-swap", "dm-uuid-LVM-ZNOfzZfdA3X5OhOfUgJcMHnlhDzXYUwu6LpzxzaiFHq0K6GPrNHagSqkp0A3ABY7"], "dm-0": ["dm-name-cs-root", "dm-uuid-LVM-ZNOfzZfdA3X5OhOfUgJcMHnlhDzXYUwuOCzhzFRkqDYJHJ1240jG0dnPPloQWXbs"], "sr0": ["ata-QEMU_DVD-ROM_QM00001"], "vda2": ["lvm-pv-uuid-R4lcg4-1eC0-Z1uc-CkCe-GaSE-iRST-RU3ktV"]}, "uuids": {"dm-1": ["e3f5eee0-9ad3-4d06-9111-30e0a6157d56"], "dm-0": ["07b23dd1-e038-4324-a254-3f407f909c46"], "vda1": ["e4f0ddbd-8ab2-499b-acca-1542ec6d216b"]}, "labels": {}, "masters": {"vda2": ["dm-0", "dm-1"]}}, "ansible_uptime_seconds": 488037, "ansible_lvm": {"lvs": {"root": {"size_g": "28.95", "vg": "cs"}, "swap": {"size_g": "2.05", "vg": "cs"}}, "vgs": {"cs": {"size_g": "31.00", "free_g": "0", "num_lvs": "2", "num_pvs": "1"}}, "pvs": {"/dev/vda2": {"size_g": "31.00", "free_g": "0", "vg": "cs"}}}, "ansible_mounts": [{"mount": "/", "device": "/dev/mapper/cs-root", "fstype": "xfs", "options": "rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota", "size_total": 31064616960, "size_available": 19558866944, "block_size": 4096, "block_total": 7584135, "block_available": 4775114, "block_used": 2809021, "inode_total": 15175680, "inode_available": 15023935, "inode_used": 151745, "uuid": "07b23dd1-e038-4324-a254-3f407f909c46"}, {"mount": "/boot", "device": "/dev/vda1", "fstype": "xfs", "options": "rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota", "size_total": 1063256064, "size_available": 778854400, "block_size": 4096, "block_total": 259584, "block_available": 190150, "block_used": 69434, "inode_total": 524288, "inode_available": 523916, "inode_used": 372, "uuid": "e4f0ddbd-8ab2-499b-acca-1542ec6d216b"}], "ansible_dns": {"search": ["lab.example.com"], "nameservers": ["172.25.250.4", "8.8.8.8"]}, "ansible_fips": false, "ansible_local": {}, "ansible_python": {"version": {"major": 3, "minor": 6, "micro": 8, "releaselevel": "final", "serial": 0}, "version_info": [3, 6, 8, "final", 0], "executable": "/usr/libexec/platform-python", "has_sslcontext": true, "type": "cpython"}, "ansible_cmdline": {"BOOT_IMAGE": "(hd0,msdos1)/vmlinuz-4.18.0-500.el8.x86_64", "root": "/dev/mapper/cs-root", "ro": true, "crashkernel": "auto", "resume": "/dev/mapper/cs-swap", "rd.lvm.lv": "cs/swap", "rhgb": true, "quiet": true}, "ansible_proc_cmdline": {"BOOT_IMAGE": "(hd0,msdos1)/vmlinuz-4.18.0-500.el8.x86_64", "root": "/dev/mapper/cs-root", "ro": true, "crashkernel": "auto", "resume": "/dev/mapper/cs-swap", "rd.lvm.lv": ["cs/root", "cs/swap"], "rhgb": true, "quiet": true}, "ansible_is_chroot": false, "ansible_selinux_python_present": true, "ansible_selinux": {"status": "enabled", "policyvers": 33, "config_mode": "enforcing", "mode": "enforcing", "type": "targeted"}, "ansible_interfaces": ["enp1s0", "enp7s0", "lo", "enp8s0", "virbr0"], "ansible_enp8s0": {"device": "enp8s0", "macaddress": "52:54:00:e6:5e:79", "mtu": 1500, "active": true, "module": "virtio_net", "type": "ether", "pciid": "virtio6", "speed": -1, "promisc": false, "ipv4": {"address": "192.168.122.11", "broadcast": "192.168.122.255", "netmask": "255.255.255.0", "network": "192.168.122.0", "prefix": "24"}, "ipv6": [{"address": "fe80::7d5d:e270:408a:4861", "prefix": "64", "scope": "link"}], "features": {"rx_checksumming": "on [fixed]", "tx_checksumming": "on", "tx_checksum_ipv4": "off [fixed]", "tx_checksum_ip_generic": "on", "tx_checksum_ipv6": "off [fixed]", "tx_checksum_fcoe_crc": "off [fixed]", "tx_checksum_sctp": "off [fixed]", "scatter_gather": "on", "tx_scatter_gather": "on", "tx_scatter_gather_fraglist": "off [fixed]", "tcp_segmentation_offload": "on", "tx_tcp_segmentation": "on", "tx_tcp_ecn_segmentation": "on", "tx_tcp_mangleid_segmentation": "off", "tx_tcp6_segmentation": "on", "generic_segmentation_offload": "on", "generic_receive_offload": "on", "large_receive_offload": "off [fixed]", "rx_vlan_offload": "off [fixed]", "tx_vlan_offload": "off [fixed]", "ntuple_filters": "off [fixed]", "receive_hashing": "off [fixed]", "highdma": "on [fixed]", "rx_vlan_filter": "on [fixed]", "vlan_challenged": "off [fixed]", "tx_lockless": "off [fixed]", "netns_local": "off [fixed]", "tx_gso_robust": "on [fixed]", "tx_fcoe_segmentation": "off [fixed]", "tx_gre_segmentation": "off [fixed]", "tx_gre_csum_segmentation": "off [fixed]", "tx_ipxip4_segmentation": "off [fixed]", "tx_ipxip6_segmentation": "off [fixed]", "tx_udp_tnl_segmentation": "off [fixed]", "tx_udp_tnl_csum_segmentation": "off [fixed]", "tx_gso_partial": "off [fixed]", "tx_tunnel_remcsum_segmentation": "off [fixed]", "tx_sctp_segmentation": "off [fixed]", "tx_esp_segmentation": "off [fixed]", "tx_udp_segmentation": "off [fixed]", "tx_gso_list": "off [fixed]", "rx_udp_gro_forwarding": "off", "rx_gro_list": "off", "tls_hw_rx_offload": "off [fixed]", "fcoe_mtu": "off [fixed]", "tx_nocache_copy": "off", "loopback": "off [fixed]", "rx_fcs": "off [fixed]", "rx_all": "off [fixed]", "tx_vlan_stag_hw_insert": "off [fixed]", "rx_vlan_stag_hw_parse": "off [fixed]", "rx_vlan_stag_filter": "off [fixed]", "l2_fwd_offload": "off [fixed]", "hw_tc_offload": "off [fixed]", "esp_hw_offload": "off [fixed]", "esp_tx_csum_hw_offload": "off [fixed]", "rx_udp_tunnel_port_offload": "off [fixed]", "tls_hw_tx_offload": "off [fixed]", "rx_gro_hw": "off [fixed]", "tls_hw_record": "off [fixed]"}, "timestamping": [], "hw_timestamp_filters": []}, "ansible_lo": {"device": "lo", "mtu": 65536, "active": true, "type": "loopback", "promisc": false, "ipv4": {"address": "127.0.0.1", "broadcast": "", "netmask": "255.0.0.0", "network": "127.0.0.0", "prefix": "8"}, "ipv6": [{"address": "::1", "prefix": "128", "scope": "host"}], "features": {"rx_checksumming": "on [fixed]", "tx_checksumming": "on", "tx_checksum_ipv4": "off [fixed]", "tx_checksum_ip_generic": "on [fixed]", "tx_checksum_ipv6": "off [fixed]", "tx_checksum_fcoe_crc": "off [fixed]", "tx_checksum_sctp": "on [fixed]", "scatter_gather": "on", "tx_scatter_gather": "on [fixed]", "tx_scatter_gather_fraglist": "on [fixed]", "tcp_segmentation_offload": "on", "tx_tcp_segmentation": "on", "tx_tcp_ecn_segmentation": "on", "tx_tcp_mangleid_segmentation": "on", "tx_tcp6_segmentation": "on", "generic_segmentation_offload": "on", "generic_receive_offload": "on", "large_receive_offload": "off [fixed]", "rx_vlan_offload": "off [fixed]", "tx_vlan_offload": "off [fixed]", "ntuple_filters": "off [fixed]", "receive_hashing": "off [fixed]", "highdma": "on [fixed]", "rx_vlan_filter": "off [fixed]", "vlan_challenged": "on [fixed]", "tx_lockless": "on [fixed]", "netns_local": "on [fixed]", "tx_gso_robust": "off [fixed]", "tx_fcoe_segmentation": "off [fixed]", "tx_gre_segmentation": "off [fixed]", "tx_gre_csum_segmentation": "off [fixed]", "tx_ipxip4_segmentation": "off [fixed]", "tx_ipxip6_segmentation": "off [fixed]", "tx_udp_tnl_segmentation": "off [fixed]", "tx_udp_tnl_csum_segmentation": "off [fixed]", "tx_gso_partial": "off [fixed]", "tx_tunnel_remcsum_segmentation": "off [fixed]", "tx_sctp_segmentation": "on", "tx_esp_segmentation": "off [fixed]", "tx_udp_segmentation": "on", "tx_gso_list": "on", "rx_udp_gro_forwarding": "off", "rx_gro_list": "off", "tls_hw_rx_offload": "off [fixed]", "fcoe_mtu": "off [fixed]", "tx_nocache_copy": "off [fixed]", "loopback": "on [fixed]", "rx_fcs": "off [fixed]", "rx_all": "off [fixed]", "tx_vlan_stag_hw_insert": "off [fixed]", "rx_vlan_stag_hw_parse": "off [fixed]", "rx_vlan_stag_filter": "off [fixed]", "l2_fwd_offload": "off [fixed]", "hw_tc_offload": "off [fixed]", "esp_hw_offload": "off [fixed]", "esp_tx_csum_hw_offload": "off [fixed]", "rx_udp_tunnel_port_offload": "off [fixed]", "tls_hw_tx_offload": "off [fixed]", "rx_gro_hw": "off [fixed]", "tls_hw_record": "off [fixed]"}, "timestamping": [], "hw_timestamp_filters": []}, "ansible_enp1s0": {"device": "enp1s0", "macaddress": "52:54:00:bd:d1:63", "mtu": 1500, "active": true, "module": "virtio_net", "type": "ether", "pciid": "virtio0", "speed": -1, "promisc": false, "ipv4": {"address": "172.25.250.11", "broadcast": "172.25.250.255", "netmask": "255.255.255.0", "network": "172.25.250.0", "prefix": "24"}, "ipv6": [{"address": "fe80::5054:ff:febd:d163", "prefix": "64", "scope": "link"}], "features": {"rx_checksumming": "on [fixed]", "tx_checksumming": "on", "tx_checksum_ipv4": "off [fixed]", "tx_checksum_ip_generic": "on", "tx_checksum_ipv6": "off [fixed]", "tx_checksum_fcoe_crc": "off [fixed]", "tx_checksum_sctp": "off [fixed]", "scatter_gather": "on", "tx_scatter_gather": "on", "tx_scatter_gather_fraglist": "off [fixed]", "tcp_segmentation_offload": "on", "tx_tcp_segmentation": "on", "tx_tcp_ecn_segmentation": "on", "tx_tcp_mangleid_segmentation": "off", "tx_tcp6_segmentation": "on", "generic_segmentation_offload": "on", "generic_receive_offload": "on", "large_receive_offload": "off [fixed]", "rx_vlan_offload": "off [fixed]", "tx_vlan_offload": "off [fixed]", "ntuple_filters": "off [fixed]", "receive_hashing": "off [fixed]", "highdma": "on [fixed]", "rx_vlan_filter": "on [fixed]", "vlan_challenged": "off [fixed]", "tx_lockless": "off [fixed]", "netns_local": "off [fixed]", "tx_gso_robust": "on [fixed]", "tx_fcoe_segmentation": "off [fixed]", "tx_gre_segmentation": "off [fixed]", "tx_gre_csum_segmentation": "off [fixed]", "tx_ipxip4_segmentation": "off [fixed]", "tx_ipxip6_segmentation": "off [fixed]", "tx_udp_tnl_segmentation": "off [fixed]", "tx_udp_tnl_csum_segmentation": "off [fixed]", "tx_gso_partial": "off [fixed]", "tx_tunnel_remcsum_segmentation": "off [fixed]", "tx_sctp_segmentation": "off [fixed]", "tx_esp_segmentation": "off [fixed]", "tx_udp_segmentation": "off [fixed]", "tx_gso_list": "off [fixed]", "rx_udp_gro_forwarding": "off", "rx_gro_list": "off", "tls_hw_rx_offload": "off [fixed]", "fcoe_mtu": "off [fixed]", "tx_nocache_copy": "off", "loopback": "off [fixed]", "rx_fcs": "off [fixed]", "rx_all": "off [fixed]", "tx_vlan_stag_hw_insert": "off [fixed]", "rx_vlan_stag_hw_parse": "off [fixed]", "rx_vlan_stag_filter": "off [fixed]", "l2_fwd_offload": "off [fixed]", "hw_tc_offload": "off [fixed]", "esp_hw_offload": "off [fixed]", "esp_tx_csum_hw_offload": "off [fixed]", "rx_udp_tunnel_port_offload": "off [fixed]", "tls_hw_tx_offload": "off [fixed]", "rx_gro_hw": "off [fixed]", "tls_hw_record": "off [fixed]"}, "timestamping": [], "hw_timestamp_filters": []}, "ansible_virbr0": {"device": "virbr0", "macaddress": "52:54:00:1c:a2:45", "mtu": 1500, "active": false, "type": "bridge", "interfaces": [], "id": "8000.5254001ca245", "stp": true, "speed": -1, "promisc": false, "ipv4": {"address": "192.168.222.1", "broadcast": "192.168.222.255", "netmask": "255.255.255.0", "network": "192.168.222.0", "prefix": "24"}, "features": {"rx_checksumming": "off [fixed]", "tx_checksumming": "on", "tx_checksum_ipv4": "off [fixed]", "tx_checksum_ip_generic": "on", "tx_checksum_ipv6": "off [fixed]", "tx_checksum_fcoe_crc": "off [fixed]", "tx_checksum_sctp": "off [fixed]", "scatter_gather": "on", "tx_scatter_gather": "on", "tx_scatter_gather_fraglist": "on", "tcp_segmentation_offload": "on", "tx_tcp_segmentation": "on", "tx_tcp_ecn_segmentation": "on", "tx_tcp_mangleid_segmentation": "on", "tx_tcp6_segmentation": "on", "generic_segmentation_offload": "on", "generic_receive_offload": "on", "large_receive_offload": "off [fixed]", "rx_vlan_offload": "off [fixed]", "tx_vlan_offload": "on", "ntuple_filters": "off [fixed]", "receive_hashing": "off [fixed]", "highdma": "on", "rx_vlan_filter": "off [fixed]", "vlan_challenged": "off [fixed]", "tx_lockless": "on [fixed]", "netns_local": "on [fixed]", "tx_gso_robust": "on", "tx_fcoe_segmentation": "on", "tx_gre_segmentation": "on", "tx_gre_csum_segmentation": "on", "tx_ipxip4_segmentation": "on", "tx_ipxip6_segmentation": "on", "tx_udp_tnl_segmentation": "on", "tx_udp_tnl_csum_segmentation": "on", "tx_gso_partial": "on", "tx_tunnel_remcsum_segmentation": "on", "tx_sctp_segmentation": "on", "tx_esp_segmentation": "on", "tx_udp_segmentation": "on", "tx_gso_list": "on", "rx_udp_gro_forwarding": "off", "rx_gro_list": "off", "tls_hw_rx_offload": "off [fixed]", "fcoe_mtu": "off [fixed]", "tx_nocache_copy": "off", "loopback": "off [fixed]", "rx_fcs": "off [fixed]", "rx_all": "off [fixed]", "tx_vlan_stag_hw_insert": "on", "rx_vlan_stag_hw_parse": "off [fixed]", "rx_vlan_stag_filter": "off [fixed]", "l2_fwd_offload": "off [fixed]", "hw_tc_offload": "off [fixed]", "esp_hw_offload": "off [fixed]", "esp_tx_csum_hw_offload": "off [fixed]", "rx_udp_tunnel_port_offload": "off [fixed]", "tls_hw_tx_offload": "off [fixed]", "rx_gro_hw": "off [fixed]", "tls_hw_record": "off [fixed]"}, "timestamping": [], "hw_timestamp_filters": []}, "ansible_enp7s0": {"device": "enp7s0", "macaddress": "52:54:00:ec:47:55", "mtu": 1500, "active": true, "module": "virtio_net", "type": "ether", "pciid": "virtio5", "speed": -1, "promisc": false, "features": {"rx_checksumming": "on [fixed]", "tx_checksumming": "on", "tx_checksum_ipv4": "off [fixed]", "tx_checksum_ip_generic": "on", "tx_checksum_ipv6": "off [fixed]", "tx_checksum_fcoe_crc": "off [fixed]", "tx_checksum_sctp": "off [fixed]", "scatter_gather": "on", "tx_scatter_gather": "on", "tx_scatter_gather_fraglist": "off [fixed]", "tcp_segmentation_offload": "on", "tx_tcp_segmentation": "on", "tx_tcp_ecn_segmentation": "on", "tx_tcp_mangleid_segmentation": "off", "tx_tcp6_segmentation": "on", "generic_segmentation_offload": "on", "generic_receive_offload": "on", "large_receive_offload": "off [fixed]", "rx_vlan_offload": "off [fixed]", "tx_vlan_offload": "off [fixed]", "ntuple_filters": "off [fixed]", "receive_hashing": "off [fixed]", "highdma": "on [fixed]", "rx_vlan_filter": "on [fixed]", "vlan_challenged": "off [fixed]", "tx_lockless": "off [fixed]", "netns_local": "off [fixed]", "tx_gso_robust": "on [fixed]", "tx_fcoe_segmentation": "off [fixed]", "tx_gre_segmentation": "off [fixed]", "tx_gre_csum_segmentation": "off [fixed]", "tx_ipxip4_segmentation": "off [fixed]", "tx_ipxip6_segmentation": "off [fixed]", "tx_udp_tnl_segmentation": "off [fixed]", "tx_udp_tnl_csum_segmentation": "off [fixed]", "tx_gso_partial": "off [fixed]", "tx_tunnel_remcsum_segmentation": "off [fixed]", "tx_sctp_segmentation": "off [fixed]", "tx_esp_segmentation": "off [fixed]", "tx_udp_segmentation": "off [fixed]", "tx_gso_list": "off [fixed]", "rx_udp_gro_forwarding": "off", "rx_gro_list": "off", "tls_hw_rx_offload": "off [fixed]", "fcoe_mtu": "off [fixed]", "tx_nocache_copy": "off", "loopback": "off [fixed]", "rx_fcs": "off [fixed]", "rx_all": "off [fixed]", "tx_vlan_stag_hw_insert": "off [fixed]", "rx_vlan_stag_hw_parse": "off [fixed]", "rx_vlan_stag_filter": "off [fixed]", "l2_fwd_offload": "off [fixed]", "hw_tc_offload": "off [fixed]", "esp_hw_offload": "off [fixed]", "esp_tx_csum_hw_offload": "off [fixed]", "rx_udp_tunnel_port_offload": "off [fixed]", "tls_hw_tx_offload": "off [fixed]", "rx_gro_hw": "off [fixed]", "tls_hw_record": "off [fixed]"}, "timestamping": [], "hw_timestamp_filters": []}, "ansible_default_ipv4": {"gateway": "192.168.122.1", "interface": "enp8s0", "address": "192.168.122.11", "broadcast": "192.168.122.255", "netmask": "255.255.255.0", "network": "192.168.122.0", "prefix": "24", "macaddress": "52:54:00:e6:5e:79", "mtu": 1500, "type": "ether", "alias": "enp8s0"}, "ansible_default_ipv6": {}, "ansible_all_ipv4_addresses": ["192.168.122.11", "172.25.250.11", "192.168.222.1"], "ansible_all_ipv6_addresses": ["fe80::7d5d:e270:408a:4861", "fe80::5054:ff:febd:d163"], "ansible_locally_reachable_ips": {"ipv4": ["127.0.0.0/8", "127.0.0.1", "172.25.250.11", "192.168.122.11", "192.168.222.1"], "ipv6": ["::1", "fe80::5054:ff:febd:d163", "fe80::7d5d:e270:408a:4861"]}, "ansible_iscsi_iqn": "iqn.1994-05.com.redhat:90c44741fcbb", "ansible_service_mgr": "systemd", "gather_subset": ["all"], "module_setup": true}, "invocation": {"module_args": {"gather_subset": ["all"], "gather_timeout": 10, "filter": [], "fact_path": "/etc/ansible/facts.d"}}}\r\n', b'Shared connection to hostb.lab.example.com closed.\r\n')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'rm -f -r /home/devops/.ansible/tmp/ansible-tmp-1713288074.0390089-26-251223453694340/ > /dev/null 2>&1 && sleep 0'"'"''
<hostb.lab.example.com> (0, b'', b'')
ok: [hostb.lab.example.com]

TASK [remove file] *************************************************************
task path: /home/student/ansible/remove_file.yml:7
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'echo ~devops && sleep 0'"'"''
<hostb.lab.example.com> (0, b'/home/devops\n', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'( umask 77 && mkdir -p "` echo /home/devops/.ansible/tmp `"&& mkdir "` echo /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804 `" && echo ansible-tmp-1713288075.0837088-39-77436140596804="` echo /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804 `" ) && sleep 0'"'"''
<hostb.lab.example.com> (0, b'ansible-tmp-1713288075.0837088-39-77436140596804=/home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804\n', b'')
Using module file /usr/local/lib/python3.12/site-packages/ansible/modules/file.py
<hostb.lab.example.com> PUT /home/runner/.ansible/tmp/ansible-local-23lubrlmkg/tmpotae1y29 TO /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/AnsiballZ_file.py
<hostb.lab.example.com> SSH: EXEC sftp -b - -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' '[hostb.lab.example.com]'
<hostb.lab.example.com> (0, b'sftp> put /home/runner/.ansible/tmp/ansible-local-23lubrlmkg/tmpotae1y29 /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/AnsiballZ_file.py\n', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'chmod u+x /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/ /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/AnsiballZ_file.py && sleep 0'"'"''
<hostb.lab.example.com> (0, b'', b'')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' -tt hostb.lab.example.com '/bin/sh -c '"'"'sudo -H -S -n  -u root /bin/sh -c '"'"'"'"'"'"'"'"'echo BECOME-SUCCESS-osfhzyrbftahzjaszfqqljedeyrrsmgn ; /usr/libexec/platform-python /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/AnsiballZ_file.py'"'"'"'"'"'"'"'"' && sleep 0'"'"''
Escalation succeeded
<hostb.lab.example.com> (0, b'\r\n{"path": "/home/exampl_file.txt", "changed": false, "state": "absent", "invocation": {"module_args": {"path": "/home/exampl_file.txt", "state": "absent", "recurse": false, "force": false, "follow": true, "modification_time_format": "%Y%m%d%H%M.%S", "access_time_format": "%Y%m%d%H%M.%S", "unsafe_writes": false, "_original_basename": null, "_diff_peek": null, "src": null, "modification_time": null, "access_time": null, "mode": null, "owner": null, "group": null, "seuser": null, "serole": null, "selevel": null, "setype": null, "attributes": null}}}\r\n', b'Shared connection to hostb.lab.example.com closed.\r\n')
<hostb.lab.example.com> ESTABLISH SSH CONNECTION FOR USER: devops
<hostb.lab.example.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="devops"' -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/ac260acc93"' hostb.lab.example.com '/bin/sh -c '"'"'rm -f -r /home/devops/.ansible/tmp/ansible-tmp-1713288075.0837088-39-77436140596804/ > /dev/null 2>&1 && sleep 0'"'"''
<hostb.lab.example.com> (0, b'', b'')
ok: [hostb.lab.example.com] => {
    "changed": false,
    "invocation": {
        "module_args": {
            "_diff_peek": null,
            "_original_basename": null,
            "access_time": null,
            "access_time_format": "%Y%m%d%H%M.%S",
            "attributes": null,
            "follow": true,
            "force": false,
            "group": null,
            "mode": null,
            "modification_time": null,
            "modification_time_format": "%Y%m%d%H%M.%S",
            "owner": null,
            "path": "/home/exampl_file.txt",
            "recurse": false,
            "selevel": null,
            "serole": null,
            "setype": null,
            "seuser": null,
            "src": null,
            "state": "absent",
            "unsafe_writes": false
        }
    },
    "path": "/home/exampl_file.txt",
    "state": "absent"
}

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

除此之外,還可以使用debug模組,將資訊透過debug模組打出來

將我們上一個儲存的變數存起來或是觀察facts,再將變數打出來，觀察資訊是否爲我們需要的

```yaml
 - name: Prints various Ansible facts 
   ansible.builtin.debug:
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

或是使用--syntax-check僅檢查你的劇本有沒有拼錯字
也可以檢查劇本輸出後,出現的json檔

```yaml
[student@hosta ansible]$ ansible-navigator run -m stdout role_basic_collection.yml  --syntax-check
playbook: /home/student/ansible/role_basic_collection.yml
```

若想要也可以不加上-m,改用ansible-naviagtor replay去查看執行結果
裡面也詳細指出錯誤的訊息（res欄位）,對比直接輸出來的好

那因為我們執行ansible的身份,有時候不一定可以提升權限到/var/log/目錄之中查看
所以我們在ansible.cfg中,會加上log_path,制定log輸出的位置

```markdown
[student@hosta ansible]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = devops
ask_sudo_pass = False
ask_pass      = False
roles_path = ./roles
collections_path = ./collections:/usr/share/ansible/collections:/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles/
log_path = /home/student/ansible/
```

管理ansible host時的故障排除
其中自己架設環境時,常發現的是,遠端連線過去,結果權限不對.
其中一個是remote_user沒有設定,第二個是.ssh的目錄權限不對,第三個該名ansible賬號的家目錄變成777所以無法使用,第四個是sudoers檔中,ansible賬號沒有設定成不用密碼提權的格式
（參考第一章設定的部分）

因此最常還是會使用--check ,句尾加上-C去執行
或是在劇本中根模組同一層加上check_mode: yes

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
      check_mode: yes
        name: postfix
        state: present
```

在句尾加上--diff 則會將被管理主機所作的改變記錄下來

```markdown
[student@hosta ansible]$ ansible-navigator run -m stdout remove_file.yml --diff

PLAY [file module remove file example] *****************************************

TASK [Gathering Facts] *********************************************************
ok: [hostb.lab.example.com]

TASK [remove file] *************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/home/exampl_file.txt",
-    "state": "file"
+    "state": "absent"
 }
changed: [hostb.lab.example.com]

PLAY RECAP *********************************************************************
hostb.lab.example.com      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

還有使用script模組,透過將本project的script丟到遠端被管理主機上,透過shell執行,
而如果回傳code非0就代表script失敗

```markdown
- name: Run a script with arguments (free form)
  ansible.builtin.script: /some/local/script.sh --some-argument 1234
```

更甚者直接使用ad hoc,直接跑ansible command也不用寫劇本直接執行
（此處用ping模組來代替）

```markdown
[student@hosta ansible]$ ansible all -i inventory -m ping 
hostc.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
hostb.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
hosta.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
hostd.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}

```

參考：
[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/script_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/script_module.html)