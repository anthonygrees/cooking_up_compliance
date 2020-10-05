# Chef Products & Architecture
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
### Chef Product Stack
  
![Architecture Stack](/labs/images/architecture_stack.png "Architecture Stack")
  
The Chef Products are:
 - ```Chef Workstation``` is the location where users interact with Chef Infra. With Chef Workstation, users can author and test cookbooks using tools such as Test Kitchen and interact with the Chef Infra Server using the knife and chef command line tools.
 - ```Chef Infra Client``` nodes are the machines that are managed by Chef Infra. The Chef Infra Client is installed on each node and is used to configure the node to its desired state.
 - ```Chef Infra Server``` acts as a hub for configuration data. Chef Infra Server stores cookbooks, the policies that are applied to nodes, and metadata that describes each registered node that is being managed by Chef. Nodes use the Chef Infra Client to ask the Chef Infra Server for configuration details, such as recipes, templates, and file distributions.
 - ```Chef Automate``` provides a suite of enterprise capabilities for node visibility, and compliance, and integrates with other products like Chef Infra, Chef InSpec, and Chef Habitat.
  - ```Chef Habitat``` provides automation capabilities for defining, packaging and delivering applications to almost any environment regardless of operating system or deployment platform. 
  - ```Chef InSpec``` is a framework for testing and auditing your applications and infrastructure. Chef InSpec works by comparing the actual state of your system with the desired state that you express in easy-to-read and easy-to-write Chef InSpec code. Chef InSpec detects violations and displays findings in the form of a report, but puts you in control of remediation.
  
### Chef High Level Architecture
  
  
![Architecture Diagram](/labs/images/architecture_diagram.png "Architecture Diagram")
  
- Estimate how many times a day your nodes will check in and whether you will use the ‘Audit’ cookbook
- Determine if multiple data centers are required
- Determine if High Availability will be needed
- Create an architecture diagram with the following documented:
-- All relevant VLANs/Firewall rules
-- All required servers and their hardware specifications 
- Document a plan for provisioning servers and verifying network/hardware requirements 
- Document procedures for Disaster Recovery and monitoring
- Verify you have both human and hardware resources allocated to execute the plans
- Calculate your CCR/min (chef-client runs per minute). For example, 500 nodes set to check in every 30 minutes is equivalent to 16.66 CCRs/min.


  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
