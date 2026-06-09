# Linux 系統自動化運維 — Class Notes Summary

**資工三 · 林小蓮 · 111210552 · National Quemoy University**

-----

## Week 01 — 24 February 2026

*Linux Environment, Vim Editor*

### Linux Environment (WSL2)

- WSL2 runs a **real Linux kernel** on Windows — no dual-boot or VM
- Distro: **Ubuntu 24.04.2 LTS**, Kernel: `5.15.167.4-microsoft-standard-WSL2`
- Check: `uname -a` | `uname -r` | `cat /etc/issue`

### Vim Editor — 3 Modes

|Mode       |Enter          |Key Actions                                                                         |
|-----------|---------------|------------------------------------------------------------------------------------|
|**Normal** |default        |`dd` delete line, `yy` copy, `p` paste, `u` undo, `gg`/`G` top/bottom               |
|**Insert** |`i` / `a` / `o`|Type freely; `Esc` to return Normal                                                 |
|**Command**|`:`            |`:w` save, `:q` quit, `:wq` save+quit, `:q!` force quit, `:%s/old/new/g` replace all|

-----

## Week 02 — 3 March 2026

*File Attributes, ACL, User Info, Exit Codes, User Management, Password Hashing*

### File Attributes — chattr & lsattr

```bash
lsattr file          # view attribute flags
chattr +i file       # immutable — no delete/modify (even root)
chattr -i file       # remove immutable flag
chattr +a file       # append-only
```

Flags: `+i` immutable, `+a` append-only, `+e` extents, `+s` secure delete

### ACL & Permissions

- `r`=4, `w`=2, `x`=1, `-`=0
- `chmod 755 file.sh` → owner=rwx, group=r-x, others=r-x

### User Info — /etc/passwd

```bash
cat /etc/passwd | grep 'benny'
id benny                        # show UID, GID, groups
getent passwd benny
cat /etc/passwd | grep "benny" > /dev/null && echo 1 || echo 0   # check if exists
```

Format: `username:x:UID:GID:comment:/home/user:/bin/sh`

### Exit Codes & Logic Operators

|Code|Meaning          |
|----|-----------------|
|0   |Success          |
|1   |General error    |
|127 |Command not found|
|126 |Permission denied|

Operators: `&&` AND (run if prev succeeded), `||` OR (run if prev failed), `|` pipe, `> /dev/null` suppress output

### User Management

```bash
whoami / who / id username
su username            # switch user
sudo su                # switch to root (keeps env)
sudo usermod -G sudo user   # add to sudo group
```

### Password Hashing — openssl

```bash
openssl passwd -1 '123456'                    # MD5 hash, random salt
openssl passwd -1 -salt 'abcdefg' '123456'   # MD5 hash, fixed salt
```

Format: `$1$` = MD5 | next segment = salt | last segment = hash

-----

## Week 03 — 10 March 2026

*sudo su, .bashrc, chown/find, Apache2, iptables, Single User Mode*

### sudo su vs sudo su -

|           |`sudo su`               |`sudo su -`          |
|-----------|------------------------|---------------------|
|Environment|Keeps current user’s env|Full root login shell|
|PATH       |Original user’s PATH    |Root’s full PATH     |
|Directory  |Unchanged               |Changes to `/root`   |

### .bashrc & source

```bash
vim .bashrc           # edit shell config
source .bashrc        # reload without restarting terminal
. .bashrc             # shorthand for source
```

### chown & find

```bash
chown user file
chown user:group file
chown -R tom.tom data              # recursive
find /path -user tom -exec mv {} /tmp \;
find / -name '*.log'
```

### Apache2 Setup

```bash
sudo apt update && sudo apt install apache2
sudo systemctl status/start/enable apache2
```

Visit `http://IP_ADDRESS` → Apache2 default page

### iptables Firewall

```bash
sudo iptables -F           # flush all rules
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -L           # list rules
```

Always flush first before setting new rules.

### Single User Mode — Reset Root Password

1. Reboot → press key to enter GRUB
1. Select Ubuntu → press `e` to edit
1. Find `linux` line → append `init=/bin/bash`
1. Boot with `Ctrl+X`
1. `passwd root` → enter new password
1. `exec /sbin/init` or `reboot`

-----

## Week 04 — 17 March 2026

*SSH, netstat, SSH Keys, OpenCode, Apache2 Deploy, MCP*

### SSH — Remote Access

```bash
ssh user@host
ssh -p 2222 user@host
ssh -i key.pem user@host
# Enable root login: edit /etc/ssh/sshd_config → PermitRootLogin yes
sudo systemctl restart sshd
```

### netstat — Check Open Ports

```bash
sudo netstat -tunlp    # all ports with process names
netstat -an            # all connections, no DNS lookup
```

