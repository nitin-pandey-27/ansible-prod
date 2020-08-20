# ANSIBLE MULTI MASTER

Please check the file pre-requisites. 

Always run the command -v flag 

Check file - k8s-ansible-multi-master - for TEXT format. 

# STEPS OF INSTALLATION 

ansible-playbook -v -i inventory/mycluster playbooks/kubectl.yaml

- import_playbook: ntpd.yaml				---> 01st - install this first 
- import_playbook: docker.yaml				---> 02nd - install docker 
- import_playbook: iptables.yaml			---> 03rd - allow rules 
- import_playbook: kubectl.yaml				---> 04th - ansible-playbook -v -i inventory/mycluster playbooks/kubectl.yaml
- import_playbook: certs.yaml				---> 05th - generate certificates 
- import_playbook: config-certs.yaml		---> 06th - generate certificates from conf files 
- import_playbook: config-files.yaml		---> 07th - generate config files 
- import_playbook: distribute.yaml			---> 08th - distribute config files to the servers  
- import_playbook: etcd-cluster.yaml		---> 09th - install and start etcd cluster 
- import_playbook: k8s-control-plane.yaml	---> 10th - install and start control plane components 
- import_playbook: load_balancer.yaml		---> 10th - install load balancer 
- import_playbook: k8s-worker-node.yaml		---> 11th - install and start worker node components  / certificates 
- import_playbook: networking.yaml			---> 12th - Download CNI Plugins  
- import_playbook: dns-networking.yaml 		---> 13th - Start Networking pods and DNS pods 
- import_playbook: rbac-permissoin.yaml 	---> 14th - configure RBAC permissions




# FOLLOW PRE-REQUISITES FILE TO CONFIRM THAT SSH KEYS HAVE BEEN CREATED. 
# YOU SHOULD ALSO CHECK THAT PASSWORDLESS SSH IS WORKING FROM - WORKSTATION - TO ALL OTHER NODES. 
# ADD HOSTNAMES ON ALL THE /etc/hosts FILE OF ALL THE HOSTS.




# EDIT MYCLUSTER AS PER YOUR ENVIRONMENT 
- vi inventory/mycluster




# INSTALL AND CONFIGURE NTPD ON ALL NODES 
- import_playbook: ntpd.yaml	

Edit the file - roles/ntpd/templates/ntp.conf - to change the NTPD server IP. 		

-->this will apply the playbook 	
#ansible-playbook -v -i inventory/mycluster playbooks/ntpd.yaml 

-->to verify if playbook have been applied or not 		
#ansible all -i inventory/mycluster -m shell -a "/usr/bin/date"	
OUTPUT - 
master02 | CHANGED | rc=0 >>
Thu Aug 20 16:29:02 +04 2020
master01 | CHANGED | rc=0 >>
Thu Aug 20 16:29:03 +04 2020
worker02 | CHANGED | rc=0 >>
Thu Aug 20 16:29:02 +04 2020
worker01 | CHANGED | rc=0 >>
Thu Aug 20 16:29:02 +04 2020
loadbalancer | CHANGED | rc=0 >>
Thu Aug 20 16:29:03 +04 2020
etcd01 | CHANGED | rc=0 >>
Thu Aug 20 16:29:06 +04 2020
etcd02 | CHANGED | rc=0 >>
Thu Aug 20 16:29:06 +04 2020

-->If you only want to sync the time 
#ansible-playbook -v -i inventory/mycluster playbooks/ntpd.yaml  --tags sync-time  
        
		
		
		

# INSTALL DOCKER ON ALL THE NODES 
- import_playbook: docker.yaml				

-->this will apply the playbook
#ansible-playbook -v -i inventory/mycluster playbooks/docker.yaml		

-->this will verify if playbook have been applied or not 					
#ansible all -i inventory/mycluster -m shell -a "docker version" | grep -C 2 CHANGED			
OUTPUT -
worker02 | CHANGED | rc=0 >>






