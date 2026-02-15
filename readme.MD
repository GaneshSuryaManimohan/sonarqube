# SonarQube Installation on AWS EC2 Instance

**SSH into the AWS EC2 Ubuntu Server:**

Example (For ubuntu)
```bash
    ssh -i helloworld.pem ubuntu@55.63.86.254
```

---
## 1️⃣ **System Update**
```bash
sudo apt update
```

---
## 2️⃣ **Install Java**
SonarQube 10 requires Java 17+.

```bash
sudo apt install -y fontconfig openjdk-17-jdk
java -version
```

---
## 3️⃣ Install PostgreSQL

* Add PostgreSQL Repository
    - ```bash sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' ```

* Add the PostgreSQL signing key: 
    - ```bash  wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add - ```


* Install PostgreSQL
    - ```bash sudo apt install postgresql postgresql-contrib -y ```

* Enable, start, & status check of the database server: 
    ```bash sudo systemctl enable postgresql ```
    ```bash sudo systemctl start postgresql ```
    ```bash sudo systemctl status postgresql ```

* Verify: psql --version

---
## 4️⃣ Configure Database for SonarQube

* Switch to postgres user:
```bash
sudo -i -u postgres
```

* Create user and database:
    1. Create User manimohan: ```bash createuser manimohan ```

    2. Log in to PostgreSQL: ```bash  psql ```

    3. Set a password for the manimohan user. Use a strong password in place of your_password_here:
        ```bash  ALTER USER ddsonar WITH ENCRYPTED PASSWORD 'your_password_here';```

    4. Create a SonarQube database and set the owner to manimohan:
        ```bash  CREATE DATABASE mmsonarqube OWNER manimohan;```

    5. Grant all the privileges on the mmsonarqube database to the manimohan user:
        ```bash  GRANT ALL PRIVILEGES ON DATABASE mmsonarqube TO manimohan;```
    
    6. To check the created database: ```bash \l ```

    7. To check the created database user: ```bash  \du ```

    8. Exit PostgreSQL: ```bash  \q ```

    9. Return to your non-root sudo user account. ```bash  exit```

---
## 5️⃣ Download and Install SonarQube

* Install the zip utility, which is needed to unzip the SonarQube files
    - ```bash sudo apt install zip -y```

* Here we are installing the latest version of SonarQube 10.0 community edition (Free)
    - ```bash  sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.0.0.68432.zip ```

* Unzip the downloaded file:
    - ```bash  sudo unzip sonarqube-10.0.0.68432.zip ``` 

* Move the unzipped files to /opt/sonarqube directory:
    - ```bash  sudo mv sonarqube-10.0.0.68432 sonarqube ```
    - ```bash  sudo mv sonarqube /opt/ ```

---
## 6️⃣ Create SonarQube Group and User

* Create a dedicated user and group for SonarQube, which can not run as the root user.

* Create a sonar group:
    - ```bash  sudo groupadd manimohan ```

* Create a sonar user and set /opt/sonarqube as the home directory.
    - ```bash  sudo useradd -d /opt/sonarqube -g manimohan manimohan```

* Grant the sonar user access to the /opt/sonarqube directory
    - ```bash  sudo chown manimohan:manimohan /opt/sonarqube -R ```

---
## 7️⃣ Configure SonarQube Database Connection

* Edit the SonarQube configuration file:
    - ```bash  sudo vi /opt/sonarqube/conf/sonar.properties ```

* Find the following lines:
                `#sonar.jdbc.username=`    
                `#sonar.jdbc.password=`
    - Uncomment the lines, and add the database user and Database password you created in steps 4️⃣ & 6️⃣
        `sonar.jdbc.username=manimohan`
        `sonar.jdbc.password=your_password_here`
    
    - Below these two lines, add the following line of code: Here, mmsonarqube is the database name created.
        `sonar.jdbc.url=jdbc:postgresql://localhost:5432/mmsonarqube`

    - Save and exit the file

* Edit the sonar script file
    - ```bash  sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh```

    - Add the following line: ```bash  RUN_AS_USER=manimohan```

    - Save and exit the file

---

## 8️⃣ Setup Systemd service

* Create a systemd service file to start SonarQube at system boot:
    - ```bash  sudo vi /etc/systemd/system/sonar.service```

    - Paste the following lines to the file:
        [Unit]
        Description=SonarQube service
        After=syslog.target network.target
        [Service]
        Type=forking
        ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
        ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
        User=manimohan
        Group=manimohan
        Restart=always
        LimitNOFILE=65536
        LimitNPROC=4096
        [Install]
        WantedBy=multi-user.target

    - Save and exit the file

* Enable the SonarQube service to run at system startup:
    - ```bash  sudo systemctl enable sonar ```

* Start the SonarQube service:
    - ```bash  sudo systemctl start sonar ```

* Check the service status:
    - ```bash  sudo systemctl status sonar ```

---
## 9️⃣ Configure Kernel Parameters (Required for Elasticsearch)

SonarQube uses Elasticsearch to store its indices in an MMap FS directory. It requires some changes to the system defaults.

* Edit the sysctl configuration file:
    - ```bash  sudo vi /etc/sysctl.conf ```
    
    - Add the following lines:
        vm.max_map_count=262144
        fs.file-max=65536
        ulimit -n 65536
        ulimit -u 4096

* Save and exit the file 

* Reboot the system to apply the changes: ```bash  sudo reboot ```

---
## 🔟 Access SonarQube Web Interface

* Access SonarQube in a web browser at your server’s IP address on port 9000.
    Example: http://13.220.121.51:9000/

Note: the default username and password are admin and admin respectively.

* Change the Old password with a New one:
    - Log in with username admin and password admin. 
    - In the next step, SonarQube will prompt you to change your password. CHANGE THE PASSWORD.
---




