# InSpec on Windows
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
InSpec is an open-source testing framework for infrastructure with a human and machine-readable language for specifying compliance, security and policy requirements.  

Don't have InSpec installed?  Here you go - https://downloads.chef.io/inspec. 

Need the code? You will find it here - https://github.com/anthonygrees/compliance-windows. 

### Step 1: Check your InSpec Version

Run the following command. 
```bash
inspec --version
```
  
Your output will be as follows:
```bash
PS C:\chef-repo> inspec --version

4.22.1

PS C:\chef-repo>
```
Note: You may have a newer version of InSpec than ```4.22.1```.  
  
### Step 2: Create a new InSpec profile
Create a new InSpec Profile.  Run the following command:
```bash
inspec init profile windows-example --platform=os
```
Your output will be as follows:
```bash
PS C:\chef-repo> inspec init profile windows-example

 ─────────────────────────── InSpec Code Generator ───────────────────────────

Creating new profile at C:/chef-repo/windows-example
 • Creating directory controls
 • Creating file controls/example.rb
 • Creating file inspec.yml
 • Creating file README.md

PS C:\chef-repo>
```
  
Change Directory into your new Profile.  Run the following command:  
```bash
cd windows-example
```
Your output will be as follows:
```bash
PS C:\chef-repo> cd windows-example
PS C:\chef-repo\windows-example>
```
   
Open Visual Studio Code Editor. Run the following command:  
```bash
code .
```
A Visual Studio Windows will open.  
![VS Code](/labs/images/w_vscode.png)
  
### Step 3: Create a simple Windows Version check
Replace the code in example.rb file with the following:

```ruby
# Windows Versions - Check for Min of Win 2012
# Win2016 - NT 10.0 | Win 2012 R2 - NT 6.3 | Win 2012 - NT 6.2
#

control 'WINDOWS VERSION' do
  impact 0.8
  title 'This test checks for a minimum Windows version of 2012 - NT 6.2.0'

  describe os.family do
    it { should eq 'windows' }
  end

  describe os.name do
    it { should eq 'windows_server_2016_datacenter' }
  end

  describe os.release do
    it { should > '10.0' }
  end
end
```
  
To execute this using InSpec, run the following command
```bash
inspec exec .
```
  
Your output will be as follows:
```bash
PS C:\chef-repo\windows-example> inspec exec .

Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS VERSION: This test checks for a minimum Windows version of 2012 - NT 6.2.0
     [PASS]  windows is expected to eq "windows"
     [PASS]  windows_server_2016_datacenter is expected to eq "windows_server_2016_datacenter"
     [PASS]  10.0.14393 is expected to > "10.0"


Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
  
## Step 4: Report in Chef Automate
Create yourself a GUID.  The New-Guid cmdlet creates a random globally unique identifier (GUID). If you need a unique ID in a script, you can create a GUID, as needed.
```bash
New-Guid
```
Your output will look like this:
```bash
PS C:\chef-repo\windows-example> New-Guid

Guid
----
b2303f06-28e4-40fb-af33-0528768213b1
```
  
Copy the GUID.
  
Create a file called ```inspec.json``` in <your_profile_name> directory and add the following:

To create the file, type `code inspec.json`

Note: Remember to update the ```json``` and put your name in ```"node_name" : "<YOUR_NAME_HERE>"``` and add your ```TOKEN``` from the spreadsheet.

```json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://anthony-a2.chef-demo.com/data-collector/v0/",
      "token" : "AAAA-AAAA-AAAA-AAAAB",
      "insecure" : true,
      "node_name" : "YourName-Win2016",
      "environment" : "windows",
      "node_uuid" : "ADD_GUID_HERE"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```

To execute this using InSpec and report to A2 run the following command

```bash
inspec exec . --json-config inspec.json
```
Your report will appear under the `compliance` tab

![A2 Reporter](/labs/images/win_reporter1.png)
  
  
You will also see the folloing STDOUT:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS VERSION: This test checks for a minimum Windows version of 2012 - NT 6.2.0
     [PASS]  windows is expected to eq "windows"
     [PASS]  windows_server_2016_datacenter is expected to eq "windows_server_2016_datacenter"
     [PASS]  10.0.14393 is expected to > "10.0"


Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
  
  
### Step 5: Check Windows Hot Fixes
Add the following example for looping through Windows KB and Hotfixes

```ruby
## Looping example WannaCry Vulnerability Check
control 'WINDOWS HOTFIX - LOOP' do
  impact 0.8
  title 'This test checks that a numberof Windows Hotfixs are installed - Looping Example'

  hotfixes = %w{ KB4012598 KB4042895 KB4041693 }

  describe.one do
    hotfixes.each do |hotfix|
      describe windows_hotfix(hotfix) do
        it { should_not be_installed }
      end
    end
  end
