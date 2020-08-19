[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
## InSpec on Linux
InSpec is an open-source testing framework for infrastructure with a human and machine-readable language for specifying compliance, security and policy requirements.

Don't have InSpec installed?  Here you go - https://downloads.chef.io/inspec

Need the code? You can find it here - https://github.com/anthonygrees/compliance-linux

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
    
  
Run the following command to check your InSpec version.
```bash
$ inspec --version
```
  
You will see an output as follows:  
```bash
[ec2-user@ip-172-31-54-152 inspec-labs]$ inspec --version
4.22.8
[ec2-user@ip-172-31-54-152 inspec-labs]$ 
```
  
### Step 2: Determine your platform
Use the InSpec ```detect``` command to check the platform you are running on.  You will need to type ```yes``` to accept the Chef License.
```bash
inspec detect
```
  
You will see an output as follows:  
```bash
[ec2-user@ip-172-31-54-152 inspec-labs]$ inspec detect
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

Name:      redhat
Families:  redhat, linux, unix, os
Release:   7.8
Arch:      x86_64
[ec2-user@ip-172-31-54-152 inspec-labs]$ 
```
  
### Step 3: Check for insecure protocol
Now, create a new InSpec profile
```bash
## Create a New InSpec Profile
inspec init profile linux-example --platform=os
  
## Change Directory into your new Profile
$ cd linux-example
  
## Write your Controls in the example.rb
controls\example.rb
```
Add the following code to your ```controls\example.rb```
```ruby
# Disallow insecure protocols by testing
  
describe package('telnetd') do
  it { should_not be_installed }
end
```
  
InSpec makes it easy to run your tests wherever you need.
```bash
inspec exec . 
```
  
You will see an output as follows:  
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec .

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  System Package telnetd
     ✔  is expected not to be installed

Test Summary: 1 successful, 0 failures, 0 skipped
```
  
### Step 4: Ensure telnet server is not enabled
Add the following InSpec check for telnet.
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled" do
  title "Ensure telnet server is not enabled"
  desc  "
    The telnet-server package contains the telnet daemon, which accepts connections from users from other systems via the telnet protocol.

    Rationale: The telnet protocol is insecure and unencrypted. The use of an unencrypted transmission medium could allow a user with access to sniff network traffic the ability to steal credentials. The ssh package provides an encrypted session and stronger security.
  "
  impact 1.0
  describe bash("egrep \"^telnet\" /etc/inetd.conf") do
    its("exit_status") { should_not eq 0 }
  end
end
```
You can run your test with the following command
```bash
inspec exec .
```
  
  
You will see an output as follows:  
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec .

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
```
  
### Step 5: Report in Chef Automate
1. You will need to create a UUID for your Linux scan, run `uuidgen` in your terminal.    
     
  You will see output as follows:
  ```bash
    [ec2-user@ip-172-31-54-168 aws]$ uuidgen
    c6754ceb-b9a8-4917-a797-c0d0589246d6
      
```

2. Next you need to create a `reporter.json` file like this in the `linux-example` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with you workstation number. You can create the file by either:  
   - right clicking on the `aws` directory or   
   - in the terminal type `touch reporter.json`  
  
``` json
{
  "reporter": {
    "automate" : {
      "stdout" : false,
      "url" : "https://anthony-a2.chef-demo.com/data-collector/v0/",
      "token" : "<Chef Automate Token>",
      "insecure" : true,
      "node_name" : "YourName-Linux-Demo",
      "environment" : "linux-example",
      "node_uuid" : "<uuidgen>"
    },
    "cli" : {
      "stdout" : true
    }
  }
}
```

To execute this using InSpec and report to A2 run the following command

```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows:  
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec . --json-config reporter.json

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
```
  
![Linux](/labs/images/linux_reporter1.png)
  
### Step 6: Ensure FTP Server is not enabled

```ruby
control "xccdf_org.cisecurity.benchmarks_rule_6.9_Ensure_FTP_Server_is_not_enabled" do
  title "Ensure FTP Server is not enabled"
  desc  "
    The File Transfer Protocol (FTP) provides networked computers with the ability to transfer files.

    Rationale: FTP does not protect the confidentiality of data or authentication credentials. It is recommended sftp be used if file transfer is required. Unless there is a need to run the system as a FTP server (for example, to allow anonymous downloads), it is recommended that the package be deleted to reduce the potential attack surface.
  "
  impact 0.0
  describe bash("initctl show-config vsftpd | egrep \"^\s*start\"") do
    its("exit_status") { should_not eq 0 }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows:  
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec . --json-config reporter.json

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.9_Ensure_FTP_Server_is_not_enabled: Ensure FTP Server is not enabled
     ✔  Bash command initctl show-config vsftpd | egrep "^ *start" exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
  
### Step 7: Ensure shadow group is empty
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_13.20_Ensure_shadow_group_is_empty" do
  title "Ensure shadow group is empty"
  desc  "
    The shadow group allows system programs which require access the ability to read the /etc/shadow file. No users should be assigned to the shadow group.

    Rationale: Any users assigned to the shadow group would be granted read access to the /etc/shadow file. If attackers can gain read access to the /etc/shadow file, they can easily run a password cracking program against the hashed passwords to break them. Other security information that is stored in the /etc/shadow file (such as expiration) could also be useful to subvert additional user accounts.
  "
  impact 1.0
  describe file("/etc/group") do
    its("content") { should_not match(/^shadow:x:15:.+$/) }
  end
  describe bash("awk -F: '($4 == \"42\") { print }' /etc/passwd") do
    its("stdout") { should_not match(/.+/) }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows:  
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec . --json-config reporter.json

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.9_Ensure_FTP_Server_is_not_enabled: Ensure FTP Server is not enabled
     ✔  Bash command initctl show-config vsftpd | egrep "^ *start" exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
  
### Step 8: Ensure tftp-server is not enabled
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_5.1.7_Ensure_tftp-server_is_not_enabled" do
  title "Ensure tftp-server is not enabled"
  desc  "
    Trivial File Transfer Protocol (TFTP) is a simple file transfer protocol, typically used to automatically transfer configuration or boot machines from a boot server. The packages tftp and atftp are both used to define and support a TFTP server.

    Rationale: TFTP does not support authentication nor does it ensure the confidentiality or integrity of data. It is recommended that TFTP be removed, unless there is a specific need for TFTP. In that case, extreme caution must be used when configuring the services.
  "
  impact 1.0
  describe bash("egrep \"^tftp\" /etc/inetd.conf") do
    its("exit_status") { should_not eq 0 }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
[ec2-user@ip-172-31-54-152 linux-example]$ inspec exec . --json-config reporter.json

Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.9_Ensure_FTP_Server_is_not_enabled: Ensure FTP Server is not enabled
     ✔  Bash command initctl show-config vsftpd | egrep "^ *start" exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_13.20_Ensure_shadow_group_is_empty: Ensure shadow group is empty
     ✔  File /etc/group content is expected not to match /^shadow:x:15:.+$/
     ✔  Bash command awk -F: '($4 == "42") { print }' /etc/passwd stdout is expected not to match /.+/
  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.7_Ensure_tftp-server_is_not_enabled: Ensure tftp-server is not enabled
     ✔  Bash command egrep "^tftp" /etc/inetd.conf exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 4 successful controls, 0 control failures, 0 controls skipped
Test Summary: 6 successful, 0 failures, 0 skipped
```
  
### Step 9: Ensure DHCP Server is not enabled
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_6.4_Ensure_DHCP_Server_is_not_enabled" do
  title "Ensure DHCP Server is not enabled"
  desc  "
    The Dynamic Host Configuration Protocol (DHCP) is a service that allows machines to be dynamically assigned IP addresses.

    Rationale: Unless a server is specifically set up to act as a DHCP server, it is recommended that this service be deleted to reduce the potential attack surface.
  "
  impact 1.0
  describe bash("initctl show-config isc-dhcp-server | egrep \"^\sstart\"") do
    its("exit_status") { should_not eq 0 }
  end
  describe bash("initctl show-config isc-dhcp-server6 | egrep \"^\sstart\"") do
    its("exit_status") { should_not eq 0 }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
Profile: InSpec Profile (linux-example)
Version: 0.1.0
Target:  local://

  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.6_Ensure_telnet_server_is_not_enabled: Ensure telnet server is not enabled
     ✔  Bash command egrep "^telnet" /etc/inetd.conf exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.9_Ensure_FTP_Server_is_not_enabled: Ensure FTP Server is not enabled
     ✔  Bash command initctl show-config vsftpd | egrep "^ *start" exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_13.20_Ensure_shadow_group_is_empty: Ensure shadow group is empty
     ✔  File /etc/group content is expected not to match /^shadow:x:15:.+$/
     ✔  Bash command awk -F: '($4 == "42") { print }' /etc/passwd stdout is expected not to match /.+/
  ✔  xccdf_org.cisecurity.benchmarks_rule_5.1.7_Ensure_tftp-server_is_not_enabled: Ensure tftp-server is not enabled
     ✔  Bash command egrep "^tftp" /etc/inetd.conf exit_status is expected not to eq 0
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.4_Ensure_DHCP_Server_is_not_enabled: Ensure DHCP Server is not enabled
     ✔  Bash command initctl show-config isc-dhcp-server | egrep "^ start" exit_status is expected not to eq 0
     ✔  Bash command initctl show-config isc-dhcp-server6 | egrep "^ start" exit_status is expected not to eq 0

  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 5 successful controls, 0 control failures, 0 controls skipped
Test Summary: 8 successful, 0 failures, 0 skipped
```
  
### Step 10: Set Password Creation Requirement Parameters
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_9.2.1_Set_Password_Creation_Requirement_Parameters_Using_pam_cracklib" do
  title "Set Password Creation Requirement Parameters Using pam_cracklib"
  desc  "
    The pam_cracklib module checks the strength of passwords. It performs checks such as making sure a password is not a dictionary word, it is a certain length, contains a mix of characters (e.g. alphabet, numeric, other) and more. The following are definitions of the pam_cracklib.so options.

     retry=3- Allow 3 tries before sending back a failure.
     minlen=14 - password must be 14 characters or more
     dcredit=-1 - provide at least one digit
     ucredit=-1 - provide at least one uppercase character
     ocredit=-1 - provide at least one special character
     lcredit=-1 - provide at least one lowercase character
    The setting shown above is one possible policy. Alter these values to conform to your own organization's password policies.

    Rationale: Strong passwords protect systems from being hacked through brute force methods.
  "
  impact 1.0
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^retry/ { if ($2 <= 3) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^minlen/ { if ($2 >= 14) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^dcredit/ { if ($2 <= -1) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^ucredit/ { if ($2 <= -1) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^lcredit/ { if ($2 <= -1) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
  describe bash("egrep -v \"^[[:space:]]#\" /etc/pam.d/common-password | egrep \"pam_cracklib.so\" | sed -e 's/#.//' | tr -s '\t ' '\n' | awk -F = '/^ocredit/ { if ($2 <= -1) print $2 }'") do
    its("stdout") { should match(/.+/) }
  end
end
```
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
  ×  xccdf_org.cisecurity.benchmarks_rule_9.2.1_Set_Password_Creation_Requirement_Parameters_Using_pam_cracklib: Set Password Creation Requirement Parameters Using pam_cracklib (6 failed)
     ×  Bash command egrep -v "^[[:space:]]#" /etc/pam.d/common-password | egrep "pam_cracklib.so" | sed -e 's/#.//' | tr -s '   ' '
     ' | awk -F = '/^retry/ { if ($2 <= 3) print $2 }' stdout is expected to match /.+/
     expected "" to match /.+/
     Diff:
     @@ -1 +1 @@
     -/.+/
     +""
     ×  Bash command egrep -v "^[[:space:]]#" /etc/pam.d/common-password | egrep "pam_cracklib.so" | sed -e 's/#.//' | tr -s '   ' '
     ' | awk -F = '/^ocredit/ { if ($2 <= -1) print $2 }' stdout is expected to match /.+/
     expected "" to match /.+/
     Diff:
     @@ -1 +1 @@
     -/.+/
     +""


  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 5 successful controls, 1 control failure, 0 controls skipped
Test Summary: 8 successful, 6 failures, 0 skipped
```
  
### Step 11: Set Password Expiration Days
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_10.1.1_Set_Password_Expiration_Days" do
  title "Set Password Expiration Days"
  desc  "
    The PASS_MAX_DAYS parameter in /etc/login.defs allows an administrator to force passwords to expire once they reach a defined age. It is recommended that the PASS_MAX_DAYS parameter be set to less than or equal to 90 days.

    Rationale: The window of opportunity for an attacker to leverage compromised credentials or successfully compromise credentials via an online brute force attack is limited by the age of the password. Therefore, reducing the maximum age of a password also reduces an attacker's window of opportunity.
  "
  impact 1.0
  describe file("/etc/login.defs") do
    its("content") { should match(/^\s*PASS_MAX_DAYS\s+90/) }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
×  xccdf_org.cisecurity.benchmarks_rule_10.1.1_Set_Password_Expiration_Days: Set Password Expiration Days
     ×  File /etc/login.defs content is expected to match /^\s*PASS_MAX_DAYS\s+90/
     expected "#\n# Please note that the parameters in this configuration file control the\n# behavior of the tools...bers exist.\n#\nUSERGROUPS_ENAB yes\n\n# Use SHA512 to encrypt password.\nENCRYPT_METHOD SHA512\n\n" to match /^\s*PASS_MAX_DAYS\s+90/
     Diff:
     @@ -1,71 +1,141 @@
     -/^\s*PASS_MAX_DAYS\s+90/
     +#
     +# Please note that the parameters in this configuration file control the
     +# behavior of the tools from the shadow-utils component. None of these
     +# tools uses the PAM mechanism, and the utilities that use PAM (such as the
     +# passwd command) should therefore be configured elsewhere. Refer to
     +# /etc/pam.d/system-auth for more information.
     +#
     +
     +# *REQUIRED*
     +#   Directory where mailboxes reside, _or_ name of file, relative to the
     +#   home directory.  If you _do_ define both, MAIL_DIR takes precedence.
     +#   QMAIL_DIR is for Qmail
     +#
     +#QMAIL_DIR        Maildir
     +MAIL_DIR  /var/spool/mail
     +#MAIL_FILE        .mail
     +
     +# Password aging controls:
     +#
     +# PASS_MAX_DAYS   Maximum number of days a password may be used.
     +# PASS_MIN_DAYS   Minimum number of days allowed between password changes.
     +# PASS_MIN_LEN    Minimum acceptable password length.
     +# PASS_WARN_AGE   Number of days warning given before a password expires.
     +#
     +PASS_MAX_DAYS     99999
     +PASS_MIN_DAYS     0
     +PASS_MIN_LEN      5
     +PASS_WARN_AGE     7
     +
     +#
     +# Min/max values for automatic uid selection in useradd
     +#
     +UID_MIN                  1000
     +UID_MAX                 60000
     +# System accounts
     +SYS_UID_MIN               201
     +SYS_UID_MAX               999
     +
     +#
     +# Min/max values for automatic gid selection in groupadd
     +#
     +GID_MIN                  1000
     +GID_MAX                 60000
     +# System accounts
     +SYS_GID_MIN               201
     +SYS_GID_MAX               999
     +
     +#
     +# If defined, this command is run when removing a user.
     +# It should remove any at/cron/print jobs etc. owned by
     +# the user to be removed (passed as the first argument).
     +#
     +#USERDEL_CMD      /usr/sbin/userdel_local
     +
     +#
     +# If useradd should create home directories for users by default
     +# On RH systems, we do. This option is overridden with the -m flag on
     +# useradd command line.
     +#
     +CREATE_HOME       yes
     +
     +# The permission mask is initialized to this value. If not specified, 
     +# the permission mask will be initialized to 022.
     +UMASK           077
     +
     +# This enables userdel to remove user groups if no members exist.
     +#
     +USERGROUPS_ENAB yes
     +
     +# Use SHA512 to encrypt password.
     +ENCRYPT_METHOD SHA512
```
  
### Step 12: Lock Inactive User Accounts
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_10.5_Lock_Inactive_User_Accounts" do
  title "Lock Inactive User Accounts"
  desc  "
    User accounts that have been inactive for over a given period of time can be automatically disabled. It is recommended that accounts that are inactive for 35 or more days be disabled.

    Rationale: Inactive accounts pose a threat to system security since the users are not logging in to notice failed login attempts or other anomalies.
  "
  impact 1.0
  describe bash("useradd -D | grep INACTIVE") do
    its("stdout") { should match(/^INACTIVE=35$/) }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
×  xccdf_org.cisecurity.benchmarks_rule_10.5_Lock_Inactive_User_Accounts: Lock Inactive User Accounts
     ×  Bash command useradd -D | grep INACTIVE stdout is expected to match /^INACTIVE=35$/
     expected "INACTIVE=-1\n" to match /^INACTIVE=35$/
     Diff:
     @@ -1 +1 @@
     -/^INACTIVE=35$/
     +INACTIVE=-1


  System Package telnetd
     ✔  is expected not to be installed

Profile Summary: 5 successful controls, 3 control failures, 0 controls skipped
Test Summary: 8 successful, 8 failures, 0 skipped
```
  
### Step 13: Check Permissions on User Home Directories
```ruby
control "xccdf_org.cisecurity.benchmarks_rule_13.7_Check_Permissions_on_User_Home_Directories" do
  title "Check Permissions on User Home Directories"
  desc  "
    While the system administrator can establish secure permissions for users' home directories, the users can easily override these.

    Rationale: Group or world-writable user home directories may enable malicious users to steal or modify other users' data or to gain another user's system privileges.
  "
  impact 1.0
  describe bash("for i in $(awk -F: '($7 != \"/usr/sbin/nologin\" && $3 >= 500) {print $6}' /etc/passwd | sort -u); do echo $i $(stat -L --format=%a $i) | grep -v ' .[0145][0145]$';done") do
    its("stdout") { should_not match(/.+/) }
  end
end
```
  
You can run your test with the following command
```bash
inspec exec . --json-config reporter.json
```
  
You will see an output in Chef Automate and on the STDOUT as follows: 
```bash
✔  xccdf_org.cisecurity.benchmarks_rule_13.7_Check_Permissions_on_User_Home_Directories: Check Permissions on User Home Directories
     ✔  Bash command for i in $(awk -F: '($7 != "/usr/sbin/nologin" && $3 >= 500) {print $6}' /etc/passwd | sort -u); do echo $i $(stat -L --format=%a $i) | grep -v ' .[0145][0145]$';done stdout is expected not to match /.+/

```
  
 ### 14. Center For Internet Security (CIS) Profile execution
The Centre for Internet Security produces a CIS CentOS, SUSE, Ubuntu and RHEL Benchmark.  
  
Chef has implemented those benchmarks uisng InSpec. We are now going to obtain that benchmark from Chef Automate and execute it against the AWS cloud. 

1. Login to Chef Automate via the terminal:  
  `inspec compliance login --insecure --user=workstation-<x> --token <Chef Automate Token> anthony-a2.chef-demo.com`   
   
  For example:  
  `inspec compliance login --insecure --user=workstation-1 --token AAAA-AAAA-AAAA-AAAAB anthony-a2.chef-demo.com`  
  ```
  Stored configuration for Chef Automate: https://anthony-a2.chef-demo.com/api/v0' with user: 'workstation-1'  
  ```
  
2. Open the Chrome Browser and in Chef Automate, go to the `Compliance` menu tab, then the `Profiles` tab on the left, see that the `CIS Red Hat Enterprise Linux 7 Benchmark Level 1 - Server` profile is available to your `workstation-x` user.  
  
  
3. Next lets execute that profile against the AWS API (replace `<x>` with your workstation number) - the tests will take about 4 minutes to run, some will emit a warning as the IAM role I am using does not have all of the required permissions, you can ignore these warnings:   
  `inspec exec compliance://workstation-<x>/cis-rhel7-level1-server --config=reporter.json` 
  
  Your output will be as follows:  
  ```bash
 ✔  xccdf_org.cisecurity.benchmarks_rule_6.2.16_Ensure_no_duplicate_UIDs_exist: Ensure no duplicate UIDs exist
     ✔  /etc/passwd uids is expected not to contain duplicates
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.2.17_Ensure_no_duplicate_GIDs_exist: Ensure no duplicate GIDs exist
     ✔  /etc/group gids is expected not to contain duplicates
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.2.18_Ensure_no_duplicate_user_names_exist: Ensure no duplicate user names exist
     ✔  /etc/passwd users is expected not to contain duplicates
  ✔  xccdf_org.cisecurity.benchmarks_rule_6.2.19_Ensure_no_duplicate_group_names_exist: Ensure no duplicate group names exist
     ✔  /etc/group groups is expected not to contain duplicates


  Profile Summary: 89 successful controls, 65 control failures, 34 controls skipped
  Test Summary: 623 successful, 190 failures, 37 skipped
  ```
  
4. Look at the scan results in the Chef Automate browser:   
![CIS RHEL Scan Results](/labs/images/linux_cis.png)
  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
