# Linux Cheat Sheet — DevOps / SysAdmin Level

---

## 1. File System Basics

```bash
pwd                     # current directory
ls -la                  # list all, long format
cd /path                # change dir
cd -                    # go to previous dir
mkdir -p a/b/c          # create nested dirs
rmdir dir                # remove empty dir
rm -rf dir               # force remove dir + contents
cp -r src dst            # copy recursive
mv src dst                # move/rename
touch file                # create empty file / update timestamp
find /path -name "*.log"        # find by name
find /path -mtime -1            # modified in last 1 day
find /path -size +100M          # files > 100MB
find /path -type f -exec rm {} \;   # find + delete
locate filename           # fast search (needs updatedb)
tree -L 2                 # dir tree, 2 levels
```

**Filesystem hierarchy (know these for interviews):**
- `/etc` — configs
- `/var/log` — logs
- `/var/www` — web files
- `/opt` — optional/3rd-party software
- `/usr/local/bin` — custom binaries
- `/tmp` — temp (cleared on reboot)
- `/proc`, `/sys` — kernel/process virtual filesystems
- `/home` — user dirs
- `/root` — root user home
- `/etc/fstab` — mount config
- `/boot` — kernel + bootloader

---

## 2. Permissions & Ownership

```bash
chmod 755 file            # rwxr-xr-x
chmod +x script.sh        # add execute
chmod -R 644 dir           # recursive
chown user:group file      # change owner+group
chown -R user:group dir
umask                       # default permission mask
```

**Permission digits:** read=4, write=2, execute=1 → sum per (owner/group/other)

**Special bits (DevOps-relevant):**
```bash
chmod u+s file      # SUID — run as file owner
chmod g+s dir        # SGID — new files inherit group
chmod +t dir         # sticky bit — only owner can delete own files (used on /tmp)
```

**ACLs (fine-grained perms beyond owner/group/other):**
```bash
getfacl file
setfacl -m u:username:rwx file
```

---

## 3. Users & Groups

```bash
whoami
id                          # uid, gid, groups
sudo useradd -m -s /bin/bash username
sudo passwd username
sudo usermod -aG groupname username   # add to group
sudo userdel -r username               # delete user + home
sudo groupadd groupname
cat /etc/passwd             # user list
cat /etc/group              # group list
sudo visudo                 # edit sudoers safely
su - username               # switch user
sudo -i                     # root shell
sudo -u username cmd        # run cmd as another user
```

---

## 4. Process Management

```bash
ps aux                      # all processes
ps -ef                      # alt format
ps aux --sort=-%mem | head  # top memory consumers
top / htop                   # live process monitor
pgrep -f "process_name"
pkill -f "process_name"
kill -9 PID                  # force kill
kill -15 PID                 # graceful kill (SIGTERM)
nice -n 10 command            # start with lower priority
renice -n 5 -p PID             # change priority of running proc
jobs                            # background jobs in shell
bg / fg                         # background/foreground job
nohup command &                 # run immune to hangup
disown                          # detach job from shell
```

**Process states:** R (running), S (sleeping), D (uninterruptible sleep), Z (zombie), T (stopped)

---

## 5. Systemd / Services (Critical for DevOps)

```bash
systemctl status nginx
systemctl start|stop|restart|reload nginx
systemctl enable|disable nginx          # start on boot or not
systemctl daemon-reload                  # reload unit files after edit
systemctl is-active nginx
systemctl list-units --type=service --state=running
journalctl -u nginx                      # logs for a service
journalctl -u nginx -f                   # follow live
journalctl --since "1 hour ago"
journalctl -p err                        # only errors
journalctl --disk-usage
```

**Custom systemd service** (`/etc/systemd/system/myapp.service`):
```ini
[Unit]
Description=My App
After=network.target

[Service]
ExecStart=/usr/bin/node /opt/myapp/index.js
Restart=always
User=appuser
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

---

## 6. Networking

```bash
ip a                        # show interfaces/IPs (modern)
ifconfig                    # legacy
ip route                    # routing table
ping host
traceroute host / mtr host
curl -I url                 # headers only
curl -v url                 # verbose (debug requests)
curl -X POST -d '{"a":1}' -H "Content-Type: application/json" url
wget url
nslookup domain / dig domain
netstat -tulpn               # legacy: listening ports + process
ss -tulpn                    # modern replacement for netstat
lsof -i :8080                 # what's using port 8080
lsof -i -P -n | grep LISTEN
telnet host port              # test port connectivity
nc -zv host port              # netcat port check
hostname -I                    # local IP
cat /etc/hosts                 # static DNS entries
```

**Firewall:**
```bash
# ufw (Ubuntu)
ufw status
ufw allow 22/tcp
ufw enable

