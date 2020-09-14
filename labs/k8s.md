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
  
  
  
[Back to the Lab Index](../README.md#cooking-up-compliance---workshop)
