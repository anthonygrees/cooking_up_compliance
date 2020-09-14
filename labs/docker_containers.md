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

### 2. Scan a Docker Container Directly
  
#### Run a Container
1. Docker is installed on the Linux node and is ready to go.  
  
2. Lets create a container that we can scan. I have chosen the Docker delivered "docker/getting-started" container for this exercise:  
  
```docker run -dp 80:80 docker/getting-started```  
  
3. You can verifiy that it is running by going to your browser:  
```http://<public ip of your Linux Node>/``` 
  
4. Now lets find out its container ID to use for InSpec scanning:  
  
```docker ps -l```
  
5. You will see the following output:  
```bash
CONTAINER ID        IMAGE                    COMMAND                  CREATED              STATUS              PORTS                NAMES
bc3e5e37cb12        docker/getting-started   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   optimistic_mclean
```
  
#### Create a Docker InSpec Profile
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
  
#### Send Your InSpec Results To Chef Automate

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
  
  
  
  
#### Scan a Postgres Docker Container
  
```bash
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```


[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)