Flags: `-t`=TCP, `-u`=UDP, `-n`=numeric, `-l`=listening, `-p`=PID

### SSH Key Authentication

```bash
ssh-keygen -t rsa -b 2048           # generate key pair
ssh-copy-id user@192.168.164.138    # copy public key to remote
ssh user@host                        # login without password
chmod 400 key.pem                    # fix permissions for PEM file
```

### OpenCode — AI Agent in Terminal

```bash
curl -fsSL https://opencode.ai/install | bash
opencode           # launch (restart terminal first)
/connect           # choose AI provider
/models            # switch models
```

### Apache2 — Deploy HTML

Web root: `/var/www/html/`

```bash
sudo chown user:user /var/www/html && sudo chmod 755 /var/www/html
sudo mv /tmp/time.htm /var/www/html/
# visit http://IP/time.htm
```

-----

## Week 05 — 24 March 2026

*apt tools, curl, ufw, MariaDB, SQL basics*

### Useful Tools

```bash
sudo apt install tree / net-tools
tree /home
sudo apt remove <package>
sudo apt list --installed | grep X
```

### curl & HTTP Status Codes

```bash
curl -o NUL -s -w "%{http_code}" http://IP    # print status code only
```

|Range|Meaning      |
|-----|-------------|
|1xx  |Informational|
|2xx  |Success      |
|3xx  |Redirection  |
|4xx  |Client Error |
|5xx  |Server Error |

### ufw — Uncomplicated Firewall

```bash
sudo ufw enable/disable
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo systemctl stop/disable ufw
```

### MariaDB Setup

```bash
sudo apt install mariadb-server
sudo systemctl status mariadb
sudo mysql -u root                              # local login
mysql -h 192.168.164.139 -uroot -p             # remote login
sudo netstat -tunlp | grep 3306                # check port
```

### MariaDB Remote Access

1. Edit `/etc/mysql/mariadb.conf.d/50-server.cnf` → `bind-address = 0.0.0.0`
1. Inside MariaDB:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### SQL Basics

```sql
show databases;
create database testdb;
use testdb;
create table addrbook(name varchar(50) not null, phone char(18));
insert into addrbook(name,phone) values ("tom","0912111222");
select * from addrbook;
select * from addrbook where name="mary";
```

-----

## Week 06 — 31 March 2026

*SQL UPDATE/DELETE, PHP, MariaDB+PHP, Apache htpasswd, UserDir, Virtual Host*

### SQL — UPDATE & DELETE

```sql
update addrbook set phone="0933123123" where name="john";
delete from addrbook where name="mary";   -- always use WHERE!
```

CRUD: `insert` → `select` → `update` → `delete`

### PHP Setup

```bash
sudo apt install php libapache2-mod-php
sudo systemctl restart apache2
# test: <?php phpinfo(); ?> at /var/www/html/test.php
```

### PHP + MariaDB

```php
$conn = mysqli_connect("localhost","root","","testdb");
$r = mysqli_query($conn,"select name,phone from addrbook");
while($row = mysqli_fetch_array($r)){ echo $row["name"]." ".$row["phone"]; }
```

### Apache htpasswd

```bash
sudo htpasswd -c /etc/apache2/.htpasswd mary      # -c = create new file
sudo htpasswd /etc/apache2/.htpasswd mary2        # add user (no -c!)
```

### Apache UserDir

```bash
sudo a2enmod userdir && sudo systemctl restart apache2
mkdir ~/public_html
echo 'My page' > ~/public_html/index.html
# visit http://server/~username/
```

### Apache Virtual Host

```bash
mkdir /var/www/www-a-com && echo "www.a.com" > /var/www/www-a-com/index.html
# Configure VirtualHost (DocumentRoot + ServerName) in Apache config
# Client hosts file: 192.168.164.139  www.a.com
```

-----

## Week 08 — 14 April 2026

*Redis, Redis+Python, GitHub Pages, VS Code, Claude Code, AnythingLLM*

### Redis — Install & Config

```bash
sudo apt install redis-server
sudo vim /etc/redis/redis.conf
# Change: bind 127.0.0.1 → bind 0.0.0.0  (for remote access)
sudo systemctl restart redis.service
sudo netstat -tunlp | grep redis    # verify port 6379
```

### Redis + Python (uv)

```bash
uv init test-redis --python=3.12 && cd test-redis && uv sync
source .venv/bin/activate
uv pip install redis mysql-connector-python
python app_session.py   # session demo: store session ID in Redis
python app_cache.py     # cache demo: Redis miss → MySQL → Redis hit
```

Pattern: check Redis first → miss → query MySQL → store in Redis

### GitHub Pages Clock

```bash
gh repo create clock-web --public --source=. --push
# Enable Pages: Settings → Pages → Deploy from branch → main
# Visit: username.github.io/clock-web/
```