# ENABLE FORWARDING RULE ON ALL NODES 
- import_playbook: iptables.yaml		

-->this will apply the playbook 	
#ansible-playbook -v -i inventory/mycluster playbooks/iptables.yaml	

-->this will verify the playbook execution 						
#ansible all -i inventory/mycluster -m shell -a " cat /etc/sysctl.d/k8s.conf"				
OUTPUT - 
master01 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
master02 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
worker02 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
worker01 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
loadbalancer | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
etcd01 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
etcd02 | CHANGED | rc=0 >>
sysctl net.bridge.bridge-nf-call-iptables=1
...
...
...
... For all other nodes 








# INSTALL KUBECTL ON ALL NODES 
- import_playbook: kubectl.yaml			

-->this will apply the playbook	
#ansible-playbook -v -i inventory/mycluster playbooks/kubectl.yaml	

-->this will verify the playbook execution 						 
#ansible all -i inventory/mycluster -m shell -a "/usr/local/bin/kubectl version --client" 	
OUTPUT 
master01 | CHANGED | rc=0 >>
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.0", GitCommit:"ddf47ac13c1a9483ea035a79cd7c10005ff21a6d", GitTreeState:"clean", BuildDate:"2018-12-03T21:04:45Z", GoVersion:"go1.11.2", Compiler:"gc", Platform:"linux/amd64"}
worker02 | CHANGED | rc=0 >>
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.0", GitCommit:"ddf47ac13c1a9483ea035a79cd7c10005ff21a6d", GitTreeState:"clean", BuildDate:"2018-12-03T21:04:45Z", GoVersion:"go1.11.2", Compiler:"gc", Platform:"linux/amd64"}
worker01 | CHANGED | rc=0 >>
...
...
...
... For all other nodes 





# GENERATE CERTS ON WORKSTATION
- import_playbook: certs.yaml			

-->this will apply the playbook.
#ansible-playbook -v -i inventory/mycluster playbooks/certs.yaml	

-->this will verify the playbook execution. You will see certs listed in this directory. 							
#ansible workstation -i inventory/mycluster -m shell -a "ls -lrth /root/k8s-hard-way/certs/"			 
OUTPUT> You will see all the certificates listed 




# GENERATE CONFIG FILES & CERTS ON WORKSTATION
- import_playbook: config-certs.yaml
	
Edit  below lines in - roles/config-certs/tasks/main.yaml file 

     25      IP.1 = 192.168.1.162		#master01 IP
     26      IP.2 = 192.168.1.218		#master02 IP. Please add IP.3 and so on for more master hosts 
     27      IP.3 = 192.168.1.60		#load balancer or ha proxy IP 
	 28      IP.4 = 10.96.0.1			#service cluster IP range. Important else , weave pod won't start 
     29      IP.5 = 127.0.0.1			
	 
	 54      IP.1 = 192.168.1.63		# etcd01 host 
     55      IP.2 = 192.168.1.182		# etcd02 host . Please add IP.3 / IP.4 and so on for more ETCD hosts 
										# can have ETCD installed on master host or different hosts 
     56      IP.3 = 127.0.0.1			

-->this will apply the playbook
#ansible-playbook -v -i inventory/mycluster playbooks/config-certs.yaml 				

-->this will verify the playbook execution 				 
#ansible workstation -i inventory/mycluster -m shell -a "ls -lrth /root/k8s-hard-way/certs/"		
OUTPUT> You will see all the certificates listed 

-->if you want to apply the playbook only for kube-apiserver. No need to execute this unless you want to re-execute the playbook only for kube-apiserver. 
#ansible-playbook -v -i inventory/mycluster playbooks/config-certs.yaml --tags=kube-apiserver





