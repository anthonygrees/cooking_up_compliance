# Scanning GCP Cloud with InSpec
## 1. Configure your workstation
1. Create a remote desktop connection to your Windows workstation, login using `chef` as the username, ask your instructor for the password.<br />
2. Open the remote desktop connection and wait for the background to change to `DevOps Better Together`, the Chrome browser and a Powershell window to open.
3. We are going to use vscode to write our code and a Linux Node to run our InSpec tests on - we will setup vscode to be our one stop environment for this:<br />
   i. In the Powershell window type<br />
   `code .`<br />
   ii. While in vscode press the F1 function key and start typing<br />
   `Remote-SSH: Connect to Host...`, select it and type<br />
   `ec2-user@<ask your instructor for the ip address>`, select it and then select `Linux` (and `continue` if this is the first time you have connected).<br />
   iii. Close the Welcome page and click on the `Explorer icon` on the top left, Select `Open Folder` and fill in `/home/ec2-user/inspec-labs` and click `OK`.</br >
   iv. From the vscode menu click `Terminal` and then `New Terminal`.<br />
   Your setup should now look similar to this:<br />
   ![Lab Setup Image](/labs/images/vscode-setup.png "Lab Setup")
4. Login to Chef Automate via the Chrome Browser, the browser should be open at the correct URL, if not ask the instructor for the URL.<br />
```
Username = workstation-x
Password = workstation!
```
Replace x with your workstation number given to you by the instructor.
## 2. Explore Your first Inspec Profile
1. Check to make sure that InSpec can talk to GCP, in the vscode terminal type:(if prompted accept the Chef License).<br />
`inspec detect -t gcp://`
2. Create an InSpec profile to scan gcp, in the terminal type:<br />
`inspec init profile gcp --platform=gcp`<br />
`cd gcp`</br >
Observe the files and directories created in the terminal or the vscode file browser on the left.<br />
3. Scan GCP with your newly created profile:<br />
`inspec exec . -t gcp:// --input gcp_project_id=chef-sa`<br />
4. Send the InSpec scan results to Chef Automate. First you need to create a `reporter.json` file like this in the `gcp` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with you workstation number. You can create the file by right clicking on the `gcp` directory or in the terminal with `touch reporter.json`<br />
You will also need to create a UUID for your GCP scan, run `uuidgen` in your terminal.
``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://<Chef Automate Hostname>/data-collector/v0/",
      "token" : "<Chef Automate Token>",
      "insecure" : true,
      "node_name" : "workstation-<x>-gcp-api",
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
`inspec exec . -t gcp:// --config=reporter.json --input gcp_project_id=chef-sa`<br />
The Chrome Browser should already be open, if not open it and ask your instructor for the Chef Automate URL to use.<br />
In Chef Automate Click the `Compliance` menu, observe your node, there may be other nodes from your classmates. In Chef Automate we refer to everything as a node, so in this case our GCP-API scan is our node.<br />
![Chef Automate Compliance](/labs/images/gcp-compliance.png "Chef Automate Compliance")<br />
Click the `"x" Nodes` menu to see the Node list.<br />
![Chef Automate Compliance](/labs/images/gcp-node.png "Chef Automate Compliance")<br />
Click on the node that is your scan to see the detail of the scan.<br />
![Chef Automate Compliance](/labs/images/gcp-node-detail.png "Chef Automate Compliance")<br />
Notice that we have two Passed Controls for your gcp-api scanned node.<br />
Click the `+` next to the `gcp-regions-loop-1.0` control to reveal the Results:<br />
![Control Result](/labs/images/gcp-control-results.png)<br />
The Source - the actual InSpec code that ran to perform the check:<br />
![Control Source](/labs/images/gcp-control-source.png)<br />
5. You may have noticed that we supplied the `gcp_project_id` as an input when we ran our scans. Lets investigate another way of providing input parameters.<br />
In the `gcp` directory create a `inputs.yml` file and add this:<br />
`gcp_project_id: 'chef-sa'`<br />
Save the changes.<br />
Observe the `inspec.yml` file to see where the input is defined as shown:<br />
``` yml
name: gcp
title: GCP InSpec Profile
maintainer: The Authors
copyright: The Authors
copyright_email: you@example.com
license: Apache-2.0
summary: An InSpec Compliance Profile For GCP
version: 0.1.0
inspec_version: '>= 2.3.5'
inputs:
- name: gcp_project_id
  required: true
  description: 'The GCP project identifier.'
  type: string
depends:
- name: inspec-gcp
  url: https://github.com/inspec/inspec-gcp/archive/master.tar.gz
supports:
- platform: gcp
```
Lets run InSpec again this time specifying the `inputs.yml` file as an input file:<br />
`inspec exec . -t gcp:// --config=reporter.json --input-file inputs.yml`<br />
(If you see `WARN: Input 'gcp_project_id' does not have a value` it is a known bug and can safely be ignored).<br />
Take a look at the results in Chef Automate, you should see the same results as before.<br />
6. What if we really do not want to run one of the controls? InSpec has a Waiver capability to allow you to do this.<br />
Under the `gcp` directory create a `waiver.yml` file and add the following to it:<br />
``` yml
gcp-regions-loop-1.0:
  expiration_date: 2021-02-28
  run: false
  justification: "Security have signed off not doing this check until the end of February 2021"
```
Run InSpec with the waiver like this:<br />
`inspec exec . -t gcp:// --config=reporter.json --input-file=inputs.yml --waiver-file waiver.yml`<br />
Look in Chef Automate to see the waiver results, including the reason for the waiver:<br />
![Control Results](/labs/images/gcp-waiver.png)<br />
## 3. Writing your own InSpec Profile
1. InSpec ships with many out of the box resources that allow you to easily implement security checks, see [https://www.inspec.io/docs/reference/resources/](https://www.inspec.io/docs/reference/resources/).<br />
In this exercise we are going to explore the Plural Resource [`google_storage_buckets`](https://www.inspec.io/docs/reference/resources/google_storage_buckets/) and its equivalent Singular Resource [`google_storage_bucket`](https://www.inspec.io/docs/reference/resources/google_storage_bucket/), we will then use the [`google_kms_crypto_key`](https://www.inspec.io/docs/reference/resources/google_kms_crypto_key/) to check for the correct encryption levels. The test will allow us to check that labelled storage buckets are encrypted correctly.<br />
Under the `controls` directory delete the `example.rb` file and create a `bucket.rb` file, type the following into the file.

``` ruby
# copyright: 2020, The Authors

title "GCP Security Checks"

gcp_project_id = input("gcp_project_id")

control 'gcp-check-bucket-encryption' do
    impact 1.0
    title 'Classification of highly-confidential Buckets must have HSM keys'
    desc 'Check that all buckets labelled highly-confidential have Hardware Encryption Keys'
  
    google_storage_buckets(project: gcp_project_id).bucket_names.each do |bucket_name|
    #require "pry"; binding.pry  
      bucket = google_storage_bucket(name: bucket_name)

      unless bucket.labels.nil?
        if (bucket.respond_to? :labels) && (bucket.labels.include? 'apprentice-chef-classification') && (bucket.labels['apprentice-chef-classification'] == 'highly-confidential')
          describe bucket do
              it { should respond_to :encryption}
          end
          if bucket.respond_to? :encryption
            items=bucket.encryption.default_kms_key_name.split('/')
            project=items[1]
            location=items[3]
            keyring=items[5]
            key=items[7]
            kms = google_kms_crypto_key(project:project,location:location,key_ring_name:keyring,name:key)
            describe bucket_name + ' ' + kms.version_template.protection_level do
                it {should eq bucket_name + ' HSM'}
          end
        end
      end
    end
  end
end
``` 

Execute your InSpec tests:<br />
`inspec exec . -t gcp:// --config=reporter.json --input-file=inputs.yml`<br />

```
Profile: GCP InSpec Profile (gcp)
Version: 0.1.0
Target:  gcp://afdaniels@chef-sa.iam.gserviceaccount.com

  ×  gcp-check-bucket-encryption: Classification of highly-confidential Buckets must have HSM keys (1 failed)
     ✔  Bucket apprentice-chef1 is expected to respond to #encryption
     ✔  apprentice-chef1 HSM is expected to eq "apprentice-chef1 HSM"
     ✔  Bucket apprentice-chef2 is expected to respond to #encryption
     ×  apprentice-chef2 SOFTWARE is expected to eq "apprentice-chef2 HSM"
     
     expected: "apprentice-chef2 HSM"
          got: "apprentice-chef2 SOFTWARE"
     
     (compared using ==)



Profile: Google Cloud Platform Resource Pack (inspec-gcp)
Version: 1.4.0
Target:  gcp://afdaniels@chef-sa.iam.gserviceaccount.com

     No tests executed.

Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
Test Summary: 3 successful, 1 failure, 0 skipped
```
You can also see the results in the Chef Automate browser.

We can see at this point that one of our buckets is incorrectly encrypted - now would be a good time to adjust the software that creates the buckets, which could be Chef remediation cookbooks.<br />
2. (Advanced - skip if you want). Debugging your test: As you build out your tests it is useful to be able to debug them, we can use `pry` for that. Uncomment the `require "pry";binding.pry` line and run your InSpec test again, you should see output like this:<br />
``` ruby
8:     impact 1.0
     9:     title 'Classification of highly-confidential Buckets must have HSM keys'
    10:     desc 'Check that all buckets labelled highly-confidential have Hardware Encryption Keys'
    11:   
    12:     google_storage_buckets(project: gcp_project_id).bucket_names.each do |bucket_name|
 => 13:     require "pry"; binding.pry  
    14:       bucket = google_storage_bucket(name: bucket_name)
    15: 
    16:       unless bucket.labels.nil?
    17:         if (bucket.respond_to? :labels) && (bucket.labels.include? 'apprentice-chef-classification') && (bucket.labels['apprentice-chef-classification'] == 'highly-confidential')
    18:           describe bucket do
```
Execution has been stopped just after the call to the `google_storage_buckets` resource. The `each` method allows us to iterate over the bucket names, we can see the first name returned by typing `bucket_name`.<br />
``` ruby
[1] pry(#<Inspec::Rule>)> bucket_name
=> "apprentice-chef1"
```
We can go even further and see what our later `google_storage_bucket(name: bucket_name)` code will do, try it:<br />
``` ruby
=> #<#<Class:0x0000000007d13560>:0x0000000008be5818
 @__backend_runner__=Inspec::Backend::Class @transport=Train::Transports::Gcp::Connection,
 @__resource_name__="google_storage_bucket",
 @acl=nil,
 @connection=#<#<Class:0x00000000087ef038>::GcpApiConnection:0x0000000008bf35a8 @service_account_file="/home/ec2-user/gcp.json">,
 @cors=nil,
 @default_event_based_hold=false,
 @default_object_acl=nil,
 @encryption=
  #<#<Class:0x00000000087ef038>::GoogleInSpec::Storage::Property::BucketEncryption:0x0000000008c48580
   @default_kms_key_name="projects/chef-sa/locations/europe-west2/keyRings/apprentice-chef/cryptoKeys/apprentice-chef-hsm",
   @parent_identifier="Bucket apprentice-chef1">,
 @fetched=
 .
 .
 .
 Truncated
  ```
  We can see the encryption information that we are perfoming our later key check against:<br />
  ``` ruby
@encryption=
  #<#<Class:0x00000000087ef038>::GoogleInSpec::Storage::Property::BucketEncryption:0x0000000008c48580
   @default_kms_key_name="projects/chef-sa/locations/europe-west2/keyRings/apprentice-chef/cryptoKeys/apprentice-chef-hsm",
```
Press `q` to exit and `exit-program` to exit pry.
## 4. Center For Internet (CIS) Profile execution
1. The Centre for Internet Security produces a CIS Docker Benchmark, Chef has implemented that benchmark uisng InSpec. We are now going to obtain that benchmark from Chef Automate and execute it against the GCP cloud.<br />
Login to Chef Automate via the terminal:<br />
`inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> <Chef Automate Hostname>`<br />
For example:<br />
`inspec compliance login --insecure --user=workstation-1 --token m6E8BQ5iCWLMBFUpIPRlRhrqR6k= afd-a2.chefdemo.cloud`
```
Stored configuration for Chef Automate2: https://afd-a2.chefdemo.cloud/api/v0' with user: 'workstation-1'
```
Open the Chrome Browser and go to the `Compliance` menu, then the `Profiles` tab on the left, see that the `CIS Google Cloud Platform Foundation Benchmark Level 1` profile is available to your `workstation-x` user.
![Chef Automate Profile](/labs/images/docker-benchmark.png)
Next lets execute that profile against our Docker Host (replace `<x>` with your workstation number) - it will only take a few seconds to run:<br />
`inspec exec compliance://workstation-<x>/cis-docker-benchmark --config=reporter.json --input-file=inputs.yml`<br />
Look at the scan results in the Chef Automate browser:<br />
![CIS GCP API Scan Results](/labs/images/docker-cis-run.png)


[Back to the Lab Index](../README.md)
