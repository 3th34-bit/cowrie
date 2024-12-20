Cowrie
######

Welcome to the Cowrie GitHub repository
*****************************************

This is the official repository for the Cowrie SSH and Telnet
Honeypot effort.

What is Cowrie
*****************************************

Cowrie is a medium to high interaction SSH and Telnet honeypot
designed to log brute force attacks and the shell interaction
performed by the attacker. In medium interaction mode (shell) it
emulates a UNIX system in Python, in high interaction mode (proxy)
it functions as an SSH and telnet proxy to observe attacker behavior
to another system.

`Cowrie <http://github.com/cowrie/cowrie/>`_ is maintained by Michel Oosterhof.

Documentation
****************************************

The Documentation can be found `here <https://cowrie.readthedocs.io/en/latest/index.html>`_.

Slack
*****************************************

You can join the Cowrie community at the following `Slack workspace <https://www.cowrie.org/slack/>`_.

Features
*****************************************

* Choose to run as an emulated shell (default):
   * Fake filesystem with the ability to add/remove files. A full fake filesystem resembling a Debian 5.0 installation is included
   * Possibility of adding fake file contents so the attacker can `cat` files such as `/etc/passwd`. Only minimal file contents are included
   * Cowrie saves files downloaded with wget/curl or uploaded with SFTP and scp for later inspection

* Or proxy SSH and telnet to another system
   * Run as a pure telnet and ssh proxy with monitoring
   * Or let Cowrie manage a pool of QEMU emulated servers to provide the systems to login to

For both settings:

* Session logs are stored in an `UML Compatible <http://user-mode-linux.sourceforge.net/>`_  format for easy replay with the `bin/playlog` utility.
* SFTP and SCP support for file upload
* Support for SSH exec commands
* Logging of direct-tcp connection attempts (ssh proxying)
* Forward SMTP connections to SMTP Honeypot (e.g. `mailoney <https://github.com/awhitehatter/mailoney>`_)
* JSON logging for easy processing in log management solutions

Docker
*****************************************

Docker versions are available.

* To get started quickly and give Cowrie a try, run::

    $ docker run -p 2222:2222 cowrie/cowrie:latest
    $ ssh -p 2222 root@localhost

* On Docker Hub: https://hub.docker.com/r/cowrie/cowrie

Configuring Cowrie in Docker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cowrie in Docker can be configured using environment variables. The
variables start with COWRIE_ then have the section name in capitals,
followed by the stanza in capitals. An example is below to enable
telnet support::

    COWRIE_TELNET_ENABLED=yes

Alternatively, Cowrie in Docker can use an `etc` volume to store
configuration data.  Create `cowrie.cfg` inside the etc volume
with the following contents to enable telnet in your Cowrie Honeypot
in Docker::

    [telnet]
    enabled = yes

Requirements
*****************************************

Software required to run locally:

* Python 3.9+
* python-virtualenv

For Python dependencies, see `requirements.txt <https://github.com/cowrie/cowrie/blob/master/requirements.txt>`_.

Files of interest:
*****************************************

* `etc/cowrie.cfg` - Cowrie's configuration file.
* `etc/cowrie.cfg.dist <https://github.com/cowrie/cowrie/blob/master/etc/cowrie.cfg.dist>`_ - default settings, don't change this file
* `etc/userdb.txt` - credentials to access the honeypot
* `src/cowrie/data/fs.pickle` - fake filesystem, this only contains metadata (path, uid, gid, size)
* `honeyfs/ <https://github.com/cowrie/cowrie/tree/master/honeyfs>`_ - contents for the fake filesystem
* `honeyfs/etc/issue.net` - pre-login banner
* `honeyfs/etc/motd <https://github.com/cowrie/cowrie/blob/master/honeyfs/etc/issue>`_ - post-login banner
* `src/cowrie/data/txtcmds/` - output for simple fake commands
* `var/log/cowrie/cowrie.json` - audit output in JSON format
* `var/log/cowrie/cowrie.log` - log/debug output
* `var/lib/cowrie/tty/` - session logs, replayable with the `bin/playlog` utility.
* `var/lib/cowrie/downloads/` - files transferred from the attacker to the honeypot are stored here
* `bin/createfs` - create your own fake filesystem
* `bin/playlog` - utility to replay session logs



# Expanding Cowrie Honeypot with Fake Services

