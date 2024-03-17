# Exercise 1

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: `default`<br>
* Documentation: [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

</p>
</details>

In this exercise, you will define Role Based Access Control (RBAC) to grant permissions to a specific user. The permissions should only apply to certain API resources and operations.

The following image shows the high-level architecture.

![rbac](imgs/rbac.png)

> [!NOTE]
> If you do not already have a cluster, you can create one by using minikube or you can use the O'Reilly interactive lab ["Setting up RBAC for a user"](https://learning.oreilly.com/scenarios/cka-prep-setting/9781492095477/).

-Create cluster by using minikube:
minikube start --no-vtx-check
-to check status:
minikube status

## Creating the User

Run the script [create-user-context.sh](./create-user-context.sh). It will achieve the following:

1. Create a private key.
2. Create and approve a CertificateSigningRequest.
3. Add a context entry named `johndoe` to the kubeconfig file to represent the user.

For detailed information, see the [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user). Assume that you do not have to memorize these instructions for the exam.


## Checking Default User Permissions

1. Change to the context to `johndoe`.
kubectl config use-context johndoe

2. Create a new Pod. What would you expect to happen?
kubectl create pod my-pod --image=nginx
minikube kubectl create -f pod.yaml
pod.yaml:
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx-container
      image: nginx

Johndoe can't create new pod 
Can't create pod because there is no role 'create' to user johndoe
This might happen if you're trying to access a Kubernetes cluster that uses RBAC (Role-Based Access Control) and authentication mechanisms such as client certificates or username/password.



## Granting Access to the User

1. Switch back to the original context with admin permissions. In minikube, this context is called `minikube`.
kubectl config use-context minikube

2. Create a new Role named `pod-reader`. The Role should grant permissions to get, watch and list Pods.
kubectl apply -f pod-reader-role.yaml
pod-reader-role.yaml:
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

3. Create a new RoleBinding named `read-pods`. Map the user `johndoe` to the Role named `pod-reader`.
kubectl apply -f pod-reader-rolebinding.yaml
pod-reader-rolebinding.yaml:
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: johndoe
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

4. Make sure that both objects have been created properly.
  kubectl describe role pod-reader
  kubectl describe rolebinding read-pods

5. Switch to the context named `johndoe`.
kubectl config use-context johndoe

6. Create a new Pod named `nginx` with the image `nginx`. What would you expect to happen?
kubectl run nginx --image=nginx --restart=Never
Can't create pod because there is no role 'create' to user johndoe, he can just list, get, watch

7. List the Pods in the namespace. What would you expect to happen?
kubectl get pods
Result: No resources found in default namespace.
