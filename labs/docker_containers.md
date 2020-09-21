![Docker](/labs/images/docker.png)
# Scanning a Docker Container with InSpec
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
### 1. Configure Your Workstation
1. Create a remote desktop connection to your Windows workstation  
  - login using `chef` as the username
  - use the password provided by your Chef Instructor

2. Open the remote desktop connection and wait for the background to change to `DevOps Better Together`, the Chrome browser and a Powershell window to open.

3. We are going to use VS Code to write our code and a Linux Node to run our InSpec tests on - we will setup VS Code to be our one stop environment for this:

    i. In the Powershell window type  
    ```bash
    code .
    ```  
    This will open VS Code for you.

    ii. While in VS Code press the F1 function key and start typing  
    ```Remote-SSH: Connect to Host...``` select it  
    ![Lab Setup Image](/labs/images/vscode-setup-remote.png "Lab Setup")
    Now type  
    `ec2-user@<ip address as provided by your instructor>`  
    select it and then select `Linux`   
    And `continue` if this is the first time you have connected).  

    iii. Close the Welcome page and click on the `Explorer icon` on the top left, Select `Open Folder` and fill in `/home/ec2-user/inspec-labs` and click `OK`.  

    iv. From the VS Code menu click `View` and then `Terminal`.  
    Your setup should now look similar to this:  
    ![Lab Setup Image](/labs/images/vscode-setup.png "Lab Setup")

4. Login to Chef Automate via the Chrome Browser, the browser should be open at the correct URL, if not, the URL is https://anthony-a2.chef-demo.com  
```
Username = workstation-x
Password = workstation!
```
Replace x with your workstation number given to you by the instructor.  
![Lab Setup Image](/labs/images/automate.png "Automate")

### 2. Some Handy Docker Commands
  
#### 2a. Running Containers
  
Show runnning containers.   
```bash
docker ps //To show only running containers.
docker ps -a //To show all containers.
docker ps -l //To show the latest created container.
docker ps -n=-1 //To show n last created containers.
docker ps -s //To display total file sizes.
```
  
#### 2b. Docker Images
Show Docker Images.  
```bash
docker image list
```
  
#### 2c. Purge Containers
Purging All Unused or Dangling Images, Containers, Volumes, and Networks
Docker provides a single command that will clean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container):  
  
```docker system prune```
  
To additionally remove any stopped containers and all unused images (not just dangling images), add the -a flag to the command:
```bash
docker system prune -a
```
  
#### 2d. Remove Containers
Stop and remove all docker containers and images
 - List all containers (only IDs) `docker ps -aq.`
 - Stop all running containers. `docker stop $(docker ps -aq)`
 - Remove all containers. `docker rm $(docker ps -aq)`
 - Remove all images. `docker rmi $(docker images -q)`
  
  
### 3. Scan a Docker Container Directly
  
#### 3a. Run a Container
  
1. Docker is installed on the Linux node and is ready to go.  
  
2. Lets create a container that we can scan. I have chosen the Docker delivered "docker/getting-started" container for this exercise:  
  
```docker run -dp 80:80 docker/getting-started```  
  
  
```bash
[ec2-user@ip-172-31-54-32 inspec-labs]$ docker run -dp 80:80 docker/getting-started
Unable to find image 'docker/getting-started:latest' locally
latest: Pulling from docker/getting-started
cbdbe7a5bc2a: Pull complete 
85434292d1cb: Pull complete 
75fcb1e58684: Pull complete 
2a8fe5451faf: Pull complete 
42ceeab04dd4: Pull complete 
bdd639f50516: Pull complete 
c446f16e1123: Pull complete 
Digest: sha256:79d5eae6e7b1dec2e911923e463240984dad111a620d5628a5b95e036438b2df
Status: Downloaded newer image for docker/getting-started:latest
1184f3bfc817a19e8744d5b487adb526d6210c3e9a3ca0299335ec64a1561111


[ec2-user@ip-172-31-54-32 inspec-labs]$ docker ps -l
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                NAMES
1184f3bfc817        docker/getting-started   "/docker-entrypoint.…"   11 seconds ago      Up 11 seconds       0.0.0.0:80->80/tcp   affectionate_lovelace
```
  
3. You can verifiy that it is running by going to your browser:  
```http://<public ip of your Linux Node>/``` 
  
