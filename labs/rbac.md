# Chef Automate IAM - Role Based Access Controls
    
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
Full details of IAM/RBAC can be found at the [Chef Automate IAM Overview Docs Site](https://automate.chef.io/docs/iam-v2-overview/)
  
### RBAC Use Case
The Multi Tenant example below shows a Government Organisation with two Government Agencies.  
 - `Government Admin` User can see ALL VM's and the Compliance Reports for ALL Agencies
 - `Agency-1 User` can only see their own VM's and the Complaince Reports for their own Agency
 - `Agency-2 User` can only see their own VM's and the Complaince Reports for their own Agency
  
Projects (Tenants) can filter policy triggers for Events and Nodes based on:
`Events`
 - Chef Organization
 - Chef Server
  
`Nodes`
 - Chef Organization
 - Chef Server
 - Environment
 - Chef Role
 - Chef Tag
 - Chef Policy Name
 - Chef Policy Group
  
### Create Multi Tenants in Chef Automate

As an Admin user in Chef Automate, go to the ```settings``` tab and  perform the following:

```Projects```
1. Create a Project called `agency-1-project`
Under the projects, create ```Rules```
1. In the Project create a Rule for Nodes
2. Rule = PolicyGroup = `agency-1`
1. Create a Project called `agency-2-project`
Under the projects, create ```Rules```
1. In the Project create a Rule for Nodes
2. Rule = PolicyGroup = `agency-2`
  
![RBAC Projects](/labs/images/rbac_project.png "RBAC Projects")
  
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
![RBAC Team](/labs/images/rbac_team.png "RBAC Team")
  
  
```Policies```
1. Open the Policy and add `agency-1-project` Project Editors
2. Add a Team as a member of the policy- Add team `agency-1-team`
  
1. Open the Policy and add `agency-2-project` Project Editors
2. Add a Team as a member of the policy- Add team `agency-2-team`
![RBAC Policy](/labs/images/rbac_policy.png "RBAC Policy")
  
### Government Admin User
The Government Admin User can see ALL VM's and Compliance Reports and can filter based on Agency.
  
![RBAC Admin](/labs/images/rbac_admin.png "RBAC Admin")
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
