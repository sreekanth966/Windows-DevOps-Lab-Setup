# 🚀 Windows DevOps Lab

## Java + Jenkins + Nexus Repository + SonarQube + Trivy + Apache Tomcat

<div align="center">

<table>
<tr>
<td align="center" width="140"><img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/java/java-original.svg" width="36" height="36" alt="Java"><br><sub><b>Java</b></sub></td>
<td align="center" width="140"><img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/jenkins/jenkins-original.svg" width="36" height="36" alt="Jenkins"><br><sub><b>Jenkins</b></sub></td>
<td align="center" width="140"><img src="https://commons.wikimedia.org/wiki/Special:Redirect/file/Logo_of_Sonatype_Nexus_Repository.svg" width="36" height="36" alt="Nexus Repository"><br><sub><b>Nexus</b></sub></td>
<td align="center" width="140"><img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/sonarqube/sonarqube-original.svg" width="36" height="36" alt="SonarQube"><br><sub><b>SonarQube</b></sub></td>
<td align="center" width="140"><img src="https://cdn.jsdelivr.net/gh/glincker/thesvg@main/public/icons/trivy/default.svg" width="36" height="36" alt="Trivy"><br><sub><b>Trivy</b></sub></td>
<td align="center" width="140"><img src="https://cdn.freebiesupply.com/logos/large/2x/tomcat-logo-black-and-white.png" width="36" height="36" alt="Apache Tomcat"><br><sub><b>Tomcat</b></sub></td>
</tr>
</table>

<p>
<img src="https://img.shields.io/badge/OS-Windows-0078D6?style=flat-square&logo=windows&logoColor=white" alt="Windows">
<img src="https://img.shields.io/badge/Java-21-ED8B00?style=flat-square&logo=openjdk&logoColor=white" alt="Java 21">
<img src="https://img.shields.io/badge/Windows%20Services-Enabled-5C2D91?style=flat-square" alt="Windows Services">
</p>

</div>

------------------------------------------------------------------------

## 🎯 Purpose

This guide provides a clean installation procedure for a Windows-based
DevOps lab using:

-   ☕ Java JDK
-   🔵 Jenkins
-   🟠 Nexus Repository
-   🔍 SonarQube
-   🛡️ Trivy — vulnerability and image scanning
-   🐱 Apache Tomcat

All application web ports are intentionally moved away from their
defaults and assigned sequentially from **8050**.

> **Run installation commands from an Administrator Command Prompt
> unless stated otherwise.**

------------------------------------------------------------------------

# 🏗️ Architecture

The recommended pipeline order is:

```text
Developer / Git
      │
      ▼
🔵 Jenkins :8050
      │
      ▼
Checkout Code
      │
      ▼
Build & Unit Test
      │
      ├──────────────► 🔍 SonarQube :8052
      │                 Code Quality / Quality Gate
      │
      ├──────────────► 🛡️ Trivy
      │                 Vulnerability / Security Scan
      │
      ▼
Quality & Security Gates
      │
      ▼
Package Application
      │
      ▼
🟠 Nexus Repository :8051
      │
      │  Store Versioned Artifact
      ▼
🐱 Apache Tomcat :8053
      │
      ▼
🚀 Application Deployed
```

### 🔄 CI/CD Flow

```text
1. Git Push
      ↓
2. Jenkins :8050
      ↓
3. Checkout
      ↓
4. Build
      ↓
5. Unit Test
      ↓
6. SonarQube :8052
      └── Code Quality / Quality Gate
      ↓
7. Trivy
      └── Vulnerability / Security Scan
      ↓
8. Quality / Security Gates
      ↓
9. Package WAR/JAR
      ↓
10. Nexus Repository :8051
      └── Store Versioned Artifact
      ↓
11. Apache Tomcat :8053
      └── Deploy Application
      ↓
12. Application Running
```

