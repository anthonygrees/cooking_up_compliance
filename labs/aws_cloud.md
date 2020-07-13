# Scanning AWS Cloud with InSpec

[Back to the Lab Index](../README.md)

### 1. Configure your workstation
1. Create a remote desktop connection to your Windows workstation, login using `chef` as the username, ask your instructor for the password.

2. Open the remote desktop connection and wait for the background to change to `DevOps Better Together`, the Chrome browser and a Powershell window to open.

3. We are going to use vscode to write our code and a Linux Node to run our InSpec tests on - we will setup vscode to be our one stop environment for this:

i. In the Powershell window type
```bash
code .
```

ii. While in vscode press the F1 function key and start typing
```Remote-SSH: Connect to Host...``` select it and type
`ec2-user@<ask your instructor for the ip address>` 
select it and then select `Linux` (and `continue` if this is the first time you have connected).

iii. Close the Welcome page and click on the `Explorer icon` on the top left, Select `Open Folder` and fill in `/home/ec2-user/inspec-labs` and click `OK`.

iv. From the vscode menu click `Terminal` and then `New Terminal`.
Your setup should now look similar to this:
![Lab Setup Image](/labs/images/vscode-setup.png "Lab Setup")

4. Login to Chef Automate via the Chrome Browser, the browser should be open at the correct URL, if not ask the instructor for the URL.
```
Username = workstation-x
Password = workstation!
```
Replace x with your workstation number given to you by the instructor.

### 2. Create Your first InSpec Profile

1. Check to make sure that InSpec can talk to AWS, in the vscode terminal type:(if prompted accept the Chef License).
`inspec detect -t aws://`

2. Create an InSpec profile to scan aws, in the terminal type:
`inspec init profile aws --platform=aws`
`cd aws`
Observe the files and directories created in the terminal or the vscode file browser on the left.

3. Scan AWS with your newly created profile:
`inspec exec . -t aws://`

### 3. Send your InSpec results to Chef Automate

1. You will need to create a UUID for your AWS scan, run `uuidgen` in your terminal.

2. Next you need to create a `reporter.json` file like this in the `aws` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with you workstation number. You can create the file by right clicking on the `aws` directory or in the terminal with `touch reporter.json`

``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://<Chef Automate Hostname>/data-collector/v0/",
      "token" : "<Chef Automate Token>",
      "insecure" : true,
      "node_name" : "YourName-aws-api",
      "environment" : "cloud-api",
      "node_uuid" : "<uuidgen>"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```

3. Lets run the scan again but now report the results to Chef Automate 
`inspec exec . -t aws:// --config=reporter.json` 

4. The Chrome Browser should already be open, if not open it and ask your instructor for the Chef Automate URL to use. 
   
In Chef Automate Click the `Compliance` menu, observe your node, there may be other nodes from your classmates. In Chef Automate we refer to everything as a node, so in this case our AWS-API scan is our node. 
![Chef Automate Compliance](/labs/images/aws-compliance.png "Chef Automate Compliance") 

Click the `"x" Nodes` menu to see the Node list. 
![Chef Automate Compliance](/labs/images/aws-node.png "Chef Automate Compliance") 

Click on the node that is your scan to see the detail of the scan. 
![Chef Automate Compliance](/labs/images/aws-node-detail.png "Chef Automate Compliance") 
Notice that we have two Passed Controls and one skipped control for your aws-api scanned node. 
Click the `+` next to the `aws-vpcs-check` control to reveal the Results: 
![Control Result](/labs/images/aws-control-results.png) 


Make a note of one of the VPC id's we are going to use that later. 
The Source - the actual InSpec code that ran to perform the check: 
![Control Source](/labs/images/aws-control-source.png) 

### 4. Using InSpec Attributes

1. Lets investigate why one of the controls was skipped. Look inside the `controls` directory and open up `example.rb`. We need to supply an input `aws_vpc_id` for the first control to run because of this line: 
`only_if { aws_vpc_id != "" }` 

