# Scanning Azure Cloud with InSpec

[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)

## 1. Configure your workstation
1. Create a remote desktop connection to your Windows workstation, login using `chef` as the username, ask your instructor for the password.<br />
2. Open the remote desktop connection and wait for the background to change to `DevOps Better Together`, the Chrome browser and a Powershell window to open.
3. We are going to use vscode to write our code and a Linux Node to run our InSpec tests on - we will setup vscode to be our one stop environment for this:<br />
   i. In the Powershell window type<br />
   `code .`<br />
   ii. While in vscode press the F1 function key and start typing<br />
   `Remote-SSH: Connect to Host...`, select it and type<br />
   `ec2-user@<ask your instructor for the ip address>`, select it and then select `Linux` (and `continue` if this is the first time you have connected).<br />
   iii. Close the Welcome page and click on the `Explorer icon` on the top left, Select `Open Folder` and fill in `/home/ec2-user/inspec-labs` and click `OK`. Close the Welome page.</br >
   iv. From the vscode menu click `Terminal` and then `New Terminal`.<br />
   Your setup should now look similar to this:<br />
   ![Lab Setup Image](/labs/images/vscode-setup.png "Lab Setup")
4. Login to Chef Automate via the Chrome Browser, the browser should be open at the correct URL, if not ask the instructor for the URL.<br />
```
Username = workstation-x
Password = workstation!
```
## 2. Explore Your first InSpec Profile
1. Check to make sure that InSpec can talk to Azure, in the vscode terminal type: <br />
`inspec detect -t azure://` (if prompted accept the Chef License).
2. Create an InSpec profile to scan azure, in the terminal type:<br />
`inspec init profile azure --platform=azure`<br />
`cd azure`</br >
Observe the files and directories created in the terminal or the vscode file browser on the left.<br />
3. Scan Azure with your newly created profile:<br />
`inspec exec . -t azure://`<br />
4. Send the InSpec scan results to Chef Automate. First you need to create a `reporter.json` file like this in the `azure` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with your workstation number. You can create the file by right clicking on the `azure` directory or in the terminal with `touch reporter.json`<br />
You will also need to create a UUID for your Azure scan, run `uuidgen` in your terminal.
``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://<Chef Automate Hostname>/data-collector/v0/",
      "token" : "<Chef Automate Token>",
      "insecure" : true,
      "node_name" : "workstation-<x>-azure-api",
      "environment" : "cloud-api",
      "node_uuid" : "<uuidgen>"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```
Lets run the scan again but now report the results to Chef Automate<br />
`inspec exec . -t azure:// --config=reporter.json`<br />
The Chrome Browser should already be open, if not open it and ask your instructor for the Chef Automate URL to use.<br />
In Chef Automate Click the `Compliance` menu, observe your node, there may be other nodes from your classmates. In Chef Automate we refer to everything as a node, so in this case our Azure-API scan is our node.<br />
![Chef Automate Compliance](/labs/images/azure-compliance.png "Chef Automate Compliance")<br />
Click the `"x" Nodes` menu to see the Node list.<br />
![Chef Automate Compliance](/labs/images/azure-node.png "Chef Automate Compliance")<br />
Click on the node that is your scan to see the detail of the scan.<br />
![Chef Automate Compliance](/labs/images/azure-node-detail.png "Chef Automate Compliance")<br />
Notice that we have a Failed Control for your azure-api scanned node.<br />
Click the `+` next to the `azure-virtual-machines-exist-check` control to reveal the Results:<br />
![Control Result](/labs/images/azure-control-results.png)<br />
The Source - the actual InSpec code that ran to perform the check:<br />
![Control Source](/labs/images/azure-control-source.png)<br />
5. InSpec executions can be Input driven. Lets convert our control to be driven by an input file.<br />
Create an `input.yml` file in the `azure` directory.<br />
Open the `input.yml` file and add the following<br />
```
loc: 'eastus'
```
Change control in `example.rb` to use the new input.<br />
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
Lets run InSpec again this time specifying the `input.yml` file as an input file:<br />
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml`<br />
6. What if we really do not want to run a control? InSpec has a Waiver capability to allow you to do this.<br />
Under the `azure` directory create a `waiver.yml` file and add the following to it:<br />
``` yml
azure-virtual-machines-exist-check:
  expiration_date: 2021-02-28
  run: false
  justification: "Security have signed off not doing this check until the end of February 2021"