# GENERATE KUBECONFIG FILES & CERTS  ON WORKSTATION
- import_playbook: config-files.yaml	
Edit  below lines in - roles/config-files/tasks/main.yaml file 	
  9      export LOADBALANCER_ADDRESS=192.168.1.60				---> enter load balancer or haproxy IP
  29     export LOADBALANCER_ADDRESS=192.168.1.60				---> enter load balancer or haproxy IP
  50     export LOADBALANCER_ADDRESS=192.168.1.60				---> enter load balancer or haproxy IP
  71     export LOADBALANCER_ADDRESS=192.168.1.60				---> enter load balancer or haproxy IP
  
  
-->this will apply the playbook
#ansible-playbook -v -i inventory/mycluster playbooks/config-files.yaml 		

-->this will show the encryption file						
#cat /root/k8s-hard-way/certs/encryption-config.yaml		

										 	 

 


# DSITRIBUTE THE FILES
- import_playbook: distribute.yaml	
This will distribute all the files generated so far on master and worker nodes.
Edit below lines in - roles/distribute/tasks/main.yaml

4    for instance in master01 master02; do						--> enter all the master hosts 
16   for instance in worker01 worker02;	do					    --> enter all the worker hosts 
25   for instance in etcd01 etcd02; do 							--> enter all the etcd hosts 

-->this will apply the playbook 
#ansible-playbook -v -i inventory/mycluster playbooks/distribute.yaml							  	

-->verify if files have been copied on all the master nodes 
#ansible dc1-k8s-masters -i inventory/mycluster -m shell -a "ls -lrth /root/*" 						

-->verify if files have been copied on all the worker nodes 
#ansible dc1-k8s-workers-vm -i inventory/mycluster -m shell -a "ls -lrth /root/*" 					

-->verify if files have been copied on all the ETCD nodes 
#ansible dc1-k8s-etcd-vm -i inventory/mycluster -m shell -a "ls -lrth /root/*" 						

-->Distribute the copy only to master. No need to execute this unless you want to re-execute the playbook only for kube-apiserver.  
#ansible-playbook -v -i inventory/mycluster playbooks/distribute.yaml --tags=kube-apiserver






# BOOTSTRAP ETCD CLUSTER 	 
- import_playbook: etcd-cluster.yaml	

 
-->This will support both cluster , either ETCD installed on master nodes or installed on different servers.

Edit below lines in - roles/etcd-cluster/tasks/main.yaml. 

####Change ETCD version 
	4		/usr/bin/wget "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"  
	
####Change ethernet port 
    18      /usr/sbin/ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1

####master01 / master02 or etcd01 / etcd02 - should contain the name/ip where you want ETCD cluster to run  
	63      --initial-cluster    master01=https://192.168.1.162:2380,master02=https://192.168.1.218:2380 \  


-->this will apply the playbook 
#ansible-playbook -v -i inventory/mycluster playbooks/etcd-cluster.yaml						
	
-->This command will list the etcd servers. Run this from all ETCD nodes. 
#ETCDCTL_API=3 etcdctl member list --endpoints=https://127.0.0.1:2379  --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key
OUTPUT 
1bb85f91c96af39d, started, etcd02, https://192.168.1.174:2380, https://192.168.1.174:2379
9f5e0acc1f346641, started, etcd01, https://192.168.1.101:2380, https://192.168.1.101:2379