This document provides a detailed guide on how to configure and enhance your Cowrie honeypot with additional fake services to simulate a more enticing and interactive environment for attackers. Each section includes step-by-step instructions for setting up various fake services.

---

## **1. Fake Web Server**
### Description
Set up a fake web server that displays an admin panel or sensitive-looking pages.

### Steps
1. Start a lightweight HTTP server using Python:
   ```bash
   python3 -m http.server 8080
   ```

2. Create a fake admin page:
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>Admin Login</title></head>
   <body>
       <h1>Welcome to Admin Panel</h1>
       <form>
           Username: <input type="text" name="username"><br>
           Password: <input type="password" name="password"><br>
           <button type="submit">Login</button>
       </form>
   </body>
   </html>
   ```
3. Place the file in the directory where the server is running.
4. Monitor access attempts in the server logs.

---

## **2. Fake MySQL Service**
### Description
Simulate an exposed MySQL database.

### Steps
1. Install `mysql_fake_server`:
   ```bash
   pip install mysql_fake_server
   ```

2. Run the fake server:
   ```bash
   mysql_fake_server --host 0.0.0.0 --port 3306
   ```

3. Customize the fake database with enticing data (e.g., `users` table with fake credentials).

---

## **3. Fake FTP Server**
### Description
Simulate an open FTP server that pretends to allow anonymous access.

### Steps
1. Install `pyftpdlib`:
   ```bash
   pip install pyftpdlib
   ```

2. Run the FTP server:
   ```bash
   python3 -m pyftpdlib -p 21
   ```

3. Add fake files to a directory (e.g., `confidential_data.csv`, `passwords.txt`) and log download attempts.

---

## **4. Fake SMTP Server**
### Description
Simulate a mail server to capture login credentials or outgoing messages.

### Steps
1. Use `FakeSMTP` with Docker:
   ```bash
   docker run -d -p 1025:25 maildev/maildev
   ```

2. Access the web interface at `http://<honeypot-ip>:1080` to view captured emails.
3. Customize SMTP responses to further trick attackers.

---

## **5. Fake Telnet Service**
### Description
Enable Telnet with an enticing banner and basic interaction.

### Steps
1. Edit `cowrie.cfg` to enable Telnet:
   ```ini
   [telnet]
   enabled = true
   listen_endpoints = tcp:23:interface=0.0.0.0
   ```

2. Customize the Telnet prompt by editing the Cowrie `motd`:
   - Open `etc/telnet_banner.txt` and add text like:
     ```
     Welcome to Classified Network System.
     Unauthorized access is strictly prohibited.
     ```

3. Restart Cowrie:
   ```bash
   ./bin/cowrie restart
   ```

---

## **6. Fake Redis Server**
### Description
Simulate an exposed Redis instance.

### Steps
1. Install `fakeredis`:
   ```bash
   pip install fakeredis
   ```

2. Run a basic Redis fake server:
   ```python
   import fakeredis

   server = fakeredis.FakeServer()
   connection = fakeredis.FakeRedis(server=server)

   connection.set("admin:password", "hunter2")
   connection.set("flag", "CTF{fake_redis_honeypot}")
   print("Fake Redis server running!")
   ```

3. Log all attempted commands.

---

## **7. Fake SMB/Windows Shares**
### Description
Simulate a vulnerable file-sharing server.

### Steps
1. Use `smbserver.py` from Impacket:
   ```bash
   git clone https://github.com/SecureAuthCorp/impacket.git
   cd impacket
   pip install .
   ```

2. Run a fake SMB server:
   ```bash
   smbserver.py fakeShare $(pwd)/fake_share -ip 0.0.0.0
   ```

3. Populate the `fake_share` directory with enticing files (e.g., `secrets.docx`, `internal_report.pdf`).

---

## **8. Fake API Server**
### Description
Simulate an API with sensitive-looking endpoints.

### Steps
1. Install Flask:
   ```bash
   pip install flask
   ```

