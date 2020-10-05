# About Chef Resources and Recipes
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
### Chef Resources
A resource is a statement of configuration policy.  
  
It describes the desired state of an element of your infrastructure and the steps needed to bring that item to the desired state.  
  
https://docs.chef.io/resources.html
  
Examples
```ruby
# The package named 'httpd' is installed.
package 'httpd' do
  action :install
end

# The service named 'ntp' is enabled (start on reboot) and started.
service 'ntp' do
  action [ :enable, :start ]
end

# The file name '/etc/motd' is created with content 'This computer is the property ...'
file '/etc/motd' do
  content 'This computer is the property...'
end

# The file name '/etc/php.ini.default' is deleted.
file '/etc/php.ini.default' do
  action :delete
end

```
  
### Create your first Recipe
  
