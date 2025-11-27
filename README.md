# Jenkins CI/CD Pipeline for Shoe Shopping Cart (Spring Boot Application)

This repository contains a fully automated CI/CD pipeline built with Jenkins to deploy, update, manage logs, and rollback the **Shoe Shopping Cart** Spring Boot application.

The pipeline uses **Scripted Pipeline (Groovy)** and supports **multi-server deployment with parameters**, backup management, and remote application lifecycle control.

---

## 🚀 Features

### ✔ Full deployment lifecycle
- **start** application  
- **stop** running instance  
- **upcode** (checkout → build → deploy → restart)  
- **backup** current version  
- **rollback** to any previous backup  
- **showlog_line** (tail logs)  
- **showlog_keyword** (grep logs)

### ✔ Additional highlights
- Maven build (`mvn clean install -DskipTests=true`)
- Zero-downtime workflow (stop → update → restart)
- Auto-permission correction (`chown shoeshop`)
- Backup with timestamp naming
- Parameterized Jenkins job
- Multi-node deployment (`node(params.server)`)

---

## 🧩 Technologies Used

- Jenkins (Scripted Pipeline)
- Groovy
- Linux commands (nohup, ps, kill, grep, zip)
- Git/GitLab SCM
- Maven
- Java Spring Boot
- Shell scripting

---

## 📂 Deployment Directory Structure

Application is deployed under:

```
/data/shoeshop/
│── run/           # Running JAR & nohup.out
│── backups/       # ZIP backups with timestamps
│── logs/          # Additional logs (optional)
```

Generated JAR name:
```
shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

Backup name format:
```
shoe-ShoppingCart_DDMMYYYY_HHMM.zip
```

---

## 🏗 CI/CD Pipeline Flow (upcode)

```
upcode →
    1. backup current version
    2. stop application
    3. checkout git commit (params.hash)
    4. build with Maven
    5. deploy JAR to /data/shoeshop/run
    6. apply permissions
    7. start new version
    8. verify PID running
```

---

## 🗂 Repository Structure

```
jenkins-automation-shoeshop/
│── Jenkinsfile
│── README.md
│── scripts/        # optional
│── images/         # optional diagrams
```

---

## 🧾 Jenkins Parameters

| Parameter | Type | Description |
|----------|------|-------------|
| **server** | Choice | Which Jenkins node/agent to deploy |
| **action** | Choice | start / stop / upcode / rollback / showlog_line / showlog_keyword |
| **hash** | String | Git commit hash for updating code |
| **rollback_version** | Choice | Backup ZIP file to restore |
| **line** | String/Number | Number of log lines to show |
| **keyword** | String | Keyword to search in logs |

---

## 🔧 Actions Detail

### 1️⃣ **start**
Runs the application:

```
nohup java -jar <jar> > nohup.out 2>&1 &
```

Pipeline verifies PID:

```
ps -ef | grep <process>
```

---

### 2️⃣ **stop**
Kills the running process:

```
kill -9 <pid>
```

Skips if no PID found.

---

### 3️⃣ **backup**
Creates a backup ZIP:

```
zip -jr /data/shoeshop/backups/<APP>_<timestamp>.zip /data/shoeshop/run
```

---

### 4️⃣ **upcode**
Full code update sequence:

```
checkout → build → deploy → start → verify
```

- Checkout branch/commit using params.hash  
- Build with Maven  
- Copy JAR to runtime directory  
- Apply permissions  
- Restart application  

---

### 5️⃣ **rollback**
Restores previous version:

```
rm -rf /data/shoeshop/run/*
unzip <backup>.zip -d /data/shoeshop/run
```

---

### 6️⃣ **showlog_line**
Tail last N log lines:

```
tail -n <params.line> nohup.out
```

---

### 7️⃣ **showlog_keyword**
Search for keyword in log:

```
grep -n "<keyword>" nohup.out
```

---

## 📘 Jenkinsfile Overview

The Jenkinsfile includes:

- Global configuration  
- Utility functions (`getProcessId`)  
- Deployment actions  
- Log inspection tools  
- Main action dispatcher via `params.action`  
- Multi-server execution using:

```
node(params.server)
```

---

## 🔒 Security Notes

- Uses Jenkins credentials:  
  `jenkins-gitlab-user`
- Application runs under restricted Linux user:  
  `shoeshop`
- No plaintext passwords inside pipeline
- All sensitive commands executed using:

```
sudo su shoeshop -c "<command>"
```

---

## 📥 How to Use

### 1️⃣ Create a Jenkins Pipeline job  
### 2️⃣ Enable **“This project is parameterized”**  
### 3️⃣ Add parameters exactly as listed  
### 4️⃣ Set SCM:

```
Git → https://github.com/<your-username>/jenkins-automation-shoeshop.git
```

### 5️⃣ Script path:

```
Jenkinsfile
```

---

## 📊 Example Usage

### ▶ Start application
```
action = start
```

### ⏹ Stop application
```
action = stop
```

### 🔄 Update to commit `abc123`
```
action = upcode
hash = abc123
```

### ⏪ Rollback:
```
action = rollback
rollback_version = shoe-ShoppingCart_25112025_1422.zip
```

### 📄 Show last 200 lines:
```
action = showlog_line
line = 200
```

### 🔍 Search for "ERROR"
```
action = showlog_keyword
keyword = ERROR
```

---