### VS Code on Linux

```bash
sudo apt-get install wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/
# Add VS Code APT repo to /etc/apt/sources.list.d/vscode.sources
sudo apt install code
```

### Claude Code (VS Code + local ollama)

```bash
# .bashrc:
export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_BASE_URL=http://127.0.0.1:11434
source .bashrc
```

### AnythingLLM — Local AI

- Install: `docs.anythingllm.com` → AppImage
- Settings → LLM Provider → ollama → pick model
- RAG: upload docs → AI answers from your files only, no internet

### Redis vs MySQL

|           |Redis                    |MySQL                       |
|-----------|-------------------------|----------------------------|
|Speed      |In-memory, microseconds  |Disk-based, milliseconds    |
|Data model |Key-value                |Tables, relations           |
|Use case   |Session, cache, queues   |User data, permanent records|
|Persistence|Lost on restart (default)|ACID transactions           |

-----

## Week 11 — 5 May 2026

*grep, sed, for loop, Docker, HAProxy, CVE exploits*

### grep — Search Flags

```bash
grep -c aaa file          # count matching lines
grep -e aaa -e bbb file   # match aaa OR bbb
grep -i -n aaa file       # case-insensitive, show line numbers
grep -A 2 pattern file    # 2 lines AFTER match
grep -B 2 pattern file    # 2 lines BEFORE match
cat auth.log | grep 'Invalid user' | awk '{print $6}' | sort | uniq   # SSH attacker names
```

### sed — Edit Lines

```bash
sed '/Linux/a newline' file     # append AFTER matching line
sed '/Linux/i newline' file     # insert BEFORE matching line
sed '1d' file                   # delete line 1
sed '1,3d' file                 # delete lines 1-3
sed '/Linux/d' file             # delete all matching lines
sed -i '/^$/d' file             # delete blank lines (write to file)
```

### sed — Substitute

```bash
sed 's#Linux#Microsoft#g' file       # replace ALL per line
sed '1,4s#Linux#Microsoft#' file     # replace only lines 1-4
sed 's/a/A/3g' file                  # from 3rd occurrence onwards
sed -e '2,3s/^/#/' file              # add # prefix to lines 2-3
sed -i '/^$/d' file                  # in-place: delete blank lines
sed -n '2,~4p' file                  # print only lines 2-4
```

### for Loop + Docker

```bash
for i in {1..5}; do echo $i; done
for filename in $(ls /tmp); do echo $filename; done

sudo docker run -d --name web1 -p 8080:80 httpd:latest
docker rm -f $(docker ps -a -q)    # remove ALL containers

for i in {1..5}; do
  docker run -d --name web$i -p 808$i:80 httpd:latest
done
```

### CVE Exploits (Educational)

- **Copy Fail (CVE-2026-31431)**: AF_ALG logic flaw → root LPE
- **dirtyfrag**: kernel page cache fragmentation → root LPE
- **Defense**: `sudo apt upgrade` → keep kernel updated

-----

## Week 12 — 12 May 2026

*sed advanced, awk, SSH Key Auth, Hydra, Fail2ban, dirtyfrag*

### awk — Fields & Patterns

```bash
awk '{print $1,$4}' file              # print fields 1 and 4
awk -F, '{print $1,$4}' file          # comma separator
awk '{print $NF}' file                # last field
awk '/^This/ {print $2}' file         # filter lines starting with 'This'
awk '$0 !~ /5000/ {print $1,$3}' file # exclude lines with 5000
awk '{count[$3]++} END{for(w in count) print w,count[w]}' file  # frequency map
```

### awk Log Analysis

```bash
# SSH attackers:
grep 'Invalid user' auth.log | awk '{print $6}' | sort | uniq
# HTTP requests per IP:
awk '{count[$1]++} END{for(w in count) print w,count[w]}' access.log
```

Log fields: `auth.log` → $6=username | `access.log` → $1=IP, $7=path, $9=status

### SSH — Disable Password Login

```bash
# /etc/ssh/sshd_config:
PasswordAuthentication no
PermitRootLogin yes
X11Forwarding yes
systemctl daemon-reload && systemctl restart ssh
```

### Hydra — SSH Brute Force (Educational)

```bash
sudo apt install hydra -y
hydra -L user.txt -P password.txt ssh://192.168.164.139 -t 4
# -L=user list, -P=password list, -t=threads
```

### Fail2ban — Block Brute Force

```bash
sudo apt install fail2ban
# /etc/fail2ban/jail.local:
# [sshd] enabled=true, maxretry=3, findtime=600, bantime=1200
systemctl start fail2ban
```

-----

## Week 13 — 19 May 2026

*awk advanced, SSH hardening, Ansible setup, Ansible ping*

### awk Advanced

