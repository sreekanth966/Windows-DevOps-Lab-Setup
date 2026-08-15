# 🚀 Windows DevOps Lab

## Java + Jenkins + Nexus Repository + SonarQube + Apache Tomcat

![Jenkins](https://cdn.simpleicons.org/jenkins/D24939)
![SonarQube](https://cdn.simpleicons.org/sonarqube/4E9BCD) ![Sonatype
Nexus](https://cdn.simpleicons.org/sonatype/1A1A1A) ![Apache
Tomcat](https://cdn.simpleicons.org/apachetomcat/F8DC75)

![Windows](https://img.shields.io/badge/OS-Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Java](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Windows
Services](https://img.shields.io/badge/Services-Windows%20Services-5C2D91?style=for-the-badge)

------------------------------------------------------------------------

## 🎯 Purpose

This guide provides a clean installation procedure for a Windows-based
DevOps lab using:

-   ☕ Java JDK
-   🔵 Jenkins
-   🟠 Nexus Repository
-   🔍 SonarQube
-   🐱 Apache Tomcat

All application web ports are intentionally moved away from their
defaults and assigned sequentially from **8050**.

> **Run installation commands from an Administrator Command Prompt
> unless stated otherwise.**

------------------------------------------------------------------------

# 🏗️ Architecture

``` mermaid
flowchart LR
    G[👨‍💻 Developer / Git] --> J[🔵 Jenkins<br/>CI/CD<br/>:8050]
    J --> S[🔍 SonarQube<br/>Code Quality<br/>:8052]
    J --> B[⚙️ Build & Test]
    B --> N[🟠 Nexus Repository<br/>Artifacts<br/>:8051]
    N --> T[🐱 Apache Tomcat<br/>Deployment<br/>:8053]
```

### CI/CD Flow

``` text
Developer
   │
   ▼
  Git
   │
   ▼
Jenkins :8050
   │
   ├──────────────► SonarQube :8052
   │                    │
   │                    └── Code Quality
   │
   ├──────────────► Build / Test
   │
   └──────────────► Nexus :8051
                         │
                         └── Store Artifact
                                  │
                                  ▼
                           Tomcat :8053
                                  │
                                  └── Deploy Application
```

------------------------------------------------------------------------

# 📦 Software

  -----------------------------------------------------------------------------------------------
  Component    File                               Purpose           Default Port         Lab Port
  ------------ ---------------------------------- ------------- ---------------- ----------------
  ☕ Java      JDK                                Runtime                    ---              ---

  🔵 Jenkins   `jenkins.msi`                      CI/CD                   `8080`         **8050**
                                                  Automation                     

  🟠 Nexus     `nexus-3.85.0-03-win-x86_64.zip`   Artifact                `8081`         **8051**
                                                  Repository                     

  🔍 SonarQube `sonarqube-25.11.0.114957.zip`     Code Quality            `9000`         **8052**

  🐱 Tomcat    `apache-tomcat-9.0.111.exe`        Application             `8080`         **8053**
                                                  Server                         
  -----------------------------------------------------------------------------------------------

> **Port plan:** `8050 → 8051 → 8052 → 8053`

### Default vs Lab Ports

  Tool          Default        Lab
  ----------- --------- ----------
  Jenkins        `8080`   **8050**
  Nexus          `8081`   **8051**
  SonarQube      `9000`   **8052**
  Tomcat         `8080`   **8053**

------------------------------------------------------------------------

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
nexus.exe start SonatypeNexusRepository
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
nexus.exe stop SonatypeNexusRepository
nexus.exe start SonatypeNexusRepository
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

------------------------------------------------------------------------

# 6️⃣ Final Port Map

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

# 7️⃣ Verify All Ports

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

# 8️⃣ Verify All Windows Services

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

# 9️⃣ Service Management

## Jenkins

``` cmd
net stop Jenkins
net start Jenkins
```

## Nexus

``` cmd
nexus.exe stop SonatypeNexusRepository
nexus.exe start SonatypeNexusRepository
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

# 🔟 Troubleshooting

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

# 1️⃣1️⃣ CI/CD Flow

``` mermaid
sequenceDiagram
    participant D as 👨‍💻 Developer
    participant G as Git
    participant J as 🔵 Jenkins :8050
    participant S as 🔍 SonarQube :8052
    participant N as 🟠 Nexus :8051
    participant T as 🐱 Tomcat :8053

    D->>G: Push Code
    G->>J: Trigger Pipeline
    J->>S: Code Analysis
    S-->>J: Quality Gate
    J->>J: Build & Test
    J->>N: Publish Artifact
    N-->>J: Artifact Stored
    J->>T: Deploy Application
    T-->>J: Deployment Result
```

------------------------------------------------------------------------

# 1️⃣2️⃣ Example Maven Application Flow

``` text
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
 │       │
 │       └── Quality Gate
 │
 ├── Package
 │       │
 │       └── application.war
 │
 ├── Upload
 │       │
 │       ▼
 │    Nexus :8051
 │
 └── Deploy
         │
         ▼
      Tomcat :8053
```

------------------------------------------------------------------------

# 1️⃣3️⃣ Installation Checklist

## ☕ Java

-   [ ] Install JDK
-   [ ] Configure `JAVA_HOME`
-   [ ] Add `%JAVA_HOME%\bin` to `PATH`
-   [ ] Verify `java -version`
-   [ ] Verify `javac -version`

## 🟠 Nexus

-   [ ] Extract Nexus ZIP
-   [ ] Install `SonatypeNexusRepository`
-   [ ] Configure port `8051`
-   [ ] Start service
-   [ ] Set startup type to Automatic
-   [ ] Verify `http://localhost:8051`
-   [ ] Retrieve `admin.password`
-   [ ] Change admin password

## 🔍 SonarQube

-   [ ] Extract SonarQube ZIP
-   [ ] Configure `sonar.web.port=8052`
-   [ ] Install `SonarQube` service
-   [ ] Start service
-   [ ] Set startup type to Automatic
-   [ ] Verify `http://localhost:8052`

## 🔵 Jenkins

-   [ ] Run `jenkins.msi`
-   [ ] Select supported JDK
-   [ ] Configure port `8050`
-   [ ] Install Jenkins service
-   [ ] Set startup type to Automatic
-   [ ] Verify `http://localhost:8050`
-   [ ] Complete initial setup

## 🐱 Tomcat

-   [ ] Run Tomcat installer
-   [ ] Enable Windows service
-   [ ] Configure HTTP port `8053`
-   [ ] Start `Tomcat9`
-   [ ] Set startup type to Automatic
-   [ ] Verify `http://localhost:8053`

------------------------------------------------------------------------

# 🎉 Final Environment

``` text
                         WINDOWS
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        🔵 Jenkins      🟠 Nexus       🔍 SonarQube
          :8050           :8051            :8052
             │              │
             │              │
             └───────┬──────┘
                     │
                     ▼
              🐱 Apache Tomcat
                    :8053
```

## 🌐 Access URLs

  Tool           URL
  -------------- -------------------------
  🔵 Jenkins     `http://localhost:8050`
  🟠 Nexus       `http://localhost:8051`
  🔍 SonarQube   `http://localhost:8052`
  🐱 Tomcat      `http://localhost:8053`

------------------------------------------------------------------------

# 📚 Official Documentation

-   Jenkins Windows Installation:
    https://www.jenkins.io/doc/book/installing/windows/
-   Jenkins Port Configuration:
    https://www.jenkins.io/doc/book/installing/initial-settings/
-   Nexus Installation:
    https://help.sonatype.com/en/install-nexus-repository.html
-   Nexus Windows Service:
    https://help.sonatype.com/en/run-as-a-service.html
-   SonarQube Windows Service:
    https://docs.sonarsource.com/sonarqube-server/10.8/setup-and-upgrade/operating-the-server
-   Apache Tomcat Windows Service:
    https://tomcat.apache.org/tomcat-9.0-doc/windows-service-howto.html

------------------------------------------------------------------------

## 🚀 Windows DevOps Lab

**Jenkins → SonarQube → Nexus → Tomcat**