# iptables (raw)
iptables -L -n -v
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# firewalld (RHEL/CentOS)
firewall-cmd --list-all
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

---

## 7. Disk & Storage

```bash
df -h                        # disk space (human readable)
du -sh *                     # size of each item in current dir
du -sh /var/log/*  | sort -h # sorted by size
mount / umount                # mount/unmount
lsblk                          # list block devices
fdisk -l                       # partition info
mkfs.ext4 /dev/sdb1              # format partition
blkid                            # UUIDs of devices
cat /etc/fstab                    # persistent mounts
free -h                           # RAM usage
vmstat 1                          # virtual memory stats every 1s
iostat -x 1                       # disk I/O stats
```

---

## 8. Text Processing (Heavily used in scripting/log parsing)

```bash
cat file
less file                       # scroll view (q to quit)
head -n 20 file
tail -n 20 file
tail -f file                    # follow live (logs!)
tail -F file                    # follow + handle log rotation
grep "pattern" file
grep -r "pattern" dir/          # recursive
grep -i "pattern" file          # case-insensitive
grep -v "pattern" file          # invert match
grep -c "pattern" file          # count matches
grep -E "regex" file            # extended regex
awk '{print $1}' file           # print column 1
awk -F',' '{print $2}' file     # custom delimiter
awk '{sum+=$1} END {print sum}' file
sed 's/old/new/g' file          # replace text
sed -i 's/old/new/g' file       # in-place edit
sort file | uniq -c | sort -nr  # frequency count (classic log analysis combo)
cut -d',' -f1,3 file            # extract columns
wc -l file                      # line count
tr 'a-z' 'A-Z' < file           # translate chars
diff file1 file2
xargs                            # build/run commands from stdin
```

**Log analysis one-liners:**
```bash
grep "ERROR" app.log | wc -l
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10   # top IPs
tail -f app.log | grep --line-buffered "ERROR"
```

---

## 9. Package Management

```bash
# Debian/Ubuntu
apt update && apt upgrade
apt install pkg
apt remove pkg
apt search pkg
dpkg -l                        # list installed

# RHEL/CentOS/Fedora
yum install pkg / dnf install pkg
yum remove pkg
rpm -qa                        # list installed

# Universal
snap install pkg
```

---

## 10. Archiving & Compression

```bash
tar -czvf archive.tar.gz dir/     # compress
tar -xzvf archive.tar.gz          # extract
tar -tzvf archive.tar.gz          # list contents without extracting
zip -r archive.zip dir/
unzip archive.zip
gzip file / gunzip file.gz
```

---

## 11. Environment Variables & Shell Config

```bash
env                              # list all env vars
export VAR=value                 # set for session
echo $PATH
echo $HOME
unset VAR
printenv VAR
```

**Config files (order matters):**
- `~/.bashrc` — interactive non-login shell
- `~/.bash_profile` / `~/.profile` — login shell
- `/etc/environment` — system-wide vars
- `/etc/profile` — system-wide login shell

---

## 12. Cron & Scheduled Tasks

```bash
crontab -e                   # edit user's crontab
crontab -l                   # list
crontab -r                   # remove all

# Format: min hour day month weekday command
0 2 * * *   /path/backup.sh          # daily at 2am
*/5 * * * * /path/healthcheck.sh     # every 5 min
0 0 * * 0   /path/weekly.sh          # weekly on Sunday

# System-wide cron dirs
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
```

`at` for one-time scheduled jobs:
```bash
echo "command" | at 10:00 PM
```

---

## 13. SSH & Remote Access

```bash
ssh user@host
ssh -i key.pem user@host
ssh -p 2222 user@host
ssh-keygen -t ed25519 -C "email"     # generate key pair
ssh-copy-id user@host                 # push public key to server
scp file user@host:/path              # copy file to remote
scp -r dir user@host:/path            # copy dir
rsync -avz src/ user@host:/dst/       # sync (better than scp, resumable)
rsync -avz --delete src/ dst/         # mirror, delete extras

# ~/.ssh/config for shortcuts
Host myserver
    HostName 1.2.3.4
    User ubuntu
    IdentityFile ~/.ssh/mykey.pem
    Port 22
```
```bash
ssh myserver          # now just this works
```

---

## 14. Shell Scripting Essentials

