# Chef Products & Architecture
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
### Create Multi Tenants in Chef Automate

As an Admin user in Chef Automate, go to the ```settings``` tab and  perform the following:

```Projects```
1. Create a Project called `agency-1-project`
2. Create a Project called `agency-2-project`
Under the projects, create ```Rules```
1. In the Project create a Rule for Nodes
2. Rule = PolicyGroup = `agency-1`
  
Click `Update Projects` to apply the changes
  
  
```Users```
1. Add a new user - `agency-1-user`
2. Add a new user - `agency-2-user`
  
  
```Teams```
1. Create a team called `agency-1-team`
2. Assign `agency-1-team` to `agency-1-project`
3. Add a User to the team - Add `agency-1-user`

1. Create a team called `agency-2-team`
2. Assign `agency-2-team` to `agency-2-project`
3. Add a User to the team - Add `agency-2-user`
  
  
```Policies```
1. Open the Policy and add `agency-1-project` Project Editors
2. Add a Team as a member of the policy- Add team `agency-1-team`
  
1. Open the Policy and add `agency-2-project` Project Editors
2. Add a Team as a member of the policy- Add team `agency-2-team`
  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