4. Now lets find out its container ID to use for InSpec scanning:  
  
```docker ps -l```
  
5. You will see the following output:  
```bash
CONTAINER ID        IMAGE                    COMMAND                  CREATED              STATUS              PORTS                NAMES
bc3e5e37cb12        docker/getting-started   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   optimistic_mclean
```
  
#### 3b. Detect the platform of the container with InSpec
  
1. You can determine the platform with the `inspec detect` command:
  
```bash
inspec detect -t docker://<CONTAINER ID>
```

```
[ec2-user@ip-172-31-54-32 inspec-labs]$ inspec detect -t docker://1184f3bfc817

 ────────────────────────────── Platform Details ────────────────────────────── 

Name:      alpine
Families:  linux, unix, os
Release:   3.11.6
Arch:      x86_64
```
  
#### 3c. Run the DevSec Linux Baseline againnst the container
  
1. You can run profiles from Chef Automate
  
Login to Chef Automate via the terminal:  
```bash
inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com
```
  
Now execute the profile from Chef Automate.  
```bash
inspec exec compliance://workstation-1/linux-baseline -t docker://<CONTAINER ID> --config=reporter.json
```
  
Your putput will look like this:
```bash
Profile: DevSec Linux Security Baseline (linux-baseline)
Version: 2.5.0
Target:  docker://1184f3bfc817a19e8744d5b487adb526d6210c3e9a3ca0299335ec64a1561111

  ✔  os-01: Trusted hosts login
     ✔  File /etc/hosts.equiv is expected not to exist
  ×  os-02: Check owner and permissions for /etc/shadow (1 failed)
     ✔  File /etc/shadow is expected to exist
     ✔  File /etc/shadow is expected to be file
     ✔  File /etc/shadow is expected to be owned by "root"
     ✔  File /etc/shadow is expected not to be executable
     ✔  File /etc/shadow is expected not to be readable by other
     ✔  File /etc/shadow group is expected to eq "shadow"
     ✔  File /etc/shadow is expected to be writable by owner
     ✔  File /etc/shadow is expected to be readable by owner
     ×  File /etc/shadow is expected not to be readable by group
     expected File /etc/shadow not to be readable by group
  ✔  os-03: Check owner and permissions for /etc/passwd
     ✔  File /etc/passwd is expected to exist
     ✔  File /etc/passwd is expected to be file
     ✔  File /etc/passwd is expected to be owned by "root"
     ✔  File /etc/passwd is expected not to be executable
     ✔  File /etc/passwd is expected to be writable by owner
     ✔  File /etc/passwd is expected not to be writable by group
     ✔  File /etc/passwd is expected not to be writable by other
     ✔  File /etc/passwd is expected to be readable by owner
     ✔  File /etc/passwd is expected to be readable by group
     ✔  File /etc/passwd is expected to be readable by other
     ✔  File /etc/passwd group is expected to eq "root"
  ✔  os-03b: Check passwords hashes in /etc/passwd
     ✔  /etc/passwd passwords is expected to be in "x" and "*"
  ✔  os-04: Dot in PATH variable
     ✔  Environment variable PATH split is expected not to include ""
     ✔  Environment variable PATH split is expected not to include "."
  ↺  os-05: Check login.defs (7 failed) (1 skipped)
     ×  File /etc/login.defs is expected to exist
     expected File /etc/login.defs to exist
     ×  File /etc/login.defs is expected to be file
     expected `File /etc/login.defs.file?` to return true, got false
     ×  File /etc/login.defs is expected to be owned by "root"
     expected `File /etc/login.defs.owned_by?("root")` to return true, got false
     ✔  File /etc/login.defs is expected not to be executable
     ×  File /etc/login.defs is expected to be readable by owner
     expected File /etc/login.defs to be readable by owner
     ×  File /etc/login.defs is expected to be readable by group
     expected File /etc/login.defs to be readable by group
     ×  File /etc/login.defs is expected to be readable by other
     expected File /etc/login.defs to be readable by other
     ×  File /etc/login.defs group is expected to eq "root"
     
     expected: "root"
          got: nil
     
     (compared using ==)

     ↺  Can't find file: /etc/login.defs
  ↺  os-05b: Check login.defs - RedHat specific
     ↺  Skipped control due to only_if condition.

Profile Summary: 15 successful controls, 2 control failures, 38 controls skipped
Test Summary: 39 successful, 8 failures, 39 skippe
```

  
### 4. Create a Docker InSpec Profile
  
