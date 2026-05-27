## K8s Miscellaneous Commands

1. kind create cluster
2. minikube start
3. kubectl config current-context
4. kubectl config use-context kind-kind
5. kubectl config use-context minikube
6. kubectl auth whoami
7. kubectl auth can-i get pods
8. kubectl auth can-i get pods --as=apache-user -n apache
9. kubectl exec -it apache-test -n apache -- sh
