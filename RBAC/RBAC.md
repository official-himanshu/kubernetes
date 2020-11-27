# RBAC for service account and for kubectl 

### Creating a user using openssl certificates

#### Create a key with openssl for a user(himanshu)
	$ openssl genrsa -out himanshu.key 2048

#### Create the csr file as following
	$ openssl req -new -key himanshu.key -out himanshu.csr -subj "/CN=himanshu/O=devops"

#### Create the .crt file for the user
	$ openssl x509 -req -in himanshu.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out himanshu.crt -days 365

#### Now create the kubeconfig file for the user 
	$ kubectl --kubeconfig himanshu.kubeconfig config set-cluster minikube --server https://192.168.49.2:8443 --certificate-authority=ca.crt

#### Now set the credentials for the user himanshu
	$ kubectl --kubeconfig himanshu.kubeconfig config set-credentials himanshu --client-certificate /home/knoldus/kubernetes/RBAC/himanshu.crt --client-key /home/knoldus/kubernetes/RBAC/himanshu.key

#### Now set the context for the user
	$ kubectl --kubeconfig himanshu.kubeconfig config set-context himanshu-minikube --cluster minikube --namespace devops --user himanshu

#### Now create a role
	$ kubectl create role devops --verb=get,list --resource=pods --namespace devops

#### Now bind that role to the user
	$ kubectl create rolebinding devops-rolebinding --role=devops --user=himanshu --namespace devops

#### get the secret token for the dashboard user

	$ kubectl get secret -n kubernetes-dashboard $(kubectl get serviceaccount read-only-user -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

#### Important websites

http://blog.cowger.us/2018/07/03/a-read-only-kubernetes-dashboard.html    --- cluster level role binding with serviceaccount
https://stackoverflow.com/questions/59353819/how-to-bind-roles-with-service-accounts-kubernetes   ---- namespace level role binding with serviceaccount
https://upcloud.com/community/tutorials/deploy-kubernetes-dashboard/  --- cluster level role binding with serviceaccount
