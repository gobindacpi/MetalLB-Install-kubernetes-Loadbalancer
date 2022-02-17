# MetalLB-Install-kubernetes-Loadbalancer

~~~

root@master:~# kubectl apply -f https://raw.githubusercontent.com/mvallim/kubernetes-under-the-hood/master/services/kube-service-load-balancer.yaml

root@master:~# kubectl get service load-balancer-service -o wide
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
load-balancer-service   LoadBalancer   10.103.112.209   <pending>     80:30017/TCP   30s   app=guestbook,tier=frontend


kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml

root@master:~# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
namespace/metallb-system created
root@master:~# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/controller configured
podsecuritypolicy.policy/speaker configured
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller unchanged
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker unchanged
role.rbac.authorization.k8s.io/config-watcher created
role.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker unchanged
rolebinding.rbac.authorization.k8s.io/config-watcher created
rolebinding.rbac.authorization.k8s.io/pod-lister created
daemonset.apps/speaker created
deployment.apps/controller created

root@master:~# kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
secret/memberlist created

root@master:~# kubectl get deploy -n metallb-system -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                      SELECTOR
controller   1/1     1            1           62s   controller   metallb/controller:v0.9.5   app=metallb,component=controller



Based on the planed network configuration (here) we will have a metallb-config.yaml as below:

Here Network addresses will be openstack noder ip series, that is why you need to configure virtual IP fron network port and must be allow security group,
otherwise need to disable security group. here I set two virtual IP address (10.10.10.222 and 10.10.10.223).

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.10.10.222-10.10.10.223


root@master:~# kubectl create -f metallb-config.yaml
configmap/config created


root@master:~# kubectl get service load-balancer-service -o wide
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE     SELECTOR
load-balancer-service   LoadBalancer   10.103.112.209   10.10.10.222   80:30017/TCP   8m21s   app=guestbook,tier=frontend


Now if you look at the status on the EXTERNAL-IP it is 10.10.10.222 and can be access directly from external, without using NodePort or ClusterIp. 
Remember this IP 10.10.10.222 isn’t assigned to any node. In this example of service we can access using http://10.10.10.222
	  


Cleaning up
**************************

kubectl delete service load-balancer-service

or you can remove namespace for delete all related of MetalLB 


~~~