```bash
awk '{print $NF}' file                              # last field (adaptive)
awk '$0 !~ /5000/ {print $1,$3}' file               # negate match
awk '{count[$3]++} END{for(w in count) print w,count[w]}' file
awk 'BEGIN{c=0} /^t/ {c++} END{print c}' file       # count lines starting with t
```

### Ansible — 3 Machine Setup

```bash
# On each machine:
hostnamectl set-hostname controller/web/db
# /etc/hosts: 192.168.164.139 controller  .135 web  .136 db

# On controller (as root):
ssh-keygen
ssh-copy-id root@web && ssh-copy-id root@db
sudo apt install ansible
```

### Ansible Inventory & ping

```ini
# /etc/ansible/hosts
[server1]
web
[server2]
db
[servers]
web
db
```

```bash
ansible server1 -m ping    # ping web → SUCCESS
ansible servers -m ping    # ping both → SUCCESS
```

-----

## Week 14 — 26 May 2026

*ARP Spoofing, ARP Defense, Ansible ad hoc, Ansible modules*

### ARP Spoofing — MITM Attack

```bash
sudo apt install dsniff wireshark
arpspoof -t <victim_IP> <gateway_IP>   # tell victim that attacker = gateway
arpspoof -t <gateway_IP> <victim_IP>   # tell gateway that attacker = victim
echo 1 > /proc/sys/net/ipv4/ip_forward  # enable IP forwarding on attacker
```

### ARP Defense

```bash
sudo arp -s 192.168.1.100 00:11:22:33:44:55    # static ARP entry (prevents poisoning)
arp -a / arp -n                                  # check ARP table
```

### Ansible — Ad Hoc vs Playbook

```bash
ansible server1 -m command -a 'pwd'              # command module (no pipes)
ansible server1 -m shell -a 'ifconfig | grep ens33'   # shell module (supports pipes)
ansible server1 -a 'pwd'                         # command is default module
```

- **command**: simple commands only, no `|` `;` `&&`
- **shell**: full bash features
- **script**: upload local script → run on remote

### Ansible Modules

```bash
# apt — package management
ansible server1 -m apt -a 'name=apache2 state=present'
ansible server1 -m apt -a 'name=apache2 state=absent'
ansible server1 -m apt -a 'name=apache2,git,wget state=present'

# copy & fetch
ansible server1 -m copy -a 'src=/root/1.txt dest=/tmp/1.txt backup=yes mode=664'
ansible server1 -m fetch -a 'src=/tmp/2.txt dest=/root'

# dpkg verify
dpkg -l | grep apache2
sudo apt remove apache2    # remove only (keep config)
sudo apt purge apache2     # remove + config
```

-----

## Week 15 — 2 June 2026

*Ansible Playbook, Handler, Variables, register/debug*

### Ansible Playbook — Apache2 Setup

```yaml
- hosts: server1
  remote_user: root
  tasks:
    - name: create file
      file: name=/tmp/newfile state=touch
    - name: create user
      user: name=test2 system=yes shell=/sbin/nologin
    - name: install apache2
      apt: name=apache2 state=present
    - name: copy config
      copy: src=/root/ports.conf dest=/etc/apache2/ports.conf
    - name: start service
      service: name=apache2 state=started
```

```bash
ansible-playbook setup_httpd.yml
```

### Handler & notify

```yaml
    - name: copy configuration file
      copy: src=/root/ports.conf dest=/etc/apache2/ports.conf
      notify: restart apache2   # only fires if task CHANGED

  handlers:
    - name: restart apache2
      service: name=apache2 state=restarted   # state=restarted = stop + start
```

`notify` → handler runs only when task status = **changed**

### Variables — vars & vars_files

```yaml
# Inline vars:
vars:
  pkgname1: vsftpd
  pkgname2: joe
tasks:
  - apt:
      name: ["{{ pkgname1 }}", "{{ pkgname2 }}"]
      state: present

# External vars_files:
vars_files:
  - ./vars_public.yml
# vars_public.yml: app1: openssh-server, app2: apache2
```

> ⚠️ Always quote `{{ }}` when it appears at the start of a value

### register & debug

```yaml
- name: check apache2 status
  shell: systemctl status apache2
  register: check_httpd
  ignore_errors: true           # continue even if apache2 not running

- name: show result
  debug:
    msg: "{{ check_httpd.stdout_lines }}"
```

Register variable attributes: `.stdout` `.stdout_lines` `.rc` `.stderr` `.failed` `.changed`

### Full Playbook + RECAP

```bash
ansible-playbook setup_httpd.yml
# PLAY RECAP: ok=7  changed=3  unreachable=0  failed=0
# ok = all tasks ran | changed = tasks that actually modified something | failed=0 = success
```

-----

*End of Notes — 林小蓮 · 111210552 · 資工三*