```bash
#!/bin/bash
set -euo pipefail       # exit on error, undefined var, pipe fail — ALWAYS use this

VAR="value"
if [ "$VAR" == "value" ]; then
  echo "match"
elif [ -f "/path/file" ]; then
  echo "file exists"
fi

for i in {1..5}; do echo $i; done
for f in *.log; do echo $f; done

while read -r line; do
  echo "$line"
done < file.txt

function myfunc() {
  local x=$1
  echo "arg: $x"
}
myfunc "hello"

# arithmetic
count=$((count + 1))

# command substitution
DATE=$(date +%F)

# check exit code
if [ $? -eq 0 ]; then echo "success"; fi

# check if command exists
command -v docker >/dev/null 2>&1 || echo "not installed"
```

**Test flags:** `-f` file exists, `-d` dir exists, `-z` string empty, `-n` string not empty, `-eq/-ne/-gt/-lt` numeric compare

---

## 15. Logs (know where things live)

```bash
/var/log/syslog          # Debian/Ubuntu system log
/var/log/messages        # RHEL/CentOS system log
/var/log/auth.log        # auth/ssh attempts (Debian)
/var/log/secure          # auth log (RHEL)
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/dmesg           # kernel ring buffer
dmesg -T | tail          # human readable timestamps
journalctl -k            # kernel logs via systemd
logrotate -d /etc/logrotate.conf   # dry-run check log rotation
```

---

## 16. Performance & Debugging

```bash
top / htop
uptime                       # load average
vmstat 1 5
iostat -x 1
sar -u 1 3                    # CPU usage history (needs sysstat)
strace -p PID                 # trace syscalls of running process
strace command                # trace syscalls of new process
ltrace command                # trace library calls
lsof                            # list open files
lsof -p PID                     # files opened by a process
free -h
uname -a                        # kernel/system info
lscpu                           # CPU info
w                                # who's logged in + load
last                             # login history
dmesg | grep -i error
```

**Load average interpretation:** compare to number of CPU cores (`nproc`). Load = cores → fully utilized.

---

## 17. Security Basics

```bash
sudo apt install fail2ban        # brute-force protection
fail2ban-client status sshd
chage -l username                 # password expiry info
lastb                              # failed login attempts
ss -tulpn                          # audit open ports
sudo find / -perm -4000 -type f    # find SUID binaries (security audit)
sudo lynis audit system            # security audit tool
openssl x509 -in cert.pem -text -noout   # inspect cert
openssl s_client -connect host:443       # test TLS handshake
```

---

## 18. Docker-adjacent Linux (since you work with containers)

```bash
docker ps
docker ps -a
docker logs -f container_id
docker exec -it container_id bash
docker inspect container_id
docker stats                     # live resource usage per container
docker system df                 # disk usage by docker
docker system prune -a           # cleanup unused images/containers
cat /proc/1/cgroup               # check cgroup (namespaces/isolation concepts)
```

**Namespaces & cgroups** are the Linux kernel features that make containers possible — worth knowing conceptually for interviews: namespaces isolate (PID, net, mount, etc.), cgroups limit resources (CPU, memory).

---

## 19. Symlinks & Links

```bash
ln -s /path/target linkname     # symbolic link
ln /path/target linkname        # hard link
readlink -f linkname            # resolve full path
```

---

## 20. Quick Reference — Common DevOps Troubleshooting Flow

| Symptom | Check |
|---|---|
| Service down | `systemctl status`, `journalctl -u` |
| High CPU | `top`, `htop`, `ps aux --sort=-%cpu` |
| High memory | `free -h`, `ps aux --sort=-%mem` |
| Disk full | `df -h`, `du -sh /* \| sort -h` |
| Port not reachable | `ss -tulpn`, `firewall`, `nc -zv` |
| App won't start | `journalctl -u app -f`, check `.service` file |
| Slow disk | `iostat -x 1`, `vmstat 1` |
| Network issue | `ping`, `traceroute`, `dig`, `ip route` |
| Permission denied | `ls -la`, `id`, `chmod/chown` |
| Log flooding | `tail -f`, `logrotate` |

---

## 21. Must-Know Keyboard/Shell Shortcuts

```
Ctrl+R      reverse search history
Ctrl+C      kill current command
Ctrl+Z      suspend to background
Ctrl+D      exit shell / EOF
!!          repeat last command
!$          last argument of previous command
history     show command history
```

---

### Suggested revision order
1. File system + permissions + users → 2. Process management + systemd → 3. Networking → 4. Text processing (grep/awk/sed) → 5. Shell scripting → 6. Disk/storage + performance tools → 7. Security + logs → 8. Cron/SSH → 9. Tie it together with Docker/cgroups concepts.