> **Important:** SonarQube and Trivy are analysis/scanning tools invoked by Jenkins. They are not normally placed in the application traffic path. Nexus stores the approved versioned artifact, and Tomcat deploys that artifact.

# 📦 Software

| Component | File / Package | Purpose | Default Port | Lab Port |
|---|---|---|---:|---:|
| ☕ Java | JDK | Runtime | — | — |
| 🔵 Jenkins | `jenkins.msi` | CI/CD Automation | `8080` | **8050** |
| 🟠 Nexus | `nexus-3.85.0-03-win-x86_64.zip` | Artifact Repository | `8081` | **8051** |
| 🔍 SonarQube | `sonarqube-25.11.0.114957.zip` | Code Quality | `9000` | **8052** |
| 🛡️ Trivy | Windows 64-bit ZIP | Vulnerability / Security Scanning | **CLI** | **No Web Port** |
| 🐱 Tomcat | `apache-tomcat-9.0.111.exe` | Application Server | `8080` | **8053** |

> **Port plan:** `8050 → 8051 → 8052 → 8053`  
> **Trivy is a command-line security scanner, not a web application, so it does not require a web port.**

### Default vs Lab Ports

| Tool | Default Port | Lab Port |
|---|---:|---:|
| Jenkins | `8080` | **8050** |
| Nexus | `8081` | **8051** |
| SonarQube | `9000` | **8052** |
| Trivy | CLI | **No Web Port** |
| Tomcat | `8080` | **8053** |


# 📁 Recommended Directory Structure

Create:

``` cmd
mkdir C:\DevOps
mkdir C:\DevOps\Nexus
mkdir C:\DevOps\SonarQube
mkdir C:\DevOps\Tomcat
```

Recommended structure:

``` text
C:\DevOps\
│
├── Nexus\
│   └── nexus-3.85.0-03\
│
├── SonarQube\
│   └── sonarqube-25.11.0.114957\
│
└── Tomcat\
    └── apache-tomcat-9.0.111\
```

> **Nexus:** avoid paths containing spaces. Sonatype specifically
> documents that the Windows Nexus installation path should not contain
> spaces.

------------------------------------------------------------------------

# 1️⃣ Install Java JDK

## Step 1 --- Install JDK

Install a JDK supported by the versions of the applications you are
using.

For the current Jenkins Windows installer documentation, Jenkins
requires **Java 21 or later**.

After installation, open **Administrator CMD**.

## Step 2 --- Verify Java

``` cmd
java -version
```

``` cmd
javac -version
```

``` cmd
where java
```

Example:

``` text
java version "21.x.x"
javac 21.x.x
```

## Step 3 --- Configure JAVA_HOME

Open:

``` text
System Properties
  → Advanced
  → Environment Variables
```

Under **System variables**, create:

``` text
JAVA_HOME
```

Example:

``` text
C:\Program Files\Java\jdk-21
```

Edit the system `Path` and add:

``` text
%JAVA_HOME%\bin
```

Close and reopen Command Prompt.

Verify:

``` cmd
echo %JAVA_HOME%
java -version
javac -version
```

## Uninstall Java JDK

### Step 1 --- Remove JDK

Open:

``` text
Control Panel
  → Programs
  → Programs and Features
```

Select the installed JDK and click:

``` text
Uninstall
```

### Step 2 --- Remove JAVA_HOME

Open:

``` text
System Properties
  → Advanced
  → Environment Variables
```

Under **System variables**, delete:

``` text
JAVA_HOME
```

### Step 3 --- Remove from PATH

Edit the system `Path` and remove the entry:

``` text
%JAVA_HOME%\bin
```

Close and reopen Command Prompt.

Verify removal:

``` cmd
java -version
```

``` text
'java' is not recognized as an internal or external command
```

------------------------------------------------------------------------

# 2️⃣ Install Nexus Repository

### File

``` text
nexus-3.85.0-03-win-x86_64.zip
```