2. Create a basic API:
   ```python
   from flask import Flask, request

   app = Flask(__name__)

   @app.route('/api/login', methods=['POST'])
   def login():
       username = request.form.get('username')
       password = request.form.get('password')
       with open('api_attempts.log', 'a') as f:
           f.write(f"Login attempt: {username} - {password}\n")
       return {"message": "Invalid credentials"}, 401

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

3. Run the server:
   ```bash
   python3 fake_api.py
   ```

---

## **9. Fake Blockchain Wallet**
### Description
Simulate a Bitcoin wallet with a vulnerable API.

### Steps
1. Add a JSON-RPC API that accepts "transactions":
   ```python
   @app.route('/api/sendbtc', methods=['POST'])
   def send_btc():
       wallet = request.form.get('wallet')
       amount = request.form.get('amount')
       with open('blockchain_attempts.log', 'a') as f:
           f.write(f"Wallet: {wallet}, Amount: {amount}\n")
       return {"status": "Transaction failed"}, 400
   ```

2. Log all transaction attempts.

---

## **10. Fake NFS (Network File System)**
### Description
Simulate a writable NFS share.

### Steps
1. Install and configure `unfs3`:
   ```bash
   sudo apt install unfs3
   ```

2. Configure `/etc/exports` with a writable share:
   ```
   /fake_nfs *(rw,sync,no_subtree_check)
   ```

3. Restart the service and monitor for connections.

---

## **Tips for Success**
1. **Log Everything:** Ensure all fake services log interactions for later analysis.
2. **Entice Attackers:** Populate fake services with enticing but fake sensitive data.
By implementing these fake services, you can create a more interactive and engaging honeypot environment while gathering valuable insights into attacker behavior.

# Expanding Cowrie Honeypot with Fake Services

This document provides a detailed guide on how to configure and enhance your Cowrie honeypot with additional fake services to simulate a more enticing and interactive environment for attackers. Each section includes step-by-step instructions for setting up various fake services and commands, as well as creating enticing fake files for a cybersecurity project.

---

## **1. Fake Web Server**

### Description

Set up a fake web server that displays an admin panel or sensitive-looking pages.

### Steps

1. Start a lightweight HTTP server using Python:

   ```bash
   python3 -m http.server 8080
   ```

2. Create a fake admin page:

   ```html
   <!DOCTYPE html>
   <html>
   <head><title>Admin Login</title></head>
   <body>
       <h1>Welcome to Admin Panel</h1>
       <form>
           Username: <input type="text" name="username"><br>
           Password: <input type="password" name="password"><br>
           <button type="submit">Login</button>
       </form>
   </body>
   </html>
   ```

3. Place the file in the directory where the server is running.

4. Monitor access attempts in the server logs.

---

## **2. Fake Files with Enticing Names**

Creating enticing files is an excellent way to lure attackers deeper into your honeypot. Below are examples of fake files organized by location.

### **In the `home/admin` Directory**

```bash
echo "AWS_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" > home/admin/aws_keys.txt
echo "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEArandomexample" > home/admin/id_rsa.pub
echo "Confidential Project Notes" > home/admin/project_notes.txt
echo "username: admin\npassword: admin2024!" > home/admin/credentials_backup.csv
echo "smtp.example.com\nusername: marketing@example.com\npassword: marketingPass123!" > home/admin/email_settings.ini
```

### **In the `var/log` Directory**

```bash
echo "ERROR: Unauthorized access detected on $(date)" > var/log/auth.log
echo "2024-12-11 12:34:56 - Database query failed: 'SELECT * FROM users WHERE admin=1'" > var/log/db_error.log
echo "Session Timeout Error\nUser: root\nAttempts: 3\n" > var/log/session_timeout.log
echo "Kernel Panic - Attempted write to readonly memory at 0xDEADBEEF" > var/log/kern.log
```

### **In the `etc/secure` Directory**

```bash
echo "admin = supersecurepassword123" > etc/secure/app_config.ini
echo "Database Connection String: postgresql://user:password123@db.example.com:5432/app" > etc/secure/db_conn.conf
echo "Encrypted Volume Key: 34f72ea6e99ba4e876d3c23c7b7b77e3" > etc/secure/disk_encryption_key.txt
echo "Do not share: CEO's password: CEOsuperSecure2024!" > etc/secure/password_policy.txt
```

### **Add Fake SSH Configurations**

```bash
mkdir -p home/admin/.ssh
echo "Host secret-server\n    HostName 192.168.10.10\n    User admin\n    IdentityFile ~/.ssh/id_rsa" > home/admin/.ssh/config
echo "-----BEGIN OPENSSH PRIVATE KEY-----\nFAKEKEYDATA\n-----END OPENSSH PRIVATE KEY-----" > home/admin/.ssh/id_rsa
chmod 600 home/admin/.ssh/id_rsa
```

### **Add Fake System Files**

```bash
echo "127.0.0.1 localhost\n192.168.10.20 secret.internal" > etc/hosts
echo "net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1" > etc/sysctl.conf
echo "root:x:0:0:root:/root:/bin/bash\nadmin:x:1000:1000:Admin:/home/admin:/bin/bash" > etc/passwd
echo "admin:$6$randomsalt$hashedpasswordhere" > etc/shadow
```

### **Add Fake Cron Jobs**

```bash
mkdir -p var/spool/cron/crontabs
echo "0 3 * * * root /bin/bash /root/backup.sh" > var/spool/cron/crontabs/root
echo "30 2 * * 1 root /usr/bin/python3 /home/admin/weekly_report.py" > var/spool/cron/crontabs/admin
```

### **Add Fake Scripts and Executables**

#### Fake Backup Script

```bash
echo "#!/bin/bash\nzip -r /backup/important_data.zip /etc/secure" > root/backup.sh
chmod +x root/backup.sh
```

#### Fake Python Script

```bash
echo "import os\nos.system('echo Running weekly report...')" > home/admin/weekly_report.py
```

### **Add Fake Database Dumps**

```bash
mkdir -p var/backups
echo "INSERT INTO users (username, password) VALUES ('admin', 'password123');" > var/backups/db_dump.sql
echo "{\"username\": \"admin\", \"password\": \"admin2024!\"}" > var/backups/user_data.json
```

### **Add Fake Docker and Kubernetes Files**

```bash
mkdir -p var/lib/docker
echo "services:\n  app:\n    image: example/app:latest\n    environment:\n      - DB_PASSWORD=SuperSecretPassword" > var/lib/docker/docker-compose.yml