end
```
  
To execute this using InSpec, run the following command
```bash
inspec exec . --json-config inspec.json
```
  
Your output will be as follows:
```bash
PS C:\chef-repo\windows-example> inspec exec .

Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS VERSION: This test checks for a minimum Windows version of 2012 - NT 6.2.0
     [PASS]  windows is expected to eq "windows"
     [PASS]  windows_server_2016_datacenter is expected to eq "windows_server_2016_datacenter"
     [PASS]  10.0.14393 is expected to > "10.0"
  [PASS]  WINDOWS HOTFIX - LOOP: This test checks that a numberof Windows Hotfixs are installed - Looping Example
     [PASS]  Windows Hotfix KB4012598 is expected not to be installed
     [PASS]  Windows Hotfix KB4042895 is expected not to be installed
     [PASS]  Windows Hotfix KB4041693 is expected not to be installed


Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 6 successful, 0 failures, 0 skipped
```
  
### Step 6: Check if a package is installed
Is a particular package installed ? Add the following code.

```ruby
control 'PACKAGE INSTALLED _ TELNET and CHROME' do
  impact 0.8
  title 'This test checks that a package is installed'

  describe package('telnetd') do
    it { should_not be_installed }
  end

  describe package('Google Chrome') do
    it { should be_installed}
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS VERSION: This test checks for a minimum Windows version of 2012 - NT 6.2.0
     [PASS]  windows is expected to eq "windows"
     [PASS]  windows_server_2016_datacenter is expected to eq "windows_server_2016_datacenter"
     [PASS]  10.0.14393 is expected to > "10.0"
  [PASS]  WINDOWS HOTFIX - LOOP: This test checks that a numberof Windows Hotfixs are installed - Looping Example
     [PASS]  Windows Hotfix KB4012598 is expected not to be installed
     [PASS]  Windows Hotfix KB4042895 is expected not to be installed
     [PASS]  Windows Hotfix KB4041693 is expected not to be installed
  [PASS]  PACKAGE INSTALLED _ TELNET and CHROME: This test checks that a package is installed
     [PASS]  System Package telnetd is expected not to be installed
     [PASS]  System Package Google Chrome is expected to be installed


Profile Summary: 3 successful controls, 0 control failures, 0 controls skipped
Test Summary: 8 successful, 0 failures, 0 skipped
```

### Step 7: Check if a Windows Service installed and Enabled
Is a particular Service installed? Add the following code.

```ruby
## service example
control 'SERVICE INSTALLED' do
  impact 0.8
  title 'This test checks the service is installed'

  describe service('DHCP Client') do
    it { should be_installed }
    it { should be_running }
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS VERSION: This test checks for a minimum Windows version of 2012 - NT 6.2.0
     [PASS]  windows is expected to eq "windows"
     [PASS]  windows_server_2016_datacenter is expected to eq "windows_server_2016_datacenter"
     [PASS]  10.0.14393 is expected to > "10.0"
  [PASS]  WINDOWS HOTFIX - LOOP: This test checks that a numberof Windows Hotfixs are installed - Looping Example
     [PASS]  Windows Hotfix KB4012598 is expected not to be installed
     [PASS]  Windows Hotfix KB4042895 is expected not to be installed
     [PASS]  Windows Hotfix KB4041693 is expected not to be installed
  [PASS]  PACKAGE INSTALLED _ TELNET and CHROME: This test checks that a package is installed
     [PASS]  System Package telnetd is expected not to be installed
     [PASS]  System Package Google Chrome is expected to be installed
  [PASS]  SERVICE INSTALLED: This test checks the service is installed
     [PASS]  Service DHCP Client is expected to be installed
     [PASS]  Service DHCP Client is expected to be running