## Step 1 --- Extract

Extract to:

``` text
C:\DevOps\Nexus
```

Expected:

``` text
C:\DevOps\Nexus\
└── nexus-3.85.0-03\
```

Nexus will use a `sonatype-work` data directory.

## Step 2 --- Install Windows Service

Open **Administrator CMD**:

``` cmd
cd C:\DevOps\Nexus\nexus-3.85.0-03\bin
```

Run:

``` cmd
install-nexus-service.bat
```

This installs:

``` text
SonatypeNexusRepository
```

## Step 3 --- Start Nexus

``` cmd
nexus.exe //ES//SonatypeNexusRepository
```

Or open:

``` cmd
services.msc
```

Find:

``` text
Sonatype Nexus Repository
```

Set:

``` text
Startup type = Automatic
```

## Step 4 --- Configure Nexus Port

Nexus default HTTP port:

``` text
8081
```

Lab port:

``` text
8051
```

Open:

``` text
C:\DevOps\Nexus\sonatype-work\nexus3\etc\nexus.properties
```

Set:

``` properties
application-port=8051
```

If the property does not exist, add it.

Restart:

``` cmd
nexus.exe //SS//SonatypeNexusRepository
nexus.exe //ES//SonatypeNexusRepository
```

Verify:

``` cmd
netstat -ano | findstr ":8051"
```

Open:

``` text
http://localhost:8051
```

## Step 5 --- Get Initial Admin Password

Nexus stores the initial password here:

``` text
sonatype-work\nexus3\admin.password
```

Example:

``` cmd
type C:\DevOps\Nexus\sonatype-work\nexus3\admin.password
```

Login:

``` text
Username: admin
Password: <value from admin.password>
```

Change the password during initial setup.

## Uninstall Nexus Repository

> The Nexus `bin` folder only ships `install-nexus-service.bat` — there is no separate `uninstall-nexus-service.bat`. Uninstalling the service is done directly with `nexus.exe`.

### Step 1 --- Stop Nexus

``` cmd
nexus.exe //SS//SonatypeNexusRepository
```

### Step 2 --- Uninstall Windows Service

Open **Administrator CMD**:

``` cmd
cd C:\DevOps\Nexus\nexus-3.85.0-03\bin
```

Run:

``` cmd
nexus.exe //DS//SonatypeNexusRepository
```

This removes the `SonatypeNexusRepository` service. Confirm it is gone:

``` cmd
sc query SonatypeNexusRepository
```

### Step 3 --- Delete Nexus Files

``` cmd
rmdir /s /q C:\DevOps\Nexus
```

> This also deletes the `sonatype-work` data directory, including repositories, blob stores, and configuration. Back up anything needed before deleting.

Verify removal:

``` cmd
netstat -ano | findstr ":8051"
```

------------------------------------------------------------------------

# 3️⃣ Install SonarQube

### File

``` text
sonarqube-25.11.0.114957.zip
```

## Step 1 --- Extract

Extract to:

``` text
C:\DevOps\SonarQube
```

Expected:

``` text
C:\DevOps\SonarQube\
└── sonarqube-25.11.0.114957\
```

## Step 2 --- Configure SonarQube Port

SonarQube default web port:

``` text
9000
```

Lab port:

``` text
8052
```

Open:

``` text
C:\DevOps\SonarQube\sonarqube-25.11.0.114957\conf\sonar.properties
```

Add or uncomment:

``` properties
sonar.web.port=8052
```

Save the file.

## Step 3 --- Install Windows Service

Open **Administrator CMD**:

``` cmd
cd C:\DevOps\SonarQube\sonarqube-25.11.0.114957\bin\windows-x86-64
```

Install:

``` cmd
SonarService.bat install
```

## Step 4 --- Start SonarQube

``` cmd
SonarService.bat start
```

Check status:

``` cmd
SonarService.bat status
```

Or:

``` text
services.msc
```

Find:

``` text
SonarQube
```

Set:

``` text
Startup type = Automatic
```

## Step 5 --- Verify

``` cmd
netstat -ano | findstr ":8052"
```

Open:

``` text
http://localhost:8052
```

### SonarQube Logs

``` text
C:\DevOps\SonarQube\sonarqube-25.11.0.114957\logs\
```

Important:

``` text
sonar.log
web.log
ce.log
es.log
```

## Uninstall SonarQube

### Step 1 --- Stop SonarQube

``` cmd
cd C:\DevOps\SonarQube\sonarqube-25.11.0.114957\bin\windows-x86-64
SonarService.bat stop
```

### Step 2 --- Uninstall Windows Service

``` cmd
SonarService.bat uninstall
```

Confirm the service is gone:

``` cmd
sc query SonarQube
```

### Step 3 --- Delete SonarQube Files

``` cmd
rmdir /s /q C:\DevOps\SonarQube
```

> This deletes the SonarQube data directory, including the embedded database, plugins, and logs. Back up anything needed before deleting.

Verify removal:

``` cmd
netstat -ano | findstr ":8052"
```

------------------------------------------------------------------------

# 4️⃣ Install Jenkins

### File

``` text
jenkins.msi
```

## Step 1 --- Run MSI

Right-click:

``` text
jenkins.msi
```

Select:

``` text
Run as administrator
```

The Jenkins Windows MSI installs Jenkins as a Windows service.

## Step 2 --- Select Java

When prompted for Java, select your supported JDK.

Example:

``` text
C:\Program Files\Java\jdk-21
```

## Step 3 --- Configure Jenkins Port

Jenkins default HTTP port:

``` text
8080
```

Lab port:

``` text
8050
```

During the MSI installation, set:

``` text
Port = 8050
```

The installer validates that the selected port is available.

## Step 4 --- Verify Jenkins Service

Open:

``` cmd
services.msc
```

Find:

``` text
Jenkins
```

Set:

``` text
Startup type = Automatic
```

## Step 5 --- Verify Port

``` cmd
netstat -ano | findstr ":8050"
```

Open:

``` text
http://localhost:8050
```

## Step 6 --- Get Initial Admin Password

For the default Jenkins installation location, check:

``` text
C:\Program Files\Jenkins\secrets\initialAdminPassword
```

If you selected another installation directory, check:

``` text
<jenkins-installation-directory>\secrets\initialAdminPassword
```

## Uninstall Jenkins

### Step 1 --- Stop the Jenkins Service

``` cmd
net stop Jenkins
```

### Step 2 --- Uninstall via Control Panel

Open:

``` text
Control Panel
  → Programs
  → Programs and Features
```

Select:

``` text
Jenkins
```

Click:

``` text
Uninstall
```

Alternatively, from an Administrator CMD:

``` cmd
wmic product where "name='Jenkins'" call uninstall
```

### Step 3 --- Remove Leftover Files

If any files remain after uninstall (default install path):

``` cmd
rmdir /s /q "C:\Program Files\Jenkins"
```

> This deletes the Jenkins home directory, including jobs, build history, plugins, and configuration. Back up `JENKINS_HOME` first if needed.

### Step 4 --- Verify Removal

``` cmd
sc query Jenkins
netstat -ano | findstr ":8050"
```

------------------------------------------------------------------------

# 5️⃣ Install Apache Tomcat 9

### File

``` text
apache-tomcat-9.0.111.exe
```

## Step 1 --- Run Installer

Right-click:

``` text
apache-tomcat-9.0.111.exe
```

Select:

``` text
Run as administrator
```

## Step 2 --- Configure Tomcat Service

Enable:

``` text
Service Startup
```

Default Windows service name:

``` text
Tomcat9
```

## Step 3 --- Configure HTTP Port

Tomcat default HTTP port:

``` text
8080
```