![alt text](https://github.com/nitin-pandey-27/ansible-prod/blob/master/external-etcd-cluster.png)

![alt text](https://github.com/nitin-pandey-27/ansible-prod/blob/master/stacked-etcd-cluster.png)



# BOOTSTRAP CONTROL PLANE 
- import_playbook: k8s-control-plane.yaml	

Edit below line in - roles/k8s-control-plane/tasks/main.yaml. 
Enter correct ETCD server IP address. 

#####Change as per your ethernet 

62     /usr/sbin/ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1

#####Change as per your ETCD servers 

 104      --etcd-servers=https://192.168.1.63:2379,https://192.168.1.182:2379 \

-->this will apply the playbook
#ansible-playbook -v -i inventory/mycluster playbooks/k8s-control-plane.yaml					 

-->Run this from both master
#kubectl get componentstatuses --kubeconfig admin.kubeconfig	
OUTPUT 
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}	

-->If you want to run only for master. Only required for re executing it for kube-apiserver. 
#ansible-playbook -v -i inventory/mycluster playbooks/k8s-control-plane.yaml --tags=kube-apiserver	


--->From all the master nodes, check status of each services 
#systemctl status kube-apiserver -l
#systemctl status kube-scheduler -l
#systemctl status kube-controller-manager  -l


-->Error in kube-controller-manager , can be fixed by restarting it. 
#systemctl restart kube-controller-manager
ERROR01 - 
Aug 20 16:26:11 master01 kube-controller-manager[13362]: E0820 16:26:11.447834   13362 leaderelection.go:270] error retrieving resource lock kube-system/kube-controller-manager: Get https://127.0.0.1:6443/api/v1/namespaces/kube-system/endpoints/kube-controller-manager?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
Aug 20 16:26:23 master01 kube-controller-manager[13362]: E0820 16:26:23.988865   13362 leaderelection.go:270] error retrieving resource lock kube-system/kube-controller-manager: Get https://127.0.0.1:6443/api/v1/namespaces/kube-system/endpoints/kube-controller-manager?timeout=10s: net/http: TLS handshake timeout
Aug 20 16:26:37 master01 kube-controller-manager[13362]: E0820 16:26:37.168296   13362 leaderelection.go:270] error retrieving resource lock kube-system/kube-controller-manager: Get https://127.0.0.1:6443/api/v1/namespaces/kube-system/endpoints/kube-controller-manager?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)




-->Error in kube-scheduler can be fixed by - 
#systemctl status kube-scheduler -l 
ERROR01 - 
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.213321   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1.StatefulSet: statefulsets.apps is forbidden: User "system:kube-scheduler" cannot list resource "statefulsets" in API group "apps" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.220720   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1.ReplicaSet: replicasets.apps is forbidden: User "system:kube-scheduler" cannot list resource "replicasets" in API group "apps" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.317352   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1.StorageClass: storageclasses.storage.k8s.io is forbidden: User "system:kube-scheduler" cannot list resource "storageclasses" in API group "storage.k8s.io" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.421567   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1.Node: nodes is forbidden: User "system:kube-scheduler" cannot list resource "nodes" in API group "" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.429492   13436 reflector.go:134] k8s.io/kubernetes/cmd/kube-scheduler/app/server.go:232: Failed to list *v1.Pod: pods is forbidden: User "system:kube-scheduler" cannot list resource "pods" in API group "" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.614838   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1beta1.PodDisruptionBudget: poddisruptionbudgets.policy is forbidden: User "system:kube-scheduler" cannot list resource "poddisruptionbudgets" in API group "policy" at the cluster scope
Aug 20 16:27:04 master01 kube-scheduler[13436]: E0820 16:27:04.827145   13436 reflector.go:134] k8s.io/client-go/informers/factory.go:132: Failed to list *v1.PersistentVolume: persistentvolumes is forbidden: User "system:kube-scheduler" cannot list resource "persistentvolumes" in API group "" at the cluster scope


-->Get the clusterrole. Check all the clusterrole in the error file. Add any roles that is missing as per the "systemctl status kube-scheduler -l "
#kubectl get clusterrole system:kube-scheduler -o yaml > error-kube-scheduler.yaml
#vi error-kube-scheduler.yaml		
	## Add below lines 
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - get
  - list
  - watch	


-->Get the clusterrolebinding . Check all the clusterrolebinding in the error file. Add any roles that is missing as per the "systemctl status kube-scheduler -l "
#kubectl get clusterrolebinding system:kube-scheduler -o yaml > error-kube-scheduler-binding.yaml

#systemctl stop kube-scheduler
#kubectl apply -f error-kube-scheduler.yaml
#systemctl start kube-scheduler
#systemctl status kube-scheduler -l










