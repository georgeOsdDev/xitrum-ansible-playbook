# Ansible playbook for quick deploy Xitrum

This is an ansible playbook for tunned Xitrum server.
Tunning settings are based on [Xitrum-doc](https://github.com/xitrum-framework/xitrum-doc)

This playbook supports Xitrum v3.18. (in master branch)

If you want to use older version of Xitrum Application.
Make sure that config files in `roles/xitrum/files` and its template files in `roles/xitrum/templates`
are match format to version you want to use.


# Usage

## Edit host ips or hostname in `playbooks/hosts`

```
[xitrum_servers]
xx.xx.xx.xx
ec2-xx-xx-xx-xx.ap-northeast-1.compute.amazonaws.com
```

## Edit variables in `playbooks/group_vars/xitrum_servers`

You can define some vars in `playbooks/group_vars/xitrum_servers` or create `hosts_vars/{host_name}` to adjust parameter per each host.

### Default configuration (`playbooks/group_vars/xitrum_servers`)


* Localtime
  `localtime` and `ntp_server` is used in datatime.task.
  Change these values to your local timezone.

  ```
  # Localtime
  localtime: /usr/share/zoneinfo/Japan
  ntp_server: ntp.nict.jp
  ```

* JDK
  Specified OracleJDK will be downloaded from oracle.com.

  ```
  # JDK
  jdk_version: jdk1.7.0_67
  jdk_rpm_dl_url: 'http://download.oracle.com/otn-pub/java/jdk/7u67-b01/jdk-7u67-linux-x64.rpm'
  jdk_rpm_file: /tmp/jdk-7u67-linux-x64.rpm
  ```

* OS user password
  OS user named `xitrum` will be created. Xitrum application will be executed by this user.
  You can change this user's password with following python command.

  ```
  # OS password of xitrum user
  # Create own password by
  # `python -c "from passlib.hash import sha512_crypt; print sha512_crypt.encrypt('<password>')"`
  # default password is `xitrum`
  user_password: $6$rounds=100000$Hv0imMoMCWSwsqZA$NYOMGPsGQImJ17F53nKsF8lHhD5ow9/FlH7/at18oxvIVCb/dvdzovkio21lH4d2aNx.tVVLhfV5qOZpMrmcb.
  ```

* run_clients
  If you will create http clients at target host(For example, run benchmark script),
  Set this value yes, Playbook will tune karnel parameter for massive clients.

  ```
  #If you want run massive client on server, set yes for tune Sysctl.
  #run_clients: yes
  ```

* Application
  Set your application repository and Boot file.
  Source code will be cloned into `app_home`.
  See also: [How to clone private repository](http://stackoverflow.com/questions/22755268/clone-a-private-repo-of-github-with-username-and-password)

  ```
  # Application
  app_git_repo: 'https://github.com/xitrum-framework/xitrum-new.git'
  app_home: /opt/src/app
  app_boot_file: quickstart.Boot
  ```

* SSL cert file and key
  Put files for ssl in `roles/xitrum/files` .These files are copied to server and also binded into xitrum.conf.

  ```
  # Put these files in role/xitrum/file
  ssl_crt:ssl_example.crt
  ssl_key:ssl_example.key
  ```

* For xitrum.conf
  Port numbers are also referenced by iptable(`role/sys/templates/iptables.j2`)
  Any other settings should be wirtten in (`role/xitrum/templates/xitrum.conf.j2`)

  ```
  # For xitrum.conf
  xitrum_conf:
    port:
      http: 8000
      https: 4430
    basicAuth:
      realm: xitrum
      username: xitrum
      password: xitrum
  ```

* For akka.conf
  Port numbers are also referenced by iptable(`role/sys/templates/iptables.j2`)

  ```
  # For akka.conf
  akka_conf:
    port: 2551
  ```

## Edit `playbooks/roles/xitrum/templates/xitrum.conf.j2`

It is used as template for xitrum.conf with binding {host|group}_vars params.
Original `config` directory cloned from git repository will removed.

This playbook create new `config` direcory inside app root.
So if you want tune your xitrum.conf and other config files,
{Edit|Create} your config {file|template} inside `roles/xitrum/{files|templates}`.

## Apply playbook with ansible

```
git clone git://github.com/ansible/ansible.git --recursive
cd ansible
source ./hacking/env-setup
cd ../
ansible-playbook playbook/init.yml -i hosts --private-key=/path/to/secret.pem -u root
```

This playbook do followings.

* Tune OS parameter for massive connection.
* Config OS datetime.
* Config iptables for xitrum application with port numbers described in `group_vars/xutrum`
* Install Oracle JDK.
* Create `xitrum` user.
* Delete current application if exist.
* Clone application from specified repository.
* Build application by `sbt/sbt xitrum-package`.
* Register application as a service named `xitrum`.
  This service is an alias to $APP_ROOT/target/xitrum/script/runner $BOOT_CLASS
* Start xitrum service (service is executed by xitrum user).

---
# Requirement

**You need easy_install, pip, etc, to use Ansible**

See also: [Ansible official document](http://docs.ansible.com/intro.html)