mkdir -p var/lib/kubernetes
echo "apiVersion: v1\nkind: Secret\nmetadata:\n  name: db-credentials\ndata:\n  username: YWRtaW4=\n  password: c3VwZXJzZWNyZXRwYXNz" > var/lib/kubernetes/db-secret.yaml
```

### **Add Fake Email Content**

```bash
mkdir -p var/mail
echo "To: ceo@example.com\nFrom: admin@example.com\nSubject: Quarterly Report\nBody: Confidential information attached." > var/mail/quarterly_report.eml
```

---

## **3. Testing and Documentation for Fake Commands**

### Steps to Test

1. SSH into your honeypot:
   ```bash
   ssh root@<honeypot-ip>
   ```
2. Run each fake command and observe the behavior.
3. Take notes on:
   - The command’s output.
   - Any amusing or frustrating effects it causes.

### Example Commands

#### Command: `passwd`
- **Description**: Simulates changing a password but never succeeds.
- **Expected Output**:
  ```plaintext
  Changing password for root.
  Current password:
  New password:
  Retype new password:
  passwd: password updated successfully.
  ```
- **Notes**: Logs all input to `passwd_attempts.log`. Appears to work but does nothing.
- **Test Status**: Pass.

#### Command: `sudo`
- **Description**: Always denies access.
- **Expected Output**:
  ```plaintext
  sudo: unable to resolve host classified-server
  Permission denied.
  ```
- **Notes**: Includes a fake hostname for added realism.
- **Test Status**: Pass.

#### Command: `dd`
- **Description**: Pretends to write data indefinitely.
- **Expected Output**:
  ```plaintext
  dd: writing data...
  ```
  (Then hangs indefinitely.)
- **Notes**: Hangs the session, requiring the attacker to terminate manually.
- **Test Status**: Pass.

---

## **4. Final Steps and Maintenance**

1. Restart Cowrie to apply changes:
   ```bash
   ./bin/cowrie stop
   ./bin/cowrie start
   ```
2. Regularly review logs to analyze attacker behavior and improve your honeypot.
3. Continuously refine fake files and commands to stay ahead of attacker tactics.

By implementing these steps, you’ll create a realistic, engaging, and frustrating honeypot environment that captures valuable data for your cybersecurity project.




Contributors
***************

Many people have contributed to Cowrie over the years. Special thanks to:

* Upi Tamminen (desaster) for all his work developing Kippo on which Cowrie was based
* Dave Germiquet (davegermiquet) for TFTP support, unit tests, new process handling
* Olivier Bilodeau (obilodeau) for Telnet support
* Ivan Korolev (fe7ch) for many improvements over the years.
* Florian Pelgrim (craneworks) for his work on code cleanup and Docker.
* Guilherme Borges (sgtpepperpt) for SSH and telnet proxy (GSoC 2019)
* And many many others.