1. InSpec allows you to scan directly against against a Docker container. Lets first create a profile to use for our scan tests:  
  
```bash
inspec init profile docker
cd docker
```  
(accept the license if prompted). 
  
2. Open the `controls/example.rb` file to see the InSpec tests that we are going to run. These are some out of the box tests to test the `/tmp` diretory.  
  
Run the InSpec profile against our Docker container (use your container id):  
  
```inspec exec . -t docker://<contianer_id>```
  
For example.  
```inspec exec . -t docker://bc3e5e37cb12```
  
Your output will look like this:  
```bash
Profile: InSpec Profile (docker)
Version: 0.1.0
Target:  docker://b6d89059de6439b02d6c84a3140109ce688924fb0d31f2e3ef5271d78bc98031

  ✔  tmp-1.0: Create /tmp directory
     ✔  File /tmp is expected to be directory

  File /tmp
     ✔  is expected to be directory

Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
```
You can create your own tests using the out of the box InSpec Resources detailed here or make use of one of the CIS InSpec profiles suitable for the operating system that your containers are sharing.  
  
#### 4a. Send Your InSpec Results To Chef Automate

1. You will need to create a UUID for your Docker scan, run `uuidgen` in your terminal.    
     
  You will see output as follows:
  ```bash
    [ec2-user@ip-172-31-54-168 aws]$ uuidgen
    c6754ceb-b9a8-4917-a797-c0d0589246d6
      
```

2. Next you need to create a `reporter.json` file like this in the `docker` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with you workstation number. You can create the file by either:  
   - right clicking on the `docker` directory or   
   - in the terminal type `touch reporter.json`  
  
``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://anthony-a2.chef-demo.com/data-collector/v0/",
      "token" : "AAAA-AAAA-AAAA-AAAAB",
      "insecure" : true,
      "node_name" : "YourName-docker-container",
      "environment" : "container",
      "node_uuid" : "<uuidgen>"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```
  
3. Lets run the scan again but now report the results to Chef Automate  
`inspec exec . -t docker://<contianer_id> --config=reporter.json`  
  
  You will see output as follows:  
  ```bash
[docker]$ inspec exec . -t docker://bc3e5e37cb12 --config=reporter.json

Profile: InSpec Profile (docker)
Version: 0.1.0
Target:  docker://bc3e5e37cb1231d8b7486838c8b69e5f33724976c2e25e52a39b87963377dbed

  ✔  tmp-1.0: Create /tmp directory
     ✔  File /tmp is expected to be directory

  File /tmp
     ✔  is expected to be directory

Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
  ```
  
![Docker](/labs/images/docker1.png "Docker") 
  
  
  
  
### 5. Scan a Postgres Docker Container
  
1. Run a Postgres Container
```bash
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```
  
You output will look like this:  
```bash
[ec2-user@ip-172-31-54-32 inspec-labs]$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
Unable to find image 'postgres:latest' locally
latest: Pulling from library/postgres
d121f8d1c412: Pull complete 
9f045f1653de: Pull complete 
fa0c0f0a5534: Pull complete 
54e26c2eb3f1: Pull complete 
cede939c738e: Pull complete 
69f99b2ba105: Pull complete 
218ae2bec541: Pull complete 
70a48a74e7cf: Pull complete 
a5a0d51f9154: Pull complete 
73e23a8be3f2: Pull complete 
770413f2d2da: Pull complete 
8ff552bf08c6: Pull complete 
a367abe73a12: Pull complete 
007c3a47b37e: Pull complete 
Digest: sha256:91462e8207eadf217fe72822163277d189215b1f3792719f29352de5beb0ad53
Status: Downloaded newer image for postgres:latest
4295fd1a7ddde8a03d664988f2dc272aad2d6d962dc9526bcea870854b851174
```
  
2. Create a ```UUID```
  
Run the command ```uuidgen```
```bash
[ec2-user@ip-172-31-54-32 inspec-labs]$ uuidgen
867fe288-e311-4844-887a-b47c9ab42da8
```
  
3. Create a `reporter.json` for Chef Automate
  
Next you need to create a reporter.json file like this in the docker directory, your instructor should have given you the Chef Automate Hostname and Token. Replace <x> with you workstation number. You can create the file by either:  
 - right clicking on the docker directory or.  
 - in the terminal type touch reporter.json.  
  
