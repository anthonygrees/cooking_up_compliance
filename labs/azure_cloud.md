![Azure Chef](/labs/images/azure.png)
# Scanning Azure Cloud with InSpec

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
  
### 2. Create Your first InSpec Profile
1. Check to make sure that InSpec can talk to Azure, in the vscode terminal type:  
`inspec detect -t azure://` (if prompted accept the Chef License).  
  
    ```bash
    +---------------------------------------------+
                Chef License Acceptance

    Before you can continue, 1 product license
    must be accepted. View the license at
    https://www.chef.io/end-user-license-agreement/

    License that need accepting:
      * Chef InSpec

    Do you accept the 1 product license (yes/no)?

    > yes

    Persisting 1 product license...
    ✔ 1 product license persisted.

    +---------------------------------------------+

    ────────────────────────────── Platform Details ────────────────────────────── 

    Name:      azure
    Families:  cloud, api
    Release:   azure_mgmt_resources-v0.17.9
    ```
2. Create an InSpec profile to scan azure, in the terminal type:  
`inspec init profile azure --platform=azure`  
`cd azure`  
    
  ```bash
    ─────────────────────────── InSpec Code Generator ─────────────────────────── 

  Creating new profile at /home/ec2-user/inspec-labs/azure
  • Creating file README.md
  • Creating directory controls
  • Creating file controls/example.rb
  • Creating file inspec.yml
 ```
   
Observe the files and directories created in the terminal or the vscode file browser on the left.  
  
3. Scan Azure with your newly created profile:  
`inspec exec . -t azure://`  
  
```bash
    Profile: Azure InSpec Profile (azure)
    Version: 0.1.0
    Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

      ×  azure-virtual-machines-exist-check: Check resource groups to see if any VMs exist. (48 failed)
        ×  Azure Virtual Machines is expected to exist
        expected Azure Virtual Machines to exist
        ×  Azure Virtual Machines is expected to exist
    *** Truncated
        ×  Azure Virtual Machines is expected to exist
        expected Azure Virtual Machines to exist
        ×  Azure Virtual Machines is expected to exist
        expected Azure Virtual Machines to exist


    Profile: Azure Resource Pack (inspec-azure)
    Version: 1.18.2
    Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

        No tests executed.

    Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
    Test Summary: 5 successful, 48 failures, 0 skipped
```
  
### 3. Send you InSpec Results to Chef Automate

1. You will need to create a UUID for your Azure scan, run `uuidgen` in your terminal.   
  
You will see output as follows:
```bash
  [ec2-user@ip-172-31-54-168 aws]$ uuidgen
  c6754ceb-b9a8-4917-a797-c0d0589246d6
```
  
2. Next you need to create a reporter.json file like this in the aws directory, your instructor should have given you the Chef Automate Hostname and Token. Replace <x> with you workstation number. You can create the file by either:  
 - right clicking on the aws directory or
 - in the terminal type `touch reporter.json`
``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://anthony-a2.chef-demo.com/data-collector/v0/",
      "token" : "<Chef Automate Token>",
      "insecure" : true,
      "node_name" : "YourName-azure-api",
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
```inspec exec . -t azure:// --config=reporter.json```  
  
You will see output as follows:  
```bash
  ×  Azure Virtual Machines is expected to exist
     expected Azure Virtual Machines to exist
     ✔  Azure Virtual Machines is expected to exist
     ×  Azure Virtual Machines is expected to exist
     expected Azure Virtual Machines to exist
     ×  Azure Virtual Machines is expected to exist
     expected Azure Virtual Machines to exist


Profile: Azure Resource Pack (inspec-azure)
Version: 1.18.4
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

     No tests executed.

Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
Test Summary: 2 successful, 48 failures, 0 skipped
```
  
![Chef Automate Compliance](/labs/images/azure-compliance.png "Chef Automate Compliance")  
  
  
3a. Click the `"x" Nodes` menu to see the Node list.  
![Chef Automate Compliance](/labs/images/azure-node.png "Chef Automate Compliance")  
  
  
3b. Click on the node that is your scan to see the detail of the scan.  
![Chef Automate Compliance](/labs/images/azure-node-detail.png "Chef Automate Compliance")  
  
  
3c. Notice that we have a Failed Control for your azure-api scanned node.  
Click the `+` next to the `azure-virtual-machines-exist-check` control to reveal the Results:  
![Control Result](/labs/images/azure-control-results.png)  
  
  
3d. The Source - the actual InSpec code that ran to perform the check:  
![Control Source](/labs/images/azure-control-source.png)  

