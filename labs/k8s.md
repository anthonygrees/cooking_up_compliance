![Docker](/labs/images/k8s.png)
# Scan Kubernetes with InSpec
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
  
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


### 2. Quick look at Kubernetes

1. Some basic K8s commands
```bash
# Get commands with basic output
kubectl get services                          # List all services in the namespace
kubectl get pods --all-namespaces             # List all pods in all namespaces
kubectl get pods -o wide                      # List all pods in the current namespace, with more details
kubectl get deployment my-dep                 # List a particular deployment
kubectl get pods                              # List all pods in the namespace
kubectl get pod my-pod -o yaml                # Get a pod's YAML

# Describe commands with verbose output
kubectl describe nodes my-node
kubectl describe pods my-pod
```
  
The output will look something like this:
```bash
[ec2-user@ip-172-31-54-198 inspec-labs]$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h24m

[ec2-user@ip-172-31-54-198 inspec-labs]$ kubectl get pods --all-namespaces 
NAMESPACE     NAME                                                                  READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6dfcd885bf-mmtsp                              1/1     Running   0          3h24m
kube-system   calico-node-trdjd                                                     1/1     Running   0          3h24m
kube-system   coredns-f9fd979d6-6zzjv                                               1/1     Running   0          3h24m
kube-system   coredns-f9fd979d6-w98rn                                               1/1     Running   0          3h24m
kube-system   etcd-ip-172-31-54-198.us-west-2.compute.internal                      1/1     Running   0          3h24m
kube-system   kube-apiserver-ip-172-31-54-198.us-west-2.compute.internal            1/1     Running   0          3h24m
kube-system   kube-controller-manager-ip-172-31-54-198.us-west-2.compute.internal   1/1     Running   0          3h24m
kube-system   kube-proxy-5cqsl                                                      1/1     Running   0          3h24m
kube-system   kube-scheduler-ip-172-31-54-198.us-west-2.compute.internal            1/1     Running   0          3h24m
```
  
### 3. Scan Kubernetes
  
1. Create a K8s InSpec profile. 
  
Run the following command.  
```bash
inspec init profile kube
```
  
Your output will look like this. (You may need to accept the license.):
```bash
[ec2-user@ip-172-31-54-198 inspec-labs]$ inspec init profile kube
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

 ─────────────────────────── InSpec Code Generator ─────────────────────────── 

Creating new profile at /home/ec2-user/inspec-labs/kube
 • Creating file README.md
 • Creating directory controls
 • Creating file controls/example.rb
 • Creating file inspec.yml
```
  
Change directory into the InSpec profile.  
```bash
cd kube
```
   
   
   
2. Update the `example.rb` in the `controls` directory
  
Delete the sample contents of `example.rb` and add the following Kubernetes control:
```ruby
control '5.6.4 - The default namespace should not be used' do
  impact 1.0
  title 'The default namespace should not be used'
  desc 'Kubernetes provides a default namespace, where objects are placed if no namespace is specified for them. Placing objects in this namespace makes application of RBAC and other controls more difficult.'
  tag level: 2

  only_if('Control only applies to the host for kubectl') { input('is_kubectl_host') }

  # CIS are imprecise on what resources are acceptable. This controls defaults to only allowing the kubernetes service
  describe json({command: 'kubectl get all -o json --namespace=default'}).items do
    its('length') { should cmp 1 }
  end

  describe json({command: 'kubectl get all -o json --namespace=default'}).items.first['kind'] do
    it { should eq 'Service' }
  end

  describe json({command: 'kubectl get all -o json --namespace=default'}).items.first['metadata']['name'] do
    it { should eq 'kubernetes' }
  end
end
```
   
   
   
3. Report in Chef Automate
You will need to create a UUID for your Linux scan, run `uuidgen` in your terminal.    
     
  You will see output as follows:
  ```bash
    [ec2-user@ip-172-31-54-168 aws]$ uuidgen
    c6754ceb-b9a8-4917-a797-c0d0589246d6
      
```
  
Next you need to create a `reporter.json` file like this in the `linux-example` directory, your instructor should have given you the Chef Automate Hostname and Token. Replace `<x>` with you workstation number. You can create the file by either:  
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
      "node_name" : "YourName-k8s-Demo",
      "environment" : "k8s",
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
[ec2-user@ip-172-31-54-198 kube]$ inspec exec . --json-config reporter.json
[2020-09-14T04:42:06+00:00] WARN: Input 'is_kubectl_host' does not have a value. Use --input-file or --input to provide a value for 'is_kubectl_host' or specify a  value with `input('is_kubectl_host', value: 'somevalue', ...)`.
[2020-09-14T04:42:06+00:00] WARN: Input 'is_kubectl_host' does not have a value. Use --input-file or --input to provide a value for 'is_kubectl_host' or specify a  value with `input('is_kubectl_host', value: 'somevalue', ...)`.

Profile: InSpec Profile (kube)
Version: 0.1.0
Target:  local://

  ✔  5.6.4 - The default namespace should not be used: The default namespace should not be used
     ✔  [{"apiVersion"=>"v1", "kind"=>"Service", "metadata"=>{"creationTimestamp"=>"2020-09-14T00:12:18Z", "labels"=>{"component"=>"apiserver", "provider"=>"kubernetes"}, "managedFields"=>[{"apiVersion"=>"v1", "fieldsType"=>"FieldsV1", "fieldsV1"=>{"f:metadata"=>{"f:labels"=>{"."=>{}, "f:component"=>{}, "f:provider"=>{}}}, "f:spec"=>{"f:clusterIP"=>{}, "f:ports"=>{"."=>{}, "k:{\"port\":443,\"protocol\":\"TCP\"}"=>{"."=>{}, "f:name"=>{}, "f:port"=>{}, "f:protocol"=>{}, "f:targetPort"=>{}}}, "f:sessionAffinity"=>{}, "f:type"=>{}}}, "manager"=>"kube-apiserver", "operation"=>"Update", "time"=>"2020-09-14T00:12:18Z"}], "name"=>"kubernetes", "namespace"=>"default", "resourceVersion"=>"158", "selfLink"=>"/api/v1/namespaces/default/services/kubernetes", "uid"=>"b42c6d97-7b9d-4fe7-8516-71d4c78a143e"}, "spec"=>{"clusterIP"=>"10.96.0.1", "ports"=>[{"name"=>"https", "port"=>443, "protocol"=>"TCP", "targetPort"=>6443}], "sessionAffinity"=>"None", "type"=>"ClusterIP"}, "status"=>{"loadBalancer"=>{}}}] length is expected to cmp == 1
     ✔  Service is expected to eq "Service"
     ✔  kubernetes is expected to eq "kubernetes"


Profile Summary: 1 successful control, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
  
![k8s](/labs/images/k8s1.png)
  
  
  
4. Create an attributes `inputs.yml` file
  
Run the command `touch inputs.yml` and add the following to the new file:
  
```yaml
is_cluster_master: true
is_etcd_master: true
is_worker: true
is_kubectl_host: true
```
  
  
5. Login to the Chef Automate server
  
Login to the A2 server to allow compliance profiles to be downloaded.  
```bash
inspec compliance login --insecure --user=delivery --token m6E8BQ5iCWLMBFUpIPRlRhrqR6k= aut-automate-server
```
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
