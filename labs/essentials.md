## Chef Essentials
![Chef Infra](/images/ChefInfra.png)
  
### Student Slides
The slides used in this workshop cen be found here in a PDF format.
  
| Nbr | Module | Link |
|---|---|---|
| 1 | Introduction to Chef | [Click Here](https://github.com/anthonygrees/compliance-workshop/blob/master/slides/01-Introduction.pdf)
| 2 | Chef Resources | [Click Here](https://github.com/anthonygrees/compliance-workshop/blob/master/slides/03-Resources.pdf)
| 3 | Creating a Chef Cookbook | [Click Here](https://github.com/anthonygrees/compliance-workshop/blob/master/slides/04-creating-web-server-cookbook.pdf)|
| 4 | Capture Details about the System | [Click Here](https://github.com/anthonygrees/compliance-workshop/blob/master/slides/05-details-about-the-system.pdf)|
  
[Click Here for the Slides](https://github.com/anthonygrees/compliance-workshop/tree/master/slides)
  
### Helpful Links
  
#### Linux
 - Module 4 - The ```.kitchen.yml``` for the ```Linux webserver cookbook``` 
   
 ```yaml
 ---
driver:
  name: ec2
  aws_ssh_key_id: <%= ENV['AWS_KEYPAIR'] %>
  region: <%= ENV['AWS_REGION'] %>
  security_group_ids: 
    - <%= ENV['AWS_SECURITY_GROUP_ID'] %>
  subnet_id: <%= ENV['AWS_SUBNET_ID'] %>
  vpc_id: <%= ENV['AWS_VPC_ID'] %>
  associate_public_ip: true
  instance_type: t2.micro
  tags:
    # Replace YOURNAME and YOURCOMPANY here
    Name: "Chef Training Node for <YOURNAME>, <%= ENV['TRAINER_NAME'] %>"
    user: Administrator
    X-Contact: <%= ENV['TRAINER_NAME'] %>
    X-Application: "Training"
    X-Dept: "sales"
    X-Customer: <%= ENV['TRAINER_NAME'] %>
    X-Project: "Student_Lab"
    X-TTL: 4

provisioner:
  name: chef_zero
  product_name: chef
  chef_license: accept

verifier:
  name: inspec
  format: documentation

platforms:
  - name: centos-6
    transport:
      username: centos
      ssh_key:  ~\.ssh\id_rsa
    driver_config:
      user_data: C:/Users/chef/user_data

suites:
  - name: default
    run_list:
      - recipe[webserver::default]
    verifier:
      inspec_tests:
      - test/integration/default
    attributes:
 ```
  
 - Module 4 - The ```default_test.rb``` test examples  
   
 ```ruby
 # # encoding: utf-8

# Inspec test for recipe webserver::default

# The Inspec reference, with examples and extensive documentation, can be
# found at http://inspec.io/docs/reference/resources/

# This is an example test, replace it with your own test.
describe port(8080) do
  it { should be_listening }
end

# The use of curl is a terrible way to test!
describe command('curl localhost:8080') do
  its('stdout') { should match ('Hello, world')}
end

describe http('http://localhost:8080') do
  its('status') { should cmp 200 }
  its('body') { should eq '<h1>Hello, world!</h1>\n' }
end


control 'Linux VERSION' do
  impact 0.8
  title 'This test checks for a minimum Linux version'

  describe os.family do
    it { should eq 'redhat' }
  end

  describe os.name do
    it { should eq 'centos' }
  end

  describe os.release do
    it { should > '6' }
  end
end
 ```
  
 #### Windows
 - Module 4 - The ```.kitchen.yml``` for the ```Windows webserver cookbook```  
   
 ```yaml
 ---
driver:
  name: ec2
  aws_ssh_key_id: <%= ENV['AWS_KEYPAIR'] %>
  region: <%= ENV['AWS_REGION'] %>
  security_group_ids: 
    - <%= ENV['AWS_SECURITY_GROUP_ID'] %>
  subnet_id: <%= ENV['AWS_SUBNET_ID'] %>
  vpc_id: <%= ENV['AWS_VPC_ID'] %>
  associate_public_ip: true
  instance_type: m3.medium ##t2.micro
  tags:
    # Replace YOURNAME and YOURCOMPANY here
    Name: "Chef Training Node for <YOURNAME>, <%= ENV['TRAINER_NAME'] %>"
    created-by: "test-kitchen"
    user: administrator
    X-Contact: <%= ENV['TRAINER_NAME'] %>
    X-Application: "Training"
    X-Dept: "Sales"
    X-Customer: <%= ENV['TRAINER_NAME'] %>
    X-Project: "Student_Lab"
    X-TTL: 4
    
transport:
    username: administrator
    ssh_key:  ~\.ssh\id_rsa
    communicator: winrm

provisioner:
  name: chef_zero
  product_name: chef
  chef_license: accept

verifier:
  name: inspec
  format: documentation

platforms:
  - name: windows-2012r2
# - name: windows-2016

suites:
  - name: default
    run_list:
      - recipe[myiis::server]
    verifier:
      inspec_tests:
        - test/integration/default
    attributes:
 ```
  
 - Module 4 - The ```default_test.rb``` test examples  
   
 ```ruby
 # # encoding: utf-8

# Inspec test for recipe myiis::default

# The Inspec reference, with examples and extensive documentation, can be
# found at http://inspec.io/docs/reference/resources/

# Test that Port 80 is open
describe port(80) do
  it { should be_listening }
end

# Test that you web page says Hello, world Windows!
describe command('Invoke-WebRequest http://localhost') do
  its(:stdout) { should match /Hello, world Windows!/ }
end

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
    it { should eq 'windows_server_2012_r2_standard' }
  end

  describe os.release do
    it { should > '6.2' }
  end
end
```
  
#### Test-Kitchen Drivers for many Platforms
A Test-Kitchen ```driver``` is what supports configuring the compute instance that is used for isolated testing. This is typically a local hypervisor, hypervisor abstraction layer (Vagrant), or cloud service (EC2).
  
ChefDK / Chef Workstation include:
- [Vagrant](https://github.com/test-kitchen/kitchen-vagrant)
- [Kitchen Azurerm](https://github.com/test-kitchen/kitchen-azurerm)
- [Kitchen AWS EC2](https://github.com/test-kitchen/kitchen-ec2)
- [Kitchen HyperV](https://github.com/test-kitchen/kitchen-hyperv)
- [Kitchen Google Compute Engine](https://github.com/test-kitchen/kitchen-google)
- [Kitchen DigitalOcean](https://github.com/test-kitchen/kitchen-digitalocean)
- [Kitchen Doken for Docker Containers](https://github.com/someara/kitchen-dokken)
  
VMware Drivers:
- [kitchen-vra](https://github.com/chef-partners/kitchen-vra)
- [kitchen-vro](https://github.com/chef-partners/kitchen-vro)
- [kitchen-vcenter](https://github.com/chef/kitchen-vcenter)
  
Other Chef and Community Drivers:
- [Kitchen Terraform](https://github.com/newcontext-oss/kitchen-terraform)
- [OpenStack](https://github.com/test-kitchen/kitchen-openstack)
- [Docker](https://github.com/test-kitchen/kitchen-docker)
  
  
### Extra Learning and Videos
  
1. [Chef Software Open Source Communities](https://github.com/chef/chef-oss-practices) - This is a starting point for contributing to all of Chef's software and a wonderful spot for information on how to join in on the fun.
2. [DevOps Kung fu](https://github.com/chef/devops-kungfu) - This repository defines Chef Style DevOps Kung fu, which explains what DevOps is, and defines a style of practice that comes from the lived experience of many DevOps professionals.
3. [Chef Style DevOps Kungfu](https://www.youtube.com/watch?v=_DEToXsgrPc) - Adam Jacob Keynote - ChefConf 2015
4. [Eggs 365](https://www.youtube.com/watch?v=XD0vRW4G82U&t=2s) - ChefConf 2015: How is your Continuous Delivery Kung Fu?
5. [DevOps: No Horse Sh**](https://www.youtube.com/watch?v=0P0HD5pE-zU) - Original (language warning) - ChefConf 2014: Nathen Harvey knows DevOps, not farming. In this video, Nathen and his feathered and hooved friends explain how DevOps is (and isn't) like a closed-loop farm. 