### 4. Using Input Attributes
    
1. InSpec executions can be Input driven. Lets convert our control to be driven by an input file.    
   
Create an `input.yml` file in the `azure` directory.  From the command line run `touch input.yml`.  
  
Open the `input.yml` file and add the following:  
```
loc: 'eastus'
```  
  
Change control in `example.rb` to use the new input.  
``` ruby
title "Input file example"

loc = attribute("loc", value: "", description: "The location to check for")

control "azure-virtual-machines-exist-check" do 
  only_if { loc != "" }
  impact 1.0
  title "Check resource groups to see if any VMs exist."
  azurerm_resource_groups.names.each do |resource_group_name|  
    azurerm_virtual_machines(resource_group: resource_group_name).vm_names.each do |vm_name|
      describe azurerm_virtual_machine(resource_group: resource_group_name, name: vm_name) do
        its('location') { should eq(loc) }
      end
    end
  end
end
```  
  
2. Lets run InSpec again this time specifying the `input.yml` file as an input file:  
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml`  

```bash
Profile: Azure InSpec Profile (azure)
Version: 0.1.0
Target:  azure://b02e675a-cee0-49bd-a056-daa7AAABBBSSS

  ×  azure-virtual-machines-exist-check: Check resource groups to see if any VMs exist. (2 failed)
     ×  'xxxxxx-test' Virtual Machine location is expected to eq "eastus"
     
     expected: "eastus"
          got: "westeurope"
     
     (compared using ==)

     ×  'sa-easdr' Virtual Machine location is expected to eq "eastus"
     
     expected: "eastus"
          got: "westus"
     
     (compared using ==)

Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
Test Summary: 0 successful, 2 failures, 0 skipped
```

### 5.  Using InSpec Waivers
     
1. What if we really do not want to run a control? InSpec has a Waiver capability to allow you to do this.  
   
Under the `azure` directory create a `waiver.yml` file.  Run the following command `touch waiver.yml`
  
Then add the following to it:  
``` yml
azure-virtual-machines-exist-check:
  expiration_date: 2021-02-28
  run: false
  justification: "Security have signed off not doing this check until the end of February 2021"
```  
  
2. Run InSpec with the waiver like this:  
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml --waiver-file waiver.yml `  
  
```bash
Profile: Azure InSpec Profile (azure)
Version: 0.1.0
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

  ↺  azure-virtual-machines-exist-check: Check resource groups to see if any VMs exist.
     ↺  Skipped control due to waiver condition: Security have signed off not doing this check until the end of February 2021


Profile: Azure Resource Pack (inspec-azure)
Version: 1.18.5
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

     No tests executed.

Profile Summary: 0 successful controls, 0 control failures, 1 control skipped
Test Summary: 0 successful, 0 failures, 1 skipped
```
  
3. Look in Chef Automate to see the waiver results, including the reason for the waiver:  
![Control Results](/labs/images/azure-waiver.png)  
  
  
## 6. Writing your own InSpec Profile  
  
