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