```
Run InSpec with the waiver like this:<br />
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml --waiver-file waiver.yml `<br />
Look in Chef Automate to see the waiver results, including the reason for the waiver:<br />
![Control Results](/labs/images/azure-waiver.png)<br />
## 3. Writing your own InSpec Profile
1. InSpec ships with many out of the box resources that allow you to easily implement security checks, see [https://www.inspec.io/docs/reference/resources/](https://www.inspec.io/docs/reference/resources/).<br />
 In this exercise we are going to use the Plural Resource [`azurerm_storage_account_blob_containers`](https://www.inspec.io/docs/reference/resources/azurerm_storage_account_blob_containers/) and its equivalent Singular Resource [`azurerm_storage_account_blob_container`](https://www.inspec.io/docs/reference/resources/azurerm_storage_account_blob_container/) to implement the test.<br />
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
Execute your InSpec tests:<br />
`inspec exec . -t azure:// --config=reporter.json --input-file=input.yml`<br />
``` 
Profile: Azure InSpec Profile (azure)
Version: 0.1.0
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

  ✔  azure-check-blob-storage: Check to see if some blob storage exists.
     ✔  apprentice-chef-test Storage Account is expected to exist
     ✔  apprentice-chef-test Storage Account name is expected to eq "apprentice-chef-test"


Profile: Azure Resource Pack (inspec-azure)
Version: 1.14.0
Target:  azure://b02e675a-cee0-49bd-a056-daa7ed05bf4e

     No tests executed.

Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
```
You can also see the resuls in the Chef Automate browser.

We can use this information to adjust the software that creates the storage blobs, which could be Chef remediation cookbooks.<br />
2. Debugging your test (advanced, skip if you want). As you build out your tests it is useful to be able to debug them, we can use `pry` for that. Uncomment the `require "pry";binding.pry` line and run your InSpec test again, you sholuld see output like this:<br />
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
Execution has been stopped just after the call to the `azurerm_storage_account_blob_containers` resource. The `names` method returns the storage names, we can see the first name returned by typing `blob`.<br />
``` ruby
[1] pry(#<Inspec::Rule>)> blob
=> "apprentice-chef-test"
```
We can go even further and see what our later<br />
`describe azurerm_storage_account_blob_container(resource_group: 'apprentice-chef', storage_account_name: 'apprenticechef',    blob_container_name: blob)`<br />
code will do, try it:<br />
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
      truncated

  ```
  We can see the instance varaible that we are perfoming our check against:<br />
  ``` ruby
      @name="apprentice-chef-test",
```
Press `q` to exit and `exit-program` to exit pry.
## 4. Center For Internet (CIS) Profile execution
1. The Center for Internet Security produces the CIS Azure Foundation Benchmark, Chef has implemented that benchmark uisng InSpec, which is also certified by CIS. We are now going to obtain that benchmark from Chef Automate and execute it against the Azure cloud.<br />
Login to Chef Automate via the terminal:<br />
`inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> <Chef Automate Hostname>`<br />
For example:<br />
`inspec compliance login --insecure --user=workstation-1 --token m6E8BQ5iCWLMBFUpIPRlRhrqR6k= afd-a2.chefdemo.cloud`
```
Stored configuration for Chef Automate2: https://afd-a2.chefdemo.cloud/api/v0' with user: 'workstation-1'
```
Open the Chrome Browser and go to the `Compliance` menu, then the `Profiles` tab on the left, see that the `CIS Azure Foundations Benchmark 1.1.0 Level 1` profile is available to your `workstation-x` user.
![Chef Automate Profile](/labs/images/azure-foundation.png)
Next lets execute that profile against the Azure API (replace `<x>` with your workstation number) - the tests will take about 4 minutes to run.<br />
`inspec exec compliance://workstation-<x>/cis-azure-foundations-level1 -t azure:// --config=reporter.json`<br />
Look at the scan results in the Chef Automate browser:<br />
![CIS Azure API Scan Results](/labs/images/azure-cis-run.png)


[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
