helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/

helm upgrade --install descheduler --namespace kube-system descheduler/descheduler --version=0.28.0 -f values.yaml


https://hwchiu.medium.com/exploring-kubernetes-descheduler-0b69903ff109