1. InSpec ships with many out of the box resources that allow you to easily implement security checks, see [https://www.inspec.io/docs/reference/resources/](https://www.inspec.io/docs/reference/resources/).  
   

2. In this exercise we are going to use the Plural Resource [`azurerm_storage_account_blob_containers`](https://www.inspec.io/docs/reference/resources/azurerm_storage_account_blob_containers/) and its equivalent Singular Resource [`azurerm_storage_account_blob_container`](https://www.inspec.io/docs/reference/resources/azurerm_storage_account_blob_container/) to implement the test.  


Under the `controls` directory delete the `example.rb` file and create a `blob.rb` file, type the following into the file.

``` ruby
title "Blob storage example"

control "azure-check-blob-storage" do 
  impact 1.0
  title "Check to see if some blob storage exists."
  azurerm_storage_account_blob_containers(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef').names.each do |blob|
  # require "pry"; binding.pry
    describe azurerm_storage_account_blob_container(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef',    blob_container_name: blob) do
      it { should exist }
      its('name') { should eq('apprentice-chef-test') }
    end
  end
end
```  
  
3. Execute your InSpec tests:  
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml`  
``` 
Profile: Azure InSpec Profile (azure)
Version: 0.1.0
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

  ×  azure-virtual-machines-exist-check: Check resource groups to see if any VMs exist. (2 failed)
     ×  'ttx-test' Virtual Machine location is expected to eq "eastus"
     
     expected: "eastus"
          got: "westeurope"
     
     (compared using ==)

     ×  'sa-rttss' Virtual Machine location is expected to eq "eastus"
     
     expected: "eastus"
          got: "westus"
     
     (compared using ==)

  ✔  azure-check-blob-storage: Check to see if some blob storage exists.
     ✔  apprentice-chef-test Storage Account is expected to exist
     ✔  apprentice-chef-test Storage Account name is expected to eq "apprentice-chef-test"

Profile Summary: 1 successful control, 1 control failure, 0 controls skipped
Test Summary: 2 successful, 2 failures, 0 skipped
```
  
4. You can also see the resuls in the Chef Automate browser.
  
We can use this information to adjust the software that creates the storage blobs, which could be Chef remediation cookbooks.  
  
### 7. Debugging InSpec profiles using Pry  
  
1. Debugging your test (advanced, skip if you want). As you build out your tests it is useful to be able to debug them, we can use `pry` for that.  
  
Uncomment the `require "pry";binding.pry` line and run your InSpec test again, you sholuld see output like this:  
``` ruby
     4: 
     5: control "azure-check-blob-storage" do 
     6:   impact 1.0
     7:   title "Check to see if some blob storage exists."
     8:   azurerm_storage_account_blob_containers(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef').names.each do |blob|
 =>  9:   require "pry"; binding.pry
    10:     describe azurerm_storage_account_blob_container(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef',    blob_container_name: blob) do
    11:       it { should exist }
    12:       its('name') { should eq('apprentice-chef-test') }
    13:     end
    14:   end
```  
  
2. Execution has been stopped just after the call to the `azurerm_storage_account_blob_containers` resource.  
  
The `names` method returns the storage names, we can see the first name returned by typing:  
 `blob`  
  
``` ruby
[1] pry(#<Inspec::Rule>)> blob
=> "apprentice-chef-test"
```  
  

3. We can go even further and see what our later code will do  
`describe azurerm_storage_account_blob_container(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef',    blob_container_name: blob)`    
  
Try it:  
``` ruby
pry(#<Inspec::Rule>)> describe azurerm_storage_account_blob_container(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef',    blob_container_name: blob)
=> [["describe",
  [#<#<Class:0x0000000006b88d40>:0x00000000076fb538
    @__backend_runner__=Inspec::Backend::Class @transport=Train::Transports::Azure::Connection,
    @__resource_name__="azurerm_storage_account_blob_container",
    @etag="\"0x8D7E78E1B0C2268\"",
    @exists=true,
    @id=
     "/subscriptions/b02e675a-cee0-49bd-a056-daa7ed05bf4e/resourceGroups/apprentice-chef/providers/Microsoft.Storage/storageAccounts/apprenticechef/blobServices/default/containers/apprentice-chef-test",
    @name="apprentice-chef-test",
    @properties=
     #<struct 
      publicAccess="None",
      leaseStatus="Unlocked",
      leaseState="Available",
      lastModifiedTime="
      .
      .
      .
      *** truncated

  ```  
  
4. We can see the instance varaible that we are perfoming our check against:  
``` ruby
      @name="apprentice-chef-test",
```  
  
Press `q` to exit and `exit-program` to exit pry.  
  
  
### 8. Center For Internet (CIS) Profile execution
  
1. The Center for Internet Security produces the CIS Azure Foundation Benchmark. Chef has implemented that benchmark uisng InSpec, which is also certified by CIS.  
  
We are now going to obtain that benchmark from Chef Automate and execute it against the Azure cloud.  
    
2. Login to Chef Automate via the terminal:  
`inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> <Chef Automate Hostname>`    
  
For example:  
`inspec compliance login --insecure --user=workstation-1 --token AAAA-AAAA-AAAA-AAAAB anthony-a2.chef-demo.com/`
  
```bash  
  
Stored configuration for Chef Automate2: https://anthony-a2.chef-demo.com/api/v0' with user: 'workstation-1'
```  
  
3. Open the Chrome Browser and go to the `Compliance` menu, then the `Profiles` tab on the left, see that the `CIS Azure Foundations Benchmark 1.1.0 Level 1` profile is available to your `workstation-x` user.  
  
![Chef Automate Profile](/labs/images/azure-foundation.png)
  
  
4. Next lets execute that profile against the Azure API (replace `<x>` with your workstation number) - the tests will take about 4 minutes to run.  
    
`inspec exec compliance://workstation-<x>/cis-azure-foundations-level1 -t azure:// --config=reporter.json`   
  
Look at the scan results in the Chef Automate browser:  
![CIS Azure API Scan Results](/labs/images/azure-cis-run.png)


[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