Profile Summary: 4 successful controls, 0 control failures, 0 controls skipped
Test Summary: 10 successful, 0 failures, 0 skipped
```
  
### Step 8: Checking HTTP and HTTPS
Use the InSpec Port resource to test HTTP and HTTPS

```ruby
control 'HTTP AND HTTPS' do
  impact 0.8
  title 'This test checks the HTTP and HTTPS protocols'

  # Test HTTP port 80, is not listening and no protocol TCP, ICMP, UDP
  describe port(80) do
      it { should_not be_listening }
      its('protocols') { should_not cmp 'tcp6' }
      its('protocols') { should_not include('icmp') }
      its('protocols') { should_not include('tcp') }
      its('protocols') { should_not include('udp') }
      its('protocols') { should_not include('udp6') }
      its('addresses') { should_not include '0.0.0.0' }
  end

  # Test HTTPS port 443, listening with TCP and UDP
  describe port(443) do
      it { should be_listening }
      its('protocols') { should_not cmp 'tcp6' }
      its('protocols') { should_not include('icmp') }
      its('protocols') { should include('tcp') }
      its('protocols') { should include('udp') }
      its('protocols') { should_not include('udp6') }
      its('addresses') { should include '0.0.0.0' }
  end
end
```
  
To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [FAIL]  HTTP AND HTTPS: This test checks the HTTP and HTTPS protocols (4 failed)
     [PASS]  Port 80 is expected not to be listening
     [PASS]  Port 80 protocols is expected not to cmp == "tcp6"
     [PASS]  Port 80 protocols is expected not to include "icmp"
     [PASS]  Port 80 protocols is expected not to include "tcp"
     [PASS]  Port 80 protocols is expected not to include "udp"
     [PASS]  Port 80 protocols is expected not to include "udp6"
     [PASS]  Port 80 addresses is expected not to include "0.0.0.0"
     [FAIL]  Port 443 is expected to be listening
     expected `Port 443.listening?` to return true, got false
     [PASS]  Port 443 protocols is expected not to cmp == "tcp6"
     [PASS]  Port 443 protocols is expected not to include "icmp"
     [FAIL]  Port 443 protocols is expected to include "tcp"
     expected [] to include "tcp"
     [FAIL]  Port 443 protocols is expected to include "udp"
     expected [] to include "udp"
     [PASS]  Port 443 protocols is expected not to include "udp6"
     [FAIL]  Port 443 addresses is expected to include "0.0.0.0"
     expected [] to include "0.0.0.0"


Profile Summary: 4 successful controls, 1 control failure, 0 controls skipped
Test Summary: 20 successful, 4 failures, 0 skipped
```
  
### Step 9:  You can also do Ranges

```ruby
control 'PORT RANGE' do
  impact 0.8
  title 'This test checks the ports in a range'
  
  describe port.where { port > 0 && port < 21 } do
    it { should_not be_listening }
  end

  describe port.where { port > 21 && port < 80 } do
    it { should_not be_listening }
  end

  describe port.where { port > 80 && port < 443 } do
    it { should_not be_listening }
  end
end
  ```
    
To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [FAIL]  PORT RANGE: This test checks the ports in a range (2 failed)
     [PASS]  Port  with port > 0 port < 21 is expected not to be listening
     [FAIL]  Port  with port > 21 port < 80 is expected not to be listening
     expected `Port  with port > 21 port < 80.listening?` to return false, got true
     [FAIL]  Port  with port > 80 port < 443 is expected not to be listening
     expected `Port  with port > 80 port < 443.listening?` to return false, got true


Profile Summary: 4 successful controls, 2 control failures, 0 controls skipped
Test Summary: 21 successful, 6 failures, 0 skipped
```
  

### Step 10: Check Windows Tasks
Use the windows_task InSpec resource to check the state of tasks.

```ruby
control 'WINDOWS TASKS' do
  impact 0.8
  title 'This test checks the Windows Tasks'

  describe windows_task('\Microsoft\Windows\AppID\PolicyConverter') do
    it { should be_disabled }
  end

  describe windows_task('\Microsoft\Windows\AppID\PolicyConverter') do
    its('logon_mode') { should eq 'Interactive/Background' }
    its('last_result') { should eq '267011' }
    its('task_to_run') { should cmp '%Windir%\system32\appidpolicyconverter.exe' }
    its('run_as_user') { should eq 'SYSTEM' }
  end

  describe windows_task('\Microsoft\Windows\Defrag\ScheduledDefrag') do
    it { should exist }
  end
