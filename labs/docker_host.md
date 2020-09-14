![Docker](/labs/images/docker.png)
# Scanning a Docker Host with InSpec
  
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


### 3. Scan the Docker Host

  
  
#### Center For Internet Security (CIS) Profile execution
The Centre for Internet Security produces a CIS Docker Foundation Benchmark.  
  
Chef has implemented that benchmark using InSpec. We are now going to obtain that benchmark from Chef Automate and execute it against the Docker Container. 

1. Login to Chef Automate via the terminal:  
`inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com`   
   
  For example:  
  `inspec compliance login --insecure --user=workstation-1 --token AAAA-AAAA-AAAA-AAAAB anthony-a2.chef-demo.com`  
  ```
  Stored configuration for Chef Automate: https://anthony-a2.chef-demo.com/api/v0' with user: 'workstation-1'  
  ```
  
2. Open the Chrome Browser and in Chef Automate, go to the `Compliance` menu tab, then the `Profiles` tab on the left, see that the `CIS Docker Benchmark Profile` profile is available to your `workstation-x` user.  
![Chef Automate Profile](/labs/images/docker2.png)
  
3. Next lets execute that profile against the Docker Container (replace `<x>` with your workstation number) - the tests will take about 4 minutes to run, some will emit a warning as the IAM role I am using does not have all of the required permissions, you can ignore these warnings:   
`inspec exec compliance://workstation-<x>/cis-docker-benchmark -t docker://<contianer_id> --config=reporter.json` 
  
  Your output will be as follows:  
  ```bash
        ×  EC2 Security Group: ID: sg-d11d0xxx Name: default VPC ID: vpc-a966exxx  in us-west-2 is expected not to allow in {:ipv4_range=>"0.0.0.0/0", :port=>22}
        expected `EC2 Security Group: ID: sg-d11d0xxx Name: default VPC ID: vpc-a966exxx  in us-west-2.allow_in?({:ipv4_range=>"0.0.0.0/0", :port=>22})` to return false, got true
      ✔  cis-aws-benchmark-iam-1.10: Ensure IAM password policy prevents password reuse
        ✔  AWS IAM Password Policy is expected to prevent password reuse


    Profile: Amazon Web Services  Resource Pack (inspec-aws)
    Version: 1.0.1
    Target:  aws://us-west-2

        No tests executed.

    Profile Summary: 14 successful controls, 18 control failures, 4 controls skipped
    Test Summary: 489 successful, 122 failures, 4 skipped
  ```
  
4. Look at the scan results in the Chef Automate browser:   
![CIS AWS API Scan Results](/labs/images/aws-cis-run.png)
  


[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
