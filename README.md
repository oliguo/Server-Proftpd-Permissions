# Proftpd User Manual on Ubuntu 18.04

## Installation

```
apt-get install proftpd proftpd-basic
```

## Knowledge

### Group , User

```
Before create the group and user, I would like to have the limitation below to fulfill some cases:
```

| Group | User | Folder                   | Read | Write |
| ----- | ---- | ------------------------ | ---- | ----- |
| A     | a1   | /var/ftp/group_A/user_a1 | Y    | Y     |
| A     | a1   | /var/ftp/group_A/user_a2 | Y    | Y     |
| A     | a1   | /var/ftp/group_A/user_a3 | Y    |       |
| A     | a1   | /var/ftp/group_A/user_b1 | Y    | Y     |
| A     | a1   | /var/ftp/group_A/user_b2 | Y    |       |
| A     | a2   | /var/ftp/group_A/user_a2 | Y    | Y     |
| A     | a3   | /var/ftp/group_A/user_a3 | Y    | Y     |
| B     | b1   | /var/ftp/group_B/user_b1 | Y    | Y     |
| B     | b1   | /var/ftp/group_B/user_a1 | Y    |       |
| B     | b1   | /var/ftp/group_B/user_a2 | Y    |       |
| B     | b1   | /var/ftp/group_B/user_a3 | Y    |       |
| B     | b2   | /var/ftp/group_B/user_b2 | Y    | Y     |

### Command, Permission

```
Refer to:
Limit: http://www.proftpd.org/docs/howto/Limit.html
FTP CMD: http://www.proftpd.org/docs/howto/FTP.html
```

## Start !

### Create Group

```
mkdir -pv /var/ftp/group_A /var/ftp/group_B
groupadd group_A && groupadd group_B
```

### Create User

```
useradd -m -d /var/ftp/group_A/user_a1 -g group_A -s /sbin/nologin user_a1
useradd -m -d /var/ftp/group_A/user_a2 -g group_A -s /sbin/nologin user_a2
useradd -m -d /var/ftp/group_A/user_a3 -g group_A -s /sbin/nologin user_a3
useradd -m -d /var/ftp/group_B/user_b1 -g group_B -s /sbin/nologin user_b1
useradd -m -d /var/ftp/group_B/user_b2 -g group_B -s /sbin/nologin user_b2

chown -Rv user_a1:group_A /var/ftp/group_A/user_a1
chown -Rv user_a2:group_A /var/ftp/group_A/user_a2
chown -Rv user_a3:group_A /var/ftp/group_A/user_a3
chown -Rv user_b1:group_B /var/ftp/group_B/user_b1
chown -Rv user_b2:group_B /var/ftp/group_B/user_b2

chmod -Rv 775 /var/ftp
```

### Folder structure

```
|-- group_A
|   |-- user_a1
|   |-- user_a2
|   `-- user_a3
|-- group_B
|   |-- user_b1
|   `-- user_b2
```

### Set users' password by 'ftpasswd'

```
cat /etc/passwd | grep 'user_'

user_a1:x:1000:1000::/var/ftp/group_A/user_a1:/sbin/nologin
user_b1:x:1001:1000::/var/ftp/group_B/user_b1:/sbin/nologin
user_b2:x:1002:1000::/var/ftp/group_B/user_b2:/sbin/nologin
user_a2:x:1003:1001::/var/ftp/group_A/user_a2:/sbin/nologin
user_a3:x:1004:1001::/var/ftp/group_A/user_a3:/sbin/nologin
```

```
mkdir -pv /usr/local/proftpd

ftpasswd  --passwd --file=/usr/local/proftpd/ftpd.passwd --name=user_a1  --uid=1000 --gid=1000  --home=/var/ftp/group_A/user_a1  --shell=/sbin/nologin
ftpasswd  --passwd --file=/usr/local/proftpd/ftpd.passwd --name=user_a2  --uid=1001 --gid=1000 --home=/var/ftp/group_A/user_a2  --shell=/sbin/nologin
ftpasswd  --passwd --file=/usr/local/proftpd/ftpd.passwd --name=user_a3  --uid=1002 --gid=1000 --home=/var/ftp/group_A/user_a3  --shell=/sbin/nologin
ftpasswd  --passwd --file=/usr/local/proftpd/ftpd.passwd --name=user_b1  --uid=1003 --gid=1001 --home=/var/ftp/group_B/user_b1  --shell=/sbin/nologin
ftpasswd  --passwd --file=/usr/local/proftpd/ftpd.passwd --name=user_b2  --uid=1004 --gid=1001 --home=/var/ftp/group_B/user_b2  --shell=/sbin/nologin
```

```
cat /usr/local/proftpd/ftpd.passwd

user_a1:$1...:1000:1000::/var/ftp/group_A/user_a1:/sbin/nologin
user_a2:$1...:1001:1000::/var/ftp/group_A/user_a2:/sbin/nologin
user_a3:$1...:1002:1000::/var/ftp/group_A/user_a3:/sbin/nologin
user_b1:$1...:1003:1001::/var/ftp/group_B/user_b1:/sbin/nologin
user_b2:$1...:1004:1001::/var/ftp/group_B/user_b2:/sbin/nologin
```

### Change the settings of proftpd.conf

```
cp /etc/proftpd/proftpd.conf /etc/proftpd/proftpd.conf.bk
```

Add settings below with new file on the /etc/proftpd/conf.d/

```
vim /etc/proftpd/conf.d/settings.conf

PassivePorts 51100 51200
RequireValidShell off
AuthUserFile /usr/local/proftpd/ftpd.passwd

<Directory "/var/ftp/group_A/user_a1">
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        DenyAll
    </Limit>
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        AllowUser user_a1
    </Limit>
    <Limit CWD RETR READ>
        AllowUser user_b1
    </Limit>
</Directory>

<Directory "/var/ftp/group_A/user_a2">
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        DenyAll
    </Limit>
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        AllowUser user_a1
        AllowUser user_a2
    </Limit>
    <Limit CWD RETR READ>
        AllowUser user_b1
    </Limit>
</Directory>

<Directory "/var/ftp/group_A/user_a3">
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        DenyAll
    </Limit>
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        AllowUser user_a3
    </Limit>
    <Limit CWD RETR READ>
        AllowUser user_a1
        AllowUser user_b1
    </Limit>
</Directory>

<Directory "/var/ftp/group_B/user_b1">
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        DenyAll
    </Limit>
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        AllowUser user_b1
        AllowUser user_a1
    </Limit>
</Directory>

<Directory "/var/ftp/group_B/user_b2">
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        DenyAll
    </Limit>
    <Limit CWD MKD RNFR READ WRITE STOR RETR>
        AllowUser user_b2
    </Limit>
    <Limit CWD RETR READ>
        AllowUser user_a1
    </Limit>
</Directory>
```

Check config whether right or not

```
proftpd -t46
```

### Restart

```
service proftpd restart
```