end
```
  
To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

  [PASS]  WINDOWS TASKS: This test checks the Windows Tasks
     [PASS]  Windows Task '\Microsoft\Windows\AppID\PolicyConverter' is expected to be disabled
     [PASS]  Windows Task '\Microsoft\Windows\AppID\PolicyConverter' logon_mode is expected to eq "Interactive/Background"
     [PASS]  Windows Task '\Microsoft\Windows\AppID\PolicyConverter' last_result is expected to eq "267011"
     [PASS]  Windows Task '\Microsoft\Windows\AppID\PolicyConverter' task_to_run is expected to cmp == "%Windir%\\system32\\appidpolicyconverter.exe"
     [PASS]  Windows Task '\Microsoft\Windows\AppID\PolicyConverter' run_as_user is expected to eq "SYSTEM"
     [PASS]  Windows Task '\Microsoft\Windows\Defrag\ScheduledDefrag' is expected to exist


Profile Summary: 5 successful controls, 2 control failures, 0 controls skipped
Test Summary: 27 successful, 6 failures, 0 skipped
```
  
### Step 11: CIS Example Profile
Compliance of the OS settings on the windows client
- Check and verify Group Policy Settings (GPO) with reference to CIS Windows 10 1703 benchmark is begin applied
- When new monthly Windows security patch is applied to the current image, to check if the new patches is successfully applied. Where possible, show the status BEFORE and AFTER the patch for comparison and highlight any errors etc.


```ruby
control "xccdf_org.cisecurity.benchmarks_rule_18.8.18.3_L1_Ensure_Configure_registry_policy_processing_Process_even_if_the_Group_Policy_objects_have_not_changed_is_set_to_Enabled_TRUE" do
  title "(L1) Ensure 'Configure registry policy processing: Process even if the Group Policy objects have not changed' is set to 'Enabled: TRUE'"
  desc  "The \"Process even if the Group Policy objects have not changed\" option updates and reapplies policies even if the policies have not changed."
  impact 1.0
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\Group Policy\\{35378EAC-683F-11D2-A89A-00C04FBBCFA2}") do
    it { should_not have_property "NoGPOListChanges" }
  end
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\Group Policy\\{35378EAC-683F-11D2-A89A-00C04FBBCFA2}") do
    its("NoGPOListChanges") { should_not cmp == 0 }
  end
end

control "xccdf_org.cisecurity.benchmarks_rule_18.8.18.4_L1_Ensure_Turn_off_background_refresh_of_Group_Policy_is_set_to_Disabled" do
  title "(L1) Ensure 'Turn off background refresh of Group Policy' is set to 'Disabled'"
  desc  "This policy setting prevents Group Policy from being updated while the computer is in use."
  impact 1.0
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System") do
    it { should_not have_property "DisableBkGndGroupPolicy" }
  end
end
```
  
To execute this using InSpec, run the following command

```bash
inspec exec . --json-config inspec.json
```
  
Your output will be in the Chef Automate UI and on the STDOUT as follows:
```bash
Profile: InSpec Profile (windows-example)
Version: 0.1.0
Target:  local://

xccdf_org.cisecurity.benchmarks_rule_18.8.18.3_L1_Ensure_Configure_registry_policy_processing_Process_even_if_the_Group_Policy_objects_have_not_changed_is_set_to_Enabled_TRUE: (L1) Ensure 'Configure registry policy processing: Process even if the Group Policy objects have not changed' is set to 'Enabled: TRUE'
     [PASS]  Registry Key HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Group Policy\{35378EAC-683F-11D2-A89A-00C04FBBCFA2} is expected not to have property "NoGPOListChanges"
     [PASS]  Registry Key HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Group Policy\{35378EAC-683F-11D2-A89A-00C04FBBCFA2} NoGPOListChanges is expected not to cmp == 0
  [PASS]  xccdf_org.cisecurity.benchmarks_rule_18.8.18.4_L1_Ensure_Turn_off_background_refresh_of_Group_Policy_is_set_to_Disabled: (L1) Ensure 'Turn off background refresh of Group Policy' is set to 'Disabled'
     [PASS]  Registry Key HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System is expected not to have property "DisableBkGndGroupPolicy"


Profile Summary: 7 successful controls, 2 control failures, 0 controls skipped
Test Summary: 30 successful, 6 failures, 0 skipped
```
  
Your report will also appear under the `compliance` tab

![A2 Reporter](/labs/images/win_reporter2.png)

  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