2. Open the `attributes.yml` file and uncomment the `aws_vpc_id: 'vpc-xxxxxxx'` line and replace xxxxxxx with the vpc id you noted earlier. Save the changes. 

3. Lets run InSpec again this time specifying the `attributes.yml` file as an input file: 
`inspec exec . -t aws:// --config=reporter.json --input-file=attributes.yml` 

4. Take a look at the results in Chef Automate, you should now see all three controls passing. 
![Control Results](/labs/images/aws-controls-passing.png) 

### 5. Using InSpec Waivers

1. What if we really do not want to run one of the controls? InSpec has a Waiver capability to allow you to do this. 
Under the `aws` directory create a `waiver.yml` file and add the following to it: 
``` yml
aws-vpcs-check:
  expiration_date: 2021-02-28
  run: false
  justification: "Security have signed off not doing this check until the end of February 2021"
```

2. Run InSpec with the waiver like this: 
`inspec exec . -t aws:// --config=reporter.json --input-file=attributes.yml --waiver-file waiver.yml ` 

3. Look in Chef Automate to see the waiver results, including the reason for the waiver: 
![Control Results](/labs/images/aws-waiver.png) 

### 6. Writing your own InSpec Profile
1. InSpec ships with many out of the box resources that allow you to easily implement security checks, see [https://www.inspec.io/docs/reference/resources/](https://www.inspec.io/docs/reference/resources/). 
  
In this exercise we are going to explore the Plural Resource [`aws_iam_roles`](https://www.inspec.io/docs/reference/resources/aws_iam_roles/) and its equivalent Singular Resource [`aws_iam_role`](https://www.inspec.io/docs/reference/resources/aws_iam_role/) to implement the test. 

2. The test will allow us to check that all AWS IAM Roles defined have an appropriate max session duration. 
Under the `controls` directory delete the `example.rb` file and create a `roles.rb` file, type the following into the file.

``` ruby
control "aws-role-session-check" do
  impact 1.0
  title "Check in all the iam roles that the session timeout is large enough"
  aws_iam_roles.role_names.each do |role|
    # require "pry";binding.pry
    describe aws_iam_role(role) do
        it { should exist }
        its('max_session_duration') { should be >= (60*120) }
    end
  end
end
```

3. Execute your InSpec tests (it will take about a minute to run): 
`inspec exec . -t aws:// --config=reporter.json --input-file=attributes.yml` 

``` 
Profile: AWS InSpec Profile (aws)
Version: 0.1.0
Target:  aws://eu-west-1

  ×  aws-role-session-check: Check in all the iam roles that the session timeout is large enough (130 failed)
     ✔  AWS IAM Role ak_logger_agent is expected to exist
     ×  AWS IAM Role ak_logger_agent max_session_duration is expected to be >= 7200
     expected: >= 7200
          got:    3600
.
.
. Truncated
.
     ✔  AWS IAM Role apprentice-role-1019d32e is expected to exist
     ×  AWS IAM Role apprentice-role-1019d32e max_session_duration is expected to be >= 7200
     expected: >= 7200
          got:    3600


Profile: Amazon Web Services  Resource Pack (inspec-aws)
Version: 1.8.2
Target:  aws://eu-west-1

     No tests executed.

Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
Test Summary: 132 successful, 130 failures, 0 skipped
```

You can also see the resuls in the Chef Automate browser.

We can see at this point that the session timeouts are set too low - now would be a good time to adjust the software that creates the IAM Roles, which could be Chef remediation cookbooks. 

### 7. Debugging an InSpec Profile
(Advanced - skip if you want).

1.  Debugging your test: As you build out your tests it is useful to be able to debug them, we can use `pry` for that. Uncomment the `require "pry";binding.pry` line and run your InSpec test again, you sholuld see output like this: 
``` ruby
     1: control "aws-role-session-check" do
     2:   impact 1.0
     3:   title "Check in all the iam roles that the session timeout is large enough"
     4:   aws_iam_roles.role_names.each do |role|
 =>  5:     require "pry";binding.pry
     6:     describe aws_iam_role(role) do
     7:         it { should exist }
     8:         its('max_session_duration') { should be >= (60*120) }
     9:     end
    10:   end
```

2. Execution has been stopped just after the call to the `aws_iam_roles` resource. The `role_names` method returns an array of role names, we can see the first name returned by typing `role`. 
``` ruby
[1] pry(#<Inspec::Rule>)> role
=> "ak_logger_agent"
```

3. We can go even further and see what our later `describe aws_iam_role(role)` code will do, try it: 
``` ruby
pry(#<Inspec::Rule>)> describe aws_iam_role(role)
=> [["describe",
  [#<#<Class:0x0000000006857838>:0x00000000042ee8a8
    @__backend_runner__=Inspec::Backend::Class @transport=TrainPlugins::Aws::Connection,
    @__resource_name__="aws_iam_role",
    @arn="arn:aws:iam::496323866215:role/ak_logger_agent",
    @assume_role_policy_document=
     "%7B%22Version%22%3A%222012-10-17%22%2C%22Statement%22%3A%5B%7B%22Effect%22%3A%22Allow%22%2C%22Principal%22%3A%7B%22Service%22%3A%22ec2.amazonaws.com%22%7D%2C%22Action%22%3A%22sts%3AAssumeRole%22%7D%5D%7D",
    @attached_policies_arns=["arn:aws:iam::aws:policy/AmazonEC2FullAccess", "arn:aws:iam::aws:policy/CloudWatchFullAccess"],
    @attached_policies_names=["AmazonEC2FullAccess", "CloudWatchFullAccess"],
    @aws=
     #<#<Class:0x00000000045edf18>::AwsConnection:0x0000000004317168
      @cache={:"Aws::IAM::Client"=>#<Aws::IAM::Client>},
      @client_args={}>,
    @create_date=2018-08-03 20:20:48 UTC,
    @description="Allows EC2 instances to call AWS services on your behalf.",
    @max_session_duration=3600,
    @opts={:role_name=>"ak_logger_agent"},
    @path="/",
    @permissions_boundary_arn=nil,
    @permissions_boundary_type=nil,
    @resource_exception_message=nil,
    @resource_failed=false,
    @resource_skipped=false,
    @role_id="AROAIU4PO3IV7CV5XVFTY",
    @role_name="ak_logger_agent",
    @supports=nil>],
  nil]]
  ```
  
4. We can see the instance varaible that we are perfoming our check against: 
``` ruby
      @max_session_duration=3600,
```

5. Press `q` to exit and `exit-program` to exit pry.

## 4. Center For Internet Security (CIS) Profile execution
The Centre for Internet Security produces a CIS AWS Foundation Benchmark, Chef has implemented that benchmark uisng InSpec, it is fully accreditied by CIS. We are now going to obtain that benchmark from Chef Automate and execute it against the AWS cloud. 

1. Login to Chef Automate via the terminal: 
`inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com` 
For example: 
`inspec compliance login --insecure --user=workstation-1 --token AAAA-AAAA-AAAA-AAAAB anthony-a2.chef-demo.com`
```
Stored configuration for Chef Automate: https://anthony-a2.chef-demo.com/api/v0' with user: 'workstation-1'
```

2. Open the Chrome Browser and go to the `Compliance` menu tab, then the `Profiles` tab on the left, see that the `CIS Amazon Web Services Foundation Benchmark Level 1` profile is available to your `workstation-x` user.
![Chef Automate Profile](/labs/images/aws-foundation.png)

3. Next lets execute that profile against the AWS API (replace `<x>` with your workstation number) - the tests will take about 4 minutes to run, some will emit a warning as the IAM role I am using does not have all of the required permissions, you can ignore these warnings: 
`inspec exec compliance://workstation-<x>/cis-aws-benchmark-level1 -t aws:// --config=reporter.json` 

4. Look at the scan results in the Chef Automate browser: 
![CIS AWS API Scan Results](/labs/images/aws-cis-run.png)


[Back to the Lab Index](../README.md)