Lab port:

``` text
8053
```

During installation:

``` text
HTTP/1.1 Connector Port = 8053
```

Recommended:

``` text
HTTP Port       = 8053
Shutdown Port   = 8005
AJP Port        = 8009
```

> The application HTTP port is part of the `8050–8053` lab range.
> Tomcat's shutdown/AJP ports are separate internal connector ports.

## Step 4 --- Verify Tomcat Service

``` cmd
services.msc
```

Find:

``` text
Apache Tomcat 9.0 Tomcat9
```

Set:

``` text
Startup type = Automatic
```

## Step 5 --- Verify Port

``` cmd
netstat -ano | findstr ":8053"
```

Open:

``` text
http://localhost:8053
```

## Uninstall Apache Tomcat 9

### Step 1 --- Stop the Tomcat Service

``` cmd
net stop Tomcat9
```

### Step 2 --- Run the Uninstaller

Tomcat's Windows installer creates an uninstaller alongside the installation. From the Tomcat installation directory:

``` cmd
Uninstall.exe
```

Or via Control Panel:

``` text
Control Panel
  → Programs
  → Programs and Features
  → Apache Tomcat 9.0 Tomcat9
  → Uninstall
```

### Step 3 --- Remove the Windows Service (if it remains)

``` cmd
sc delete Tomcat9
```

### Step 4 --- Delete Leftover Files

``` cmd
rmdir /s /q C:\DevOps\Tomcat
```

> This deletes deployed applications under `webapps`, along with logs and configuration. Back up anything needed before deleting.

### Step 5 --- Verify Removal

``` cmd
sc query Tomcat9
netstat -ano | findstr ":8053"
```

------------------------------------------------------------------------

# 6️⃣ Install Trivy

### Purpose

**Trivy** is a vulnerability scanner that can scan:

- 🐳 Container images
- 📦 Filesystems
- 📁 Source code repositories
- ☸️ Kubernetes configurations
- 🔐 Dependencies and packages

Trivy is a **CLI tool**, not a Windows web service, so it does not require a port.

The official Windows installation method is to download the `trivy_x.xx.x_windows-64bit.zip` release, extract it, and add the directory containing `trivy.exe` to `PATH`.

## Step 1 — Download Trivy

Download the latest Windows 64-bit ZIP from the official Trivy releases.

Expected file:

```text
trivy_x.xx.x_windows-64bit.zip
```

Extract to:

```text
C:\DevOps\Trivy
```

Expected:

```text
C:\DevOps\Trivy\
└── trivy.exe
```

## Step 2 — Add Trivy to PATH

Open:

```text
System Properties
  → Advanced
  → Environment Variables
```

Edit the **System `Path`** and add:

```text
C:\DevOps\Trivy
```

Close and reopen Command Prompt.

Verify:

```cmd
trivy --version
```

You should see the installed Trivy version.

## Step 3 — Update Vulnerability Database

Run:

```cmd
trivy image --download-db-only
```

Trivy maintains a local vulnerability database that is used during scans.

## Step 4 — Scan a Container Image

If Docker Desktop is installed and running:

```cmd
docker pull nginx:latest
```

Scan the image:

```cmd
trivy image nginx:latest
```

For a CI/CD-friendly scan:

```cmd
trivy image --severity HIGH,CRITICAL nginx:latest
```

Fail the command when HIGH or CRITICAL vulnerabilities are found:

```cmd
trivy image --severity HIGH,CRITICAL --exit-code 1 nginx:latest
```

## Step 5 — Scan a Local Project

From the project directory:

```cmd
trivy fs .
```

Only scan HIGH and CRITICAL vulnerabilities:

```cmd
trivy fs --severity HIGH,CRITICAL .
```

## Step 6 — Generate JSON Report

```cmd
trivy image --format json -o trivy-report.json nginx:latest
```

The report will be created as:

```text
trivy-report.json
```