# BOOTSTRAP LOAD BALANCER
- import_playbook: load_balancer.yaml		

Edit below lines in - 

	##### enter load balancer IP:Port
     53        bind 192.168.1.60:6443		 
	 
	##### enter master server's IP:Port
	 61        server master 192.168.1.162:6443 check fall 3 rise 2
     62        server node03 192.168.1.218:6443 check fall 3 rise 2
	 
	##### enter load balancer stats port 
	 64        bind 192.168.1.60:9000 # Listen on localhost:9000

-->this will apply the playbook
#ansible-playbook -v -i inventory/mycluster playbooks/load-balancer.yaml

-->To verify haproxy.conf file 
#curl  https://192.168.1.60:6443/version -k
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}[root@loadbalancer ~]#

-->To verify the haproxy UI. Goto this URL from browser - 
http://192.168.1.60:9000/haproxy_stats 
admin:password 








# BOOTSTRAP WORKER NODE 
- import_playbook: k8s-worker-node.yaml		

Edit role - roles/k8s-worker-node/tasks/main.yaml

#####Change network interface - 

    17     /usr/sbin/ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1
	
#####Change Load Balancer IP - 
    77     export LOADBALANCER_ADDRESS=192.168.1.60
	
#####Set CIDR as per your environment - 
    204      clusterCIDR: "192.168.1.0/24"
	
#####Change clusterDNS 
    146      clusterDNS:
    147      - "192.168.1.0"
    148      resolvConf: "/run/systemd/resolve/resolv.conf"

-->To distribute the certificates / config files to workers. Re run this command only for WORKER nodes. 
#ansible-playbook -v -i inventory/mycluster playbooks/distribute.yaml --tags worker 

-->Execute the playbook 
#ansible-playbook -v -i inventory/mycluster playbooks/k8s-worker-nodes.yaml

-->If you get any privilege escalation error use below - 
#ansible-playbook -v -i inventory/mycluster -b  playbooks/k8s-worker-nodes.yaml

-->To verify the the worker node, from master node run this 
#kubectl get nodes --kubeconfig admin.kubeconfig
OUTPUT 
NAME       STATUS     ROLES    AGE   VERSION
worker01   NotReady   <none>   16s   v1.13.0
worker02   NotReady   <none>   15s   v1.13.0
[root@master01 ~]#


-->From all the worker nodes, check the status of kubelet and kube-proxy service 
#systemctl status kubelet -l
#systemctl status kube-proxy -l 

-->ERROR01 
Please ignore this error as this is related with networking. If will be fixed once you install weavenet. 
#systemctl status kubelet -l 
Aug 20 17:44:44 worker01 kubelet[12719]: W0820 17:44:44.341224   12719 cni.go:203] Unable to update cni config: No networks found in /etc/cni/net.d
Aug 20 17:44:44 worker01 kubelet[12719]: E0820 17:44:44.341393   12719 kubelet.go:2192] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized








# DOWNLOAD WEAVENET PLUGIN AND INSTALL SYSTEMD-RESOLVED PACKAGE ON WORKER NODES 
- import_playbook: networking.yaml	

-->Execute the playbook. This will only download the plugin on worker nodes. But will not start networking. 
#ansible-playbook -v -i inventory/mycluster -b  playbooks/networking.yaml

