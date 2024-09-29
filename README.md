# PAM
Создаём пользователя otusadm и otus:
```
sudo useradd otusadm && sudo useradd otus
```
Создаём пользователям пароли:

![image](https://github.com/user-attachments/assets/c382542e-57c7-4e29-89b9-08ba84777946)


Создаём группу admin:``` sudo groupadd -f admin ```

Добавляем пользователей vagrant,root и otusadm в группу admin:
```
usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```

Проверяем возможность подключение по ssh

![image](https://github.com/user-attachments/assets/4d7e622d-39ab-4e68-8fde-f172ce9f4c70)


Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
```cat /etc/group | grep admin```

![image](https://github.com/user-attachments/assets/1977240a-1899-4a48-a28a-e90178896135)


Создадим файл-скрипт /usr/local/bin/login.sh

```
#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
```

Добавим права на исполнение файла: 
```
chmod +x /usr/local/bin/login.sh
```

Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:


```
auth       substack     password-auth
auth       include      postlogin
auth required pam_exec.so debug /usr/local/bin/login.sh
account    required     dad
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin
```

Проверяем работу скрипта:

![image](https://github.com/user-attachments/assets/ffc0354a-bf90-438b-96e2-3c7d78465fba)

Доступ для пользователя "otus" запрещен в выходные.
