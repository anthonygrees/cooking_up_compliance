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
  
Work within the home directory.  
```bash
cd c:\chef-repo\cookbooks
```
  
Create your recipe
```bash
code hello.rb
```
  
In your recipe add the following resource. 
```ruby
file 'C:\hello.txt' do
  content 'Hello, world!'
end

```
  
  
### How do we run our Recipe ?
The chef-client is an agent that runs locally on every node that is under management by Chef.  
  
When a chef-client is run, it will perform all of the steps that are required to bring the node into the expected state.  
  
https://docs.chef.io/chef_client.html
  
  
Apply the recipe with the following command.  
```bash
chef-apply hello.rb -l info
```
Note: chef-apply is an executable program that runs a single recipe from the command line.  
  
  
Your output will look like this:  
```bash
PS C:\chef-repo\cookbooks> chef-apply hello.rb -l info
[2020-10-05T06:27:38+00:00] INFO: Run List is []
[2020-10-05T06:27:38+00:00] INFO: Run List expands to []
Recipe: (chef-apply cookbook)::(chef-apply recipe)
  * file[C:\hello.txt] action create[2020-10-05T06:27:38+00:00] INFO: Processing file[C:\hello.txt] action create ((chef-apply cookbook)::(chef-apply recipe) line 1)
[2020-10-05T06:27:38+00:00] INFO: file[C:\hello.txt] created file C:\hello.txt

    - create new file C:\hello.txt[2020-10-05T06:27:38+00:00] INFO: file[C:\hello.txt] updated file contents C:\hello.txt

    - update content in file C:\hello.txt from none to 315f5b
    --- C:\hello.txt    2020-10-05 06:27:38.772146400 +0000
    +++ C:\chef-hello20201005-3744-legk29.txt   2020-10-05 06:27:38.771151100 +0000
    @@ -1 +1,2 @@
    +Hello, world!
```
  
  
Check that the file was created with the text.  
```bash
gc c:\hello.txt
```
  
Your output will look like this:  
```bash
PS C:\chef-repo\cookbooks> gc c:\hello.txt
Hello, world!
```
  
  
### Is it safe to run multiple times ?
  
  
Let's find out. Apply the recipe AGAIN with the following command.  
```bash
chef-apply hello.rb -l info
```
  
Your output will look like this:  
```bash
PS C:\chef-repo\cookbooks> chef-apply hello.rb -l info
[2020-10-05T06:32:10+00:00] INFO: Run List is []
[2020-10-05T06:32:10+00:00] INFO: Run List expands to []
Recipe: (chef-apply cookbook)::(chef-apply recipe)
  * file[C:\hello.txt] action create[2020-10-05T06:32:10+00:00] INFO: Processing file[C:\hello.txt] action create ((chef-apply cookbook)::(chef-apply recipe) line 1)
 (up to date)
```
  
Yes, Chef recognises there are NO changes to be made.  