-->Verify the output of above command - 
####To check if weavenet have been installed or not 
#ansible dc1-k8s-workers-vm -v -i inventory/mycluster -m shell -a "ls -lrth /opt/cni/bin"
OUTPUT - contents of /opt/cni/bin 
worker01 | CHANGED | rc=0 >>
total 48M
-rwxr-xr-x. 1 root root 2.8M Mar 15  2019 flannel
-rwxr-xr-x. 1 root root 3.4M Mar 15  2019 portmap
-rwxr-xr-x. 1 root root 2.8M Mar 15  2019 tuning
-rwxr-xr-x. 1 root root 3.9M Mar 15  2019 bridge
-rwxr-xr-x. 1 root root 3.0M Mar 15  2019 host-device
-rwxr-xr-x. 1 root root 3.5M Mar 15  2019 ipvlan
-rwxr-xr-x. 1 root root 3.0M Mar 15  2019 loopback
-rwxr-xr-x. 1 root root 3.5M Mar 15  2019 macvlan
-rwxr-xr-x. 1 root root 3.9M Mar 15  2019 ptp
-rwxr-xr-x. 1 root root 3.5M Mar 15  2019 vlan
-rwxr-xr-x. 1 root root 9.8M Mar 15  2019 dhcp
-rwxr-xr-x. 1 root root 2.9M Mar 15  2019 host-local
-rwxr-xr-x. 1 root root 2.6M Mar 15  2019 sample

####To check if systemd-resolved have been installed 
#ansible dc1-k8s-workers-vm -v -i inventory/mycluster -m shell -a "systemctl status systemd-resolved"
OUTPUT 
Aug 20 17:50:05 worker02 systemd[1]: Starting Network Name Resolution...
Aug 20 17:50:05 worker02 systemd-resolved[14539]: Using system hostname 'worker02'.
Aug 20 17:50:05 worker02 systemd-resolved[14539]: Switching to system DNS server 192.168.1.1.
Aug 20 17:50:05 worker02 systemd[1]: Started Network Name Resolution.







# ENABLE PORT  10250 ON WORKER NODES  
- import_playbook: dns-networking-worker.yaml

-->Execute playbook and allow ports on worker nodes 
#ansible-playbook -v -i inventory/mycluster -b  playbooks/dns-networking-worker.yaml






# TO START NETWORKING ON WORKER NODES. 
- import_playbook: dns-networking.yaml 	

####Edit the entry in dns-networking playbook file -  playbooks/dns-networking.yaml - only enter 1 master name 
  1       hosts: master01		

-->Execute the playbook for weave networking. This will run only on 1 master node. 
#ansible-playbook -v -i inventory/mycluster  playbooks/dns-networking.yaml --tags weave

-->Verify from the master node. Both weave pods should be up and running. It will take some time to pull the images. 
#kubectl get pods -n kube-system
OUTPUT 
NAME              READY   STATUS    RESTARTS   AGE
weave-net-cxs8g   1/2     Running   0          46s
weave-net-sjcgc   2/2     Running   0          46s

#kubectl describe pods weave-net-cxs8g -n kube-system
...
...
...check output of weavenet pods 
...
...

-->Verify from the master node. Check if NODES are in READY state or not. The nodes should be in READY state. 
#kubectl get nodes --kubeconfig admin.kubeconfig
OUTPUT 
NAME       STATUS   ROLES    AGE   VERSION
worker01   Ready    <none>   18m   v1.13.0
worker02   Ready    <none>   18m   v1.13.0







# EXECUTE THIS COMMAND TO START DNS FROM EACH WORKER NODE 

-->Execute the playbook for coredns. This will run only on 1 master node.
#ansible-playbook -v -i inventory/mycluster  playbooks/dns-networking.yaml --tags dns 


-->To verify if coredns pods are running or not. Run this command from master node. You may get error. Please proceed further.  

#kubectl get pods -n kube-system -o wide
NAME                       READY   STATUS             RESTARTS   AGE     IP              NODE       NOMINATED NODE   READINESS GATES
coredns-69cbb76ff8-pqvkx   1/1     Running            2          57s     10.44.0.1       worker01   <none>           <none>
coredns-69cbb76ff8-thpwf   0/1     CrashLoopBackOff   1          57s     10.32.0.2       worker02   <none>           <none>



	
	
TO ALLOW RBAC PERMISSION TO ALLOW K8S API SERVER TO ACCESS KUBELET API ON EACH WORKER NODE 	
- import_playbook: rbac-permissoin.yaml 	

