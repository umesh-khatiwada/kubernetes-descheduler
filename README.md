helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/

helm upgrade --install descheduler --namespace kube-system descheduler/descheduler --version=0.28.0 -f values.yaml