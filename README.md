# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
   ssh youruser@remotehost
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran
# 2) A detailed, step-by-step explanation of what happened at each stage
```
# SSH Connection to Remote Server – Step-by-Step Explanation

## 1. SSH Command Used

```bash
ssh ubuntu@chat.erarslan.tk
```

I used this command from my local Linux terminal (WSL in my case). I had already added my public SSH key to the remote server, so I was not asked for a password.

---

## 2. What Happened – Step by Step

### Step 1: TCP Connection to Port 22

After I ran the command, my computer opened a **TCP connection** to the server `chat.erarslan.tk` on **port 22**, which is the default port for SSH.

This is done using the **TCP 3-way handshake** (SYN, SYN-ACK, ACK), establishing a reliable connection between my local machine and the SSH server.

---

### Step 2: SSH Protocol Handshake

Once the TCP connection was established, the **SSH protocol handshake** started.

At this stage:
- Both client and server agreed on which **encryption algorithms**, **key exchange method**, and **compression algorithms** to use.
- The server sent me its **public host key**.
- My SSH client checked this key against the list of known hosts to verify the server's identity.
- Then, a **session key** was generated using a key exchange method like **ECDH** or **Curve25519**, so all further communication would be encrypted.

---

### Step 3: Authentication (Public Key)

Because I had previously uploaded my **public key** to the server (`~/.ssh/authorized_keys`), the authentication used the **public key method**:

- The server sent a challenge.
- My client signed it with my **private key**.
- The server verified it with my public key.
- As a result, I was logged in without entering a password.

---

### Step 4: Shell Allocation

Once authenticated, the server gave me access to an **interactive shell session** (in this case, Bash). I was now logged in as the `ubuntu` user on the remote machine.

The prompt changed to something like:

```bash
ubuntu@home-server:~$
```

I could now run commands directly on the remote server.

---

### Step 5: Exit the Session

After I finished my tasks, I exited the remote shell with:

```bash
exit
```

This closed the SSH session and ended the TCP connection cleanly.

---
---

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Why Ed25519 is preferred (performance, security).

**Provide:**

```bash
# 1) The ssh-keygen command you ran
# 2) The file paths of the generated keys
# 3) Your written explanation (3–5 sentences) of the signature process
```
# SSH Key Generation and Signature Process Explanation

## 1. SSH Key Generation Command Used

```bash
ssh-keygen -t ed25519 -C "mikailerarslan@chat.erarslan.tk"
```

I ran this command from my local terminal to generate an Ed25519 SSH key pair with my email as a comment.

---

## 2. File Paths of the Generated Keys

By accepting the default location, the key pair was saved to:

- **Private key**: `C:\Users\Mikail Erarslan/.ssh/id_ed25519`
- **Public key**: `C:\Users\Mikail Erarslan/.ssh/id_ed25519.pub`

---

## 3. Signature Process – Written Explanation

- The **private key** remains on my local machine and is used to **digitally sign challenges** sent by the SSH server during authentication.
- The **public key**, which I installed on the remote server, is used to **verify the authenticity** of the signature.
- The server can confirm that the signature is valid **without ever seeing or learning the private key**, thanks to asymmetric cryptography.
- **Ed25519** is preferred because it is **faster**, **more secure**, and **smaller** than older algorithms like RSA, with strong resistance against known cryptographic attacks.
- This method ensures a highly secure, passwordless login system for accessing remote servers.

---


---

### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

   * How SSH reads `~/.ssh/config` and matches hosts.
   * The difference between `HostName` and `Host`.
   * How aliases prevent long commands.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
# 2) A short explanation (3–4 sentences) of how the config simplifies connections
```
# SSH Config File – Simplifying SSH Connections

## 1. Full Contents of `~/.ssh/config`

```ssh-config
Host chat
    HostName chat.erarslan.tk
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

This configuration allows me to connect to the server with a short alias.

---

## 2. Explanation – How `~/.ssh/config` Simplifies Connections

The `~/.ssh/config` file allows SSH to use **custom aliases** (defined by `Host`) for remote connections. When I type `ssh chat`, SSH reads this file, finds the matching `Host chat` block, and uses the specified `HostName`, `User`, and `IdentityFile` for the connection.

- `Host` is the **alias** I can use in the terminal.
- `HostName` is the **real server address**.
- This avoids typing long, repetitive commands like:

```bash
ssh ubuntu@chat.erarslan.tk -i ~/.ssh/id_ed25519
```

That entire command is now simplified to just:

```bash
ssh chat
```
By defining multiple entries, I can manage multiple servers with short, easy-to-remember names. This makes my workflow faster, cleaner, and less error-prone.

---

### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran
# 2) Any flags or options used
# 3) A brief explanation (2–3 sentences) of scp’s mechanism
```
# SCP File Transfers – Task 4

## 1. SCP Commands Used

```bash
# Local → Remote
scp .ssh/testing.txt ubuntu@chat.erarslan.tk:~/.ssh

# Remote → Local
scp ubuntu@chat.erarslan.tk:~/.ssh/testing_remote.txt ./.ssh

# Remote → Remote (on same host)
scp ubuntu@chat.erarslan.tk:~/.ssh/testing_remote.txt ubuntu@chat.erarslan.tk:~
```

---

## 2. Flags or Options Used

- No additional flags were required for these specific transfers.
- The `-r` flag is only necessary for copying directories recursively.
- The default port 22 was used, so no `-P` flag was needed.

---

## 3. SCP Mechanism – Short Explanation

SCP (**Secure Copy Protocol**) initiates an **SSH session** in the background to securely transfer files between hosts. It ensures **confidentiality and integrity** by encrypting all transferred data, including filenames and file content. Authentication and key verification are handled exactly like standard SSH, making it a secure and reliable way to move files across systems.

---

> After each transfer, I verified the file with `ls -la` to confirm matching **timestamps** and **file sizes**, ensuring a successful copy.

---

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells).
   * Why and when each file is read.
   * How sourcing differs from executing.

**Provide:**

```bash
# 1) The contents of login_tasks.sh
# 2) The lines you added to ~/.bashrc or ~/.profile
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
```

# Login Shell Script & Profile Setup

## 1) Contents of `~/login_tasks.sh`

```bash
#!/usr/bin/env bash 

cat << "KEY"
      _______
    /         \
   |           |
    \_________/
        ||
        ||
        ||
        ||
        ||
        ||____
        ||    \
        ||     |
        ||____/

Welcome to Erarslan Technologies, ubuntu! Today is Wed May 14 16:17:01 UTC 2025.
KEY

uptime
ls ~/projects
```

---

## 2) Line Added to `~/.bashrc`

```bash
source ~/login_tasks.sh
```

---

## 3) Explanation – Shell Init Files & Sourcing vs. Executing

- `~/.bashrc` is executed for **interactive non-login shells**, such as when opening a new terminal tab or session inside an already logged-in shell.  
- `~/.profile` is used for **login shells**, such as when logging in via SSH or TTY.  
- By adding `source ~/login_tasks.sh` to `.bashrc`, the script is automatically run at the start of every new terminal session, keeping it interactive.

**Sourcing** (`source script.sh`) runs the script **in the current shell context**, allowing variables and changes to persist in the environment.  
**Executing** (e.g., `./script.sh`) runs the script in a **subshell**, which does not affect the current shell's environment.

---

**Remember:** Stop working after **90 minutes** and record where you stopped.

Total used time: 55 Minutes
