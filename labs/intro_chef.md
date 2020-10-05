# Chef - Infrastructure as Code
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
### About Chef ?
Chef can automate how you build, deploy, and manage your infrastructure.
  
Chef can integrate with infrastructure and cloud platforms to automatically provision and configure new machines such as:
- Bare Metal
- VMware
- Hyper-V
- AWS
- Azure
- Google Cloud
- AliCloud
- Docker
  
Chef is a large set of tools that are able to be used on multiple platforms and in numerous configurations. 
  
Learning Chef is like learning a language. You will learn the basic concepts very fast but it will take practice until you become comfortable.
  
A great way to learn Chef is to use Chef !
  
### What actually is Chef ?
Chef is....
- An automation framework that enables Infrastructure as Code
- A robust set of tooling for testing Chef code
- A large library of reusable patterns (supermarket.chef.io)
- Available for Linux variants, Unix variants, and Windows, all as first class citizens.
  
### DSL - Domain Specific Language
Chef is a DSL
- Programmatically provision and configure components
- Declarative DSL with the flexibility 
- Built on Ruby
- Extensible through Ruby
  
therefore you treat it like any programing language and ensure Chef is:
- Stored in source control like `git`
- Testing Coverage
- Part of your CI or CD pipelines
  
### Chef Core Concepts
![Chef](/labs/images/chef_core_concepts.png)
  
  
#### What is a Resource ?
- A Resource is a system state you define, for example: Package installed, state of a service, configuration file existing.
- You declare what the state of the resource is. Chef will automatically determine HOW that state is achieved.
  
A Linux example
```ruby
package 'httpd'
```
  
or
  
```ruby
package 'Install Apache' do
  case node[:platform]
  when 'redhat', 'centos'
    package_name 'httpd'
  when 'ubuntu', 'debian'
    package_name 'apache2'
  end
end
```
  
A Windows example
```ruby
windows_package '7zip' do
  action :install
  source 'C:\7z920.msi'
end
```
  
#### What is a Recipe ?
- A recipe is a collection of Resources
- Resources are executed in the order they are listed 
  
  
```ruby
execute "apt-get update" do
 command "apt-get update"
end

apt_package "vim" do
 action :install
end
```
  
#### What is a Cookbook ?
- A cookbook is a set of recipes
- A cookbook is a defined set of items and different outcomes that you expect to address
- Example: A cookbook could have a recipe to install apache2/httpd but also another set of recipes to activate modules required.

```bash
myiis/
├── Berksfile
├── LICENSE
├── README.md
├── chefignore
├── metadata.rb
├── recipes
│   ├── default.rb
│   └── server.rb
├── spec
│   ├── spec_helper.rb
│   └── unit
│       └── recipes
│           └── default_spec.rb
└── test
    └── integration
        └── default
            └── default_test.rb

7 directories, 10 files
```

  
  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