## Step 7 — Jenkins Integration

A typical Jenkins flow is:

```text
Jenkins :8050
     │
     ├── Checkout
     │
     ├── Build
     │
     ├── Trivy Security Scan
     │       │
     │       ├── HIGH
     │       └── CRITICAL
     │
     ├── SonarQube :8052
     │
     ├── Unit Tests
     │
     ├── Publish Artifact
     │       │
     │       ▼
     │    Nexus :8051
     │
     └── Deploy
             │
             ▼
          Tomcat :8053
```

Example Jenkins Windows batch step:

```cmd
trivy image --severity HIGH,CRITICAL --exit-code 1 %IMAGE_NAME%:%IMAGE_TAG%
```

> Trivy does not need a Windows service or a dedicated HTTP port.

## Uninstall Trivy

Trivy is a standalone CLI tool with no installer and no Windows service, so removal is manual.

### Step 1 — Remove Trivy from PATH

Open:

```text
System Properties
  → Advanced
  → Environment Variables
```

Edit the **System `Path`** and remove:

```text
C:\DevOps\Trivy
```

Close and reopen Command Prompt.

### Step 2 — Delete the Trivy Directory

```cmd
rmdir /s /q C:\DevOps\Trivy
```

### Step 3 — Remove the Vulnerability Database Cache (optional)

Trivy caches its vulnerability database under the user profile:

```cmd
rmdir /s /q %USERPROFILE%\.cache\trivy
```

Verify removal:

```cmd
trivy --version
```

```text
'trivy' is not recognized as an internal or external command
```

------------------------------------------------------------------------

# 7️⃣ Final Port Map

``` text
┌──────────────────────────────────────────────┐
│              WINDOWS DEVOPS LAB              │
├──────────────────────────────────────────────┤
│                                              │
│  🔵 Jenkins       8050   ← Default: 8080    │
│  🟠 Nexus         8051   ← Default: 8081    │
│  🔍 SonarQube     8052   ← Default: 9000    │
│  🐱 Tomcat        8053   ← Default: 8080    │
│                                              │
└──────────────────────────────────────────────┘
```

------------------------------------------------------------------------

# 8️⃣ Verify All Ports

Open **Administrator CMD**:

``` cmd
netstat -ano | findstr ":8050"
netstat -ano | findstr ":8051"
netstat -ano | findstr ":8052"
netstat -ano | findstr ":8053"
```

Expected:

``` text
8050 → Jenkins
8051 → Nexus
8052 → SonarQube
8053 → Tomcat
```

------------------------------------------------------------------------

# 9️⃣ Verify All Windows Services

Open:

``` cmd
services.msc
```

Verify:

``` text
🔵 Jenkins                         Running
🟠 Sonatype Nexus Repository       Running
🔍 SonarQube                       Running
🐱 Apache Tomcat 9.0 Tomcat9      Running
```

Recommended:

``` text
Startup Type = Automatic
```

------------------------------------------------------------------------

# 🔟 Service Management

## Jenkins

``` cmd
net stop Jenkins
net start Jenkins
```

## Nexus

``` cmd
nexus.exe //SS//SonatypeNexusRepository
nexus.exe //ES//SonatypeNexusRepository
```

## SonarQube

``` cmd
SonarService.bat stop
SonarService.bat start
SonarService.bat status
```

## Tomcat

``` cmd
net stop Tomcat9
net start Tomcat9
```

------------------------------------------------------------------------

# 1️⃣1️⃣ Troubleshooting

## Check Java

``` cmd
java -version
javac -version
echo %JAVA_HOME%
where java
```

## Check Ports

``` cmd
netstat -ano | findstr ":8050"
netstat -ano | findstr ":8051"
netstat -ano | findstr ":8052"
netstat -ano | findstr ":8053"
```

Find process by PID:

``` cmd
tasklist | findstr <PID>
```

## Check Windows Services