```json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://anthony-a2.chef-demo.com/data-collector/v0/",
      "token" : "AAAA-AAAA-AAAA-AAAAB",
      "insecure" : true,
      "node_name" : "YourName-postgres-container",
      "environment" : "container",
      "node_uuid" : "<uuidgen>"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```
  
4. Get the Container ID
  
```bash
docker ps -l
```
  
You output will look like this:  
```bash
[ec2-user@ip-172-31-54-32 inspec-labs]$ docker ps -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4295fd1a7ddd        postgres            "docker-entrypoint.s…"   8 seconds ago       Up 6 seconds        5432/tcp            some-postgres
```
  
5. Detect the Container platform
  
```bash
inspec detect -t docker://
```
  
You output will look like this:  
```bash
[ec2-user@ip-172-31-54-32 inspec-labs]$ inspec detect -t docker://4295fd1a7ddd

 ────────────────────────────── Platform Details ────────────────────────────── 

Name:      debian
Families:  debian, linux, unix, os
Release:   10.5
Arch:      x86_64
```
  
6. You can run profiles from Chef Automate
  
Login to Chef Automate via the terminal:  
```bash
inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com
```
  
Now execute the profile from Chef Automate.  
```bash
inspec exec compliance://workstation-1/linux-baseline -t docker://<CONTAINER ID> --config=reporter.json
```
  
Your putput will look like this:
```bash
Profile: DevSec Linux Security Baseline (linux-baseline)
Version: 2.2.2
Target:  docker://4295fd1a7ddde8a03d664988f2dc272aad2d6d962dc9526bcea870854b851174

  ✔  os-01: Trusted hosts login
     ✔  File /etc/hosts.equiv is expected not to exist
  ✔  os-02: Check owner and permissions for /etc/shadow
     ✔  File /etc/shadow is expected to exist
     ✔  File /etc/shadow is expected to be file
  ↺  sysctl-33: CPU No execution Flag or Kernel ExecShield
     ↺  Skipped control due to only_if condition.


Profile Summary: 15 successful controls, 1 control failure, 38 controls skipped
Test Summary: 53 successful, 3 failures, 38 skipped
```
  
7. Run the Postgres Baseline
  
Login to Chef Automate via the terminal:  
```bash
inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com
```
  
Now execute the profile from Chef Automate.  
```bash
inspec exec compliance://workstation-1/postgres-baseline -t docker://<CONTAINER ID> --config=reporter.json
```
  
Your putput will look like this:
```bash
Profile: Hardening Framework Postgres Hardening Test Suite (postgres-baseline)
Version: 2.0.3
Target:  docker://4295fd1a7ddde8a03d664988f2dc272aad2d6d962dc9526bcea870854b851174

  ×  postgres-02: Use stable postgresql version (1 failed)
     ×  Command: `psql -V` stdout is expected to match /9.[1-5]/
     expected "psql (PostgreSQL) 12.4 (Debian 12.4-1.pgdg100+1)\n" to match /9.[1-5]/
     Diff:
     @@ -1 +1 @@
     -/9.[1-5]/
     +psql (PostgreSQL) 12.4 (Debian 12.4-1.pgdg100+1)

     ✔  Command: `psql -V` stdout is expected not to match /RC/
     ✔  Command: `psql -V` stdout is expected not to match /DEVEL/
     ✔  Command: `psql -V` stdout is expected not to match /BETA/
  ×  postgres-03: Run one postgresql instance per operating system
     ×  Processes postgres list.length 
     DEPRECATION: The processes `list` property is deprecated. Please use `entries` instead. This property was removed in InSpec 4.0. (used at postgres-baseline-2.0.3/controls/postgres_spec.rb:134)
  ✔  postgres-04: Only "c" and "internal" should be used as non-trusted procedural languages
     ✔  PostgreSQL query: SELECT count (*) FROM pg_language WHERE lanpltrusted = 'f' AND lanname!='internal' AND lanname!='c'; output is expected to eq "0"

  ↺  postgres-15: Enable logging functions
     ↺  Can't find file: /etc/postgresql/12.4-1/postgresql.conf


Profile Summary: 4 successful controls, 6 control failures, 4 controls skipped
Test Summary: 31 successful, 20 failures, 4 skipped
```

[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
