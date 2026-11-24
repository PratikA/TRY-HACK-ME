# Bounty Hacker — TryHackMe Lab Write-up

**Lab date:** November 24, 2026  
**Operating system observed:** Ubuntu 20.04.6 LTS  
**Outcome:** Recovered both challenge flags and confirmed root privileges.

## 1. Check connectivity from Kali

Replace `LAB_IP` with the address shown for your room instance.

```bash
ping -c 16 LAB_IP
```

The lab VM responded to 6 of 16 ICMP requests. That was 62.5% packet loss, with an average round-trip time of 216.9 ms among the replies. The subsequent TCP scan completed despite the packet loss.

## 2. Record the services found

The TCP scan reported three open ports.

| Port | Service | Version reported by Nmap |
| --- | --- | --- |
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 |
| 80/tcp | HTTP | Apache 2.4.41 (Ubuntu) |

Nmap also reported 967 filtered ports and 30 closed ports. It received no responses on the filtered ports and TCP resets on the closed ports. The scan reported no HTML title for the HTTP service.

## 3. Review the anonymous FTP finding

The FTP server accepted anonymous authentication with reply code `230`. Nmap's passive-mode directory listing received `550 Permission denied`. The interactive FTP client then obtained a listing using an active-mode data connection and successfully downloaded two files.

| File | Observation |
| --- | --- |
| `task.txt` | A short task note signed `lin`. |
| `locks.txt` | A list of 26 password-like entries. |

The passive-mode failure and the successful active-mode transfer show why an error on one FTP data connection does not establish that the files are protected.

If your lab copies are already in `~/Downloads`, run these commands from Kali to read the note and count lines in the list without printing the entries.

```bash
cd ~/Downloads
cat task.txt
wc -l locks.txt
```

## 4. Verify the account in the lab session

The lab transcript shows that one of the exposed entries matched the SSH password for the local account. A subsequent interactive SSH login succeeded, confirming the credential finding. From an existing lab shell, run:

```bash
id
pwd
```

`id` reports your account and groups. In the transcript, the initial session was logged in as `lin`, and its working directory was the account's Desktop.

## 5. Review the lab account’s sudo permissions

Run this from the lab account's shell:

```bash
sudo -l
```

The transcript shows this rule:

```text
(root) /bin/tar
```

The sudo policy allowed the lab account to run the general-purpose archive utility `tar` with root privileges. The recorded session demonstrated that the permission allowed execution beyond the intended archive operation. The session then returned `uid=0(root) gid=0(root) groups=0(root)`, confirming the privilege boundary had been crossed. To check the privilege level of a shell you already have, use `id` again. UID 0 identifies the root account.

## 6. Review the challenge results

| Checkpoint | Result |
| --- | --- |
| User flag | Retrieved from the user's Desktop. |
| Root flag | Retrieved from the root account's home directory. |

In the recorded session, `pwd` showed `/home/lin/Desktop` and `ls` showed `user.txt`. From that directory, the user flag could be read with `cat user.txt`.

<details>
<summary>Flag values (spoilers)</summary>

| Flag | Value |
| --- | --- |
| User | `THM{CR1M3_SyNd1C4T3}` |
| Root | `THM{80UN7Y_h4cK3r}` |

</details>

## Defensive takeaways

- Disable anonymous FTP when public file access is unnecessary. Audit the contents of any directory that is intentionally exposed.
- Remove password lists from shared storage and rotate credentials that may have been exposed. Prefer key-based SSH authentication where appropriate.
- Review sudo rules for general-purpose utilities. If a privileged operation is necessary, grant access only to the specific operation with controlled arguments.