``` cmd
services.msc
```

## Nexus Logs

``` text
C:\DevOps\Nexus\sonatype-work\nexus3\log\
```

## SonarQube Logs

``` text
C:\DevOps\SonarQube\sonarqube-25.11.0.114957\logs\
```

## Tomcat Logs

``` text
<tomcat-installation>\logs\
```

## Jenkins

Check the Jenkins installation directory and Windows Event Viewer if the
service fails to start.

------------------------------------------------------------------------

# 1️⃣2️⃣ CI/CD Flow

The CI/CD pipeline is orchestrated by Jenkins. SonarQube and Trivy perform quality and security checks before the artifact is published to Nexus.

```text
Developer / Git
      │
      ▼
🔵 Jenkins :8050
      │
      ▼
1. Checkout Code
      │
      ▼
2. Build
      │
      ▼
3. Unit Test
      │
      ▼
4. 🔍 SonarQube :8052
      │
      └── Code Quality / Quality Gate
      │
      ▼
5. 🛡️ Trivy
      │
      └── Vulnerability / Security Scan
      │
      ▼
6. Quality & Security Gates
      │
      ▼
7. Package WAR/JAR
      │
      ▼
8. 🟠 Nexus Repository :8051
      │
      └── Store Versioned Artifact
      │
      ▼
9. 🐱 Apache Tomcat :8053
      │
      └── Deploy Application
      │
      ▼
🚀 Application Running
```

### 🛡️ Where Trivy Fits

For this Windows DevOps lab, Trivy is a **CLI tool invoked by Jenkins**. It is not a Windows web service and does not have a port.

For a Java application, Jenkins can use Trivy to scan the workspace, dependencies, filesystem, or a generated container image.

```text
Build Application
      │
      ▼
SonarQube
      │
      ▼
Trivy
      │
      ├── Filesystem / Dependency Scan
      │
      └── Container Image Scan (if Docker is used)
      │
      ▼
Security Gate
      │
      ▼
Publish Approved Artifact
```

### 🐳 Container Image Scan

If the pipeline also builds a Docker image:

```text
Build Application
      │
      ▼
Docker Build
      │
      ▼
🛡️ Trivy Image Scan
      │
      ├── PASS ──► Push Image
      │
      └── FAIL ──► Stop Pipeline
```

### 📦 Maven Application Flow

```text
Git
 │
 ▼
Jenkins :8050
 │
 ├── Checkout
 │
 ├── Compile
 │
 ├── Unit Test
 │
 ├── SonarQube :8052
 │       └── Code Quality / Quality Gate
 │
 ├── Trivy
 │       └── Vulnerability / Security Scan
 │
 ├── Package
 │       └── application.war
 │
 ├── Nexus :8051
 │       └── Store Versioned Artifact
 │
 └── Tomcat :8053
         └── Deploy Application
```

> **Important:** SonarQube and Trivy are analysis/scanning tools invoked by Jenkins. They are not placed in the application's runtime traffic path. Nexus stores the approved versioned artifact, and Tomcat deploys that artifact.

# 1️⃣4️⃣ Installation Checklist

## ☕ Java

- [ ] Install JDK
- [ ] Configure `JAVA_HOME`
- [ ] Add `%JAVA_HOME%\bin` to `PATH`
- [ ] Verify `java -version`
- [ ] Verify `javac -version`

## 🟠 Nexus

- [ ] Extract Nexus ZIP
- [ ] Install `SonatypeNexusRepository`
- [ ] Configure port `8051`
- [ ] Start service
- [ ] Set startup type to Automatic
- [ ] Verify `http://localhost:8051`
- [ ] Retrieve `admin.password`
- [ ] Change admin password

## 🔍 SonarQube

- [ ] Extract SonarQube ZIP
- [ ] Configure `sonar.web.port=8052`
- [ ] Install `SonarQube` service
- [ ] Start service
- [ ] Set startup type to Automatic
- [ ] Verify `http://localhost:8052`

