## InSpec on Linux
InSpec is an open-source testing framework for infrastructure with a human and machine-readable language for specifying compliance, security and policy requirements.

Don't have InSpec installed?  Here you go - https://downloads.chef.io/inspec

Need the code? You can find it here - https://github.com/anthonygrees/compliance-linux

### Step 1: Check your InSpec Version
Run the following command to check your InSpec version.
```bash
$ inspec --version
```
![InSpec Version](/images/1inspecversion.png)

### Step 2: Determine your platform
Use the InSpec ```detect``` command to check the platform you are running on.  Replace the ```999.999.999.999``` with the IP address given to you by the instructor.
```bash
inspec detect -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa
```
or
```
inspec detect -t ssh://centos@ec2-54-190-15-224.us-west-2.compute.amazonaws.com -i C:\Users\chef\.ssh\id_rsa
```
![Platform](/images/l1.png)

### Step 3: Let's look at InSpec Shell
It's a pry-based Read–Eval–Print Loop that can be used to quickly run InSpec controls and tests without having to write it to a file. Its functionality is similar to ```chef-shell```.

We'll start by connecting to the shell with our key and transport information, and then play around in the shell for a bit and write our first test.
```bash
$ inspec help shell
$ inspec shell -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa

inspec> help
inspec> help resources
inspec> help sshd_config
inspec> sshd_config.params
inspec> sshd_config.port
inspec> sshd_config.Protocol
inspec> help matchers
```
Ok.  Now try writing a control based test.
```ruby
inspec> describe sshd_config do
inspec> its('Protocol') { should eq '2' }
inspec> end
```
![Shell](/images/l2.png)

Type ```exit``` to leave the shell.
```ruby
inspec> exit
```

### Step 4: Check for insecure protocol
Now, create a new InSpec profile
```bash
## Create a New InSpec Profile
$ inspec init profile <your_profile_name>

## Change Directory into your new Profile
$ cd <your_profile_name>

## Write your Controls in the example.rb
$ code controls\example.rb
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa
```
![Telnet](/images/l3.png)

### Step 5: Ensure telnet server is not enabled
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa
```
![Telnet Enabled](/images/l4.png)

### Step 6: Report in Chef Automate
Create a file called ```inspec.json``` in <your_profile_name> directory and add the following:

Note: Remember to update the ```json``` and put your name in ```"node_name" : "<YOUR_NAME_HERE>"``` and add your ```TOKEN``` from the spreadsheet.

```json
{
    "reporter" : {
        "automate" : {
            "stdout" : "false",
            "url" : "https://anthony-a2.chef-demo.com/data-collector/v0",
            "token" : "w-rPchmGDbqbYkDWzhflw8bAvYs=",
            "insecure" : true,
            "node_name" : "<YOUR_NAME_HERE>",
            "node_uuid" : "12345678-1234-1234-1234-123456789012",
            "environment" : "dev"
        }
    }
}
```

To execute this using InSpec and report to A2 run the following command

```bash
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Report](/images/l6.png)

### Step 7: Ensure FTP Server is not enabled

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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![FTP](/images/l7.png)

### Step 8: Ensure shadow group is empty
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Shadow](/images/l8.png)

### Step 9: Ensure tftp-server is not enabled
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
# run test on remote host on SSH
inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![tFTP](/images/l9.png)

### Step 10: Ensure DHCP Server is not enabled
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![DHCP](/images/l10.png)

### Step 11: Set Password Creation Requirement Parameters
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Pass](/images/l11.png)

### Step 12: Set Password Expiration Days
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Expire](/images/l12.png)

### Step 13: Lock Inactive User Accounts
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Lock](/images/l13.png)

### Step 14: Check Permissions on User Home Directories
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
# run test on remote host on SSH
$ inspec exec . -t ssh://ubuntu@999.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json
```
![Permissions](/images/l14.png)


--------------------
## UUID Error
If you get a node uuid error, then create a file called ```inspec.json```

```
{
    "reporter": {
        "cli" : {
            "stdout" : true,
            "node_uuid" : "<yourname>-1234"
        }
    }
}
```

And execute with the following command

```inspec exec . -t ssh://centos@99.999.999.999 -i C:\Users\chef\.ssh\id_rsa --json-config inspec.json```

Otherwise you can create a UUID for InSpec when you create your node:
```
file '/etc/machine-id' do
  content '12345678-1234-1234-1234-123456789012'
end
```