####Edit the file and only enter 1 master -  playbooks/rbac-permissoin.yaml - 
	1 		hosts: master01

-->To execute playbook - 
#ansible-playbook -v -i inventory/mycluster  playbooks/rbac-permissoin.yaml

--->From master nodes run below command to check if RBAC is active or not. 
#kubectl get clusterrole system:kube-apiserver-to-kubelet
OUTPUT 
NAME                               AGE
system:kube-apiserver-to-kubelet   58s

#kubectl get clusterrolebinding system:kube-apiserver
NAME                    AGE
system:kube-apiserver   118s






# TO FIX COREDNS ISSUE. 

-->Check coredns pods logs from any of the WORKER NODE. 
#docker ps -a 
#docker logs -f <dnscore-container-id>
.:53
2020/08/20 14:12:14 [INFO] CoreDNS-1.2.2
2020/08/20 14:12:14 [INFO] linux/amd64, go1.11, eb51e8b
CoreDNS-1.2.2
linux/amd64, go1.11, eb51e8b
2020/08/20 14:12:14 [INFO] plugin/reload: Running configuration MD5 = 2e2180a5eeb3ebf92a5100ab081a6381
2020/08/20 14:12:20 [FATAL] plugin/loop: Seen "HINFO IN 5069710642687157789.7623809984875323427." more than twice, loop detected


-->Change /etc/resolv.conf file from each WORKER NODE. 
#cat /etc/resolv.conf
#Generated by NetworkManager
search in.idemia.com
#nameserver 192.168.1.1	#comment out this line and enable 8.8.8.8 
nameserver 8.8.8.8
[root@worker01 ~]#

#systemctl restart systemd-resolved
#systemctl status systemd-resolved -l 
OUTPUT -
Aug 20 18:16:04 worker02 systemd-resolved[22059]: Using system hostname 'worker02'.
Aug 20 18:16:04 worker02 systemd-resolved[22059]: Switching to system DNS server 8.8.8.8.
Aug 20 18:16:04 worker02 systemd[1]: Started Network Name Resolution.

-->From any master node, delete coredns pods. Get POD  NAME and then DELETE IT. 
#kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS             RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
coredns-69cbb76ff8-pqvkx   0/1     CrashLoopBackOff   6          12m   10.44.0.1       worker01   <none>           <none>
coredns-69cbb76ff8-thpwf   0/1     CrashLoopBackOff   6          12m   10.32.0.2       worker02   <none>           <none>
weave-net-cxs8g            2/2     Running            0          18m   192.168.1.172   worker01   <none>           <none>
weave-net-sjcgc            2/2     Running            0          18m   192.168.1.87    worker02   <none>           <none>
#
#kubectl delete pods coredns-69cbb76ff8-pqvkx coredns-69cbb76ff8-thpwf -n kube-system
pod "coredns-69cbb76ff8-pqvkx" deleted
pod "coredns-69cbb76ff8-thpwf" deleted

-->New coredns pods will be created automatically 
#kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
coredns-69cbb76ff8-fgmwq   1/1     Running   0          43s   10.44.0.1       worker01   <none>           <none>
coredns-69cbb76ff8-fhq5m   1/1     Running   0          42s   10.32.0.2       worker02   <none>           <none>



-->Run this command 
#kubectl run --generator=run-pod/v1  busybox --image=busybox:1.28 --command -- sleep 3600
pod/busybox created
#kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
busybox   1/1     Running 			   0          8s
#

-->check if you have proper clusterDNS name on worker nodes 
#vi /var/lib/kubelet/kubelet-config.yaml
clusterDNS:
- "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"


Verify the permission & DNS resolution from master node 
#kubectl run --generator=run-pod/v1  busybox --image=busybox:1.28 --command -- sleep 3600
pod/busybox created
#kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
busybox   0/1     ContainerCreating   0          8s

#kubectl exec -ti busybox -- nslookup kubernetes
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local