## 🔵 Jenkins

- [ ] Run `jenkins.msi`
- [ ] Select supported JDK
- [ ] Configure port `8050`
- [ ] Install Jenkins service
- [ ] Set startup type to Automatic
- [ ] Verify `http://localhost:8050`
- [ ] Complete initial setup

## 🐱 Tomcat

- [ ] Run Tomcat installer
- [ ] Enable Windows service
- [ ] Configure HTTP port `8053`
- [ ] Start `Tomcat9`
- [ ] Set startup type to Automatic
- [ ] Verify `http://localhost:8053`

## 🛡️ Trivy

- [ ] Download the Windows 64-bit Trivy ZIP
- [ ] Extract to `C:\DevOps\Trivy`
- [ ] Add `C:\DevOps\Trivy` to `PATH`
- [ ] Verify `trivy --version`
- [ ] Run/update the Trivy vulnerability database
- [ ] Test a filesystem/dependency scan
- [ ] Test a container image scan if Docker is used
- [ ] Configure HIGH/CRITICAL vulnerability handling
- [ ] Integrate Trivy into Jenkins
- [ ] Generate JSON/SARIF/HTML reports if required
- [ ] **Do not configure a web port — Trivy is a CLI tool**

# 🎉 Final Environment

```text
                         WINDOWS DEVOPS LAB
                                │
                                ▼
                         🔵 Jenkins :8050
                                │
                                ▼
                         Build / Unit Test
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼
        🔍 SonarQube :8052                🛡️ Trivy (CLI)
        Code Quality                      Security Scan
                │                               │
                └───────────────┬───────────────┘
                                │
                                ▼
                       Quality / Security Gates
                                │
                                ▼
                       📦 Package WAR/JAR
                                │
                                ▼
                       🟠 Nexus Repository :8051
                                │
                                ▼
                       🐱 Apache Tomcat :8053
                                │
                                ▼
                       🚀 Application Deployed
```

## 🌐 Access URLs

| Tool | URL / Access |
|---|---|
| 🔵 Jenkins | `http://localhost:8050` |
| 🟠 Nexus | `http://localhost:8051` |
| 🔍 SonarQube | `http://localhost:8052` |
| 🛡️ Trivy | **CLI — No Web Port** |
| 🐱 Tomcat | `http://localhost:8053` |

# 📚 Official Documentation

| Component | Official Documentation |
|---|---|
| ☕ Java / OpenJDK | https://openjdk.org/ |
| 🔵 Jenkins — Windows Installation | https://www.jenkins.io/doc/book/installing/windows/ |
| 🔵 Jenkins — Initial Configuration / Port | https://www.jenkins.io/doc/book/installing/initial-settings/ |
| 🟠 Nexus Repository — Installation | https://help.sonatype.com/en/install-nexus-repository.html |
| 🟠 Nexus Repository — Windows Service | https://help.sonatype.com/en/run-as-a-service.html |
| 🔍 SonarQube — Server Installation | https://docs.sonarsource.com/sonarqube-server/server-installation/ |
| 🔍 SonarQube — Windows Service | https://docs.sonarsource.com/sonarqube-server/server-installation/from-zip-file/starting-stopping-server/running-as-a-service |
| 🛡️ Trivy — Installation | https://trivy.dev/docs/latest/getting-started/installation/ |
| 🛡️ Trivy — Getting Started | https://trivy.dev/docs/latest/getting-started/ |
| 🐱 Apache Tomcat 9 — Windows Service | https://tomcat.apache.org/tomcat-9.0-doc/windows-service-howto.html |


## 🚀 Windows DevOps Lab

**Jenkins → Build/Test → SonarQube → Trivy → Nexus → Tomcat**

---

<div align="center">

### Sreekanth K

**Lead DevSecOps and Site Reliability Engineer**

</div>