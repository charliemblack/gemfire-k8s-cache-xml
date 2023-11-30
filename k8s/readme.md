kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml

sleep 60

kubectl create namespace gemfireoperator

kubectl apply -f secret.yml --namespace=gemfireoperator

helm install gemfire-crd oci://registry.tanzu.vmware.com/tanzu-gemfire-for-kubernetes/gemfire-crd --version 2.3.0 --namespace gemfireoperator  --set operatorReleaseName=gemfire-operator 

helm install gemfire-operator oci://registry.tanzu.vmware.com/tanzu-gemfire-for-kubernetes/gemfire-operator --version 2.3.0 --namespace gemfireoperator 

kubectl create namespace gemfiredemo

kubectl apply -f secret.yml --namespace=gemfiredemo

sleep 5

kubectl apply -f cluster.yml --namespace=gemfiredemo