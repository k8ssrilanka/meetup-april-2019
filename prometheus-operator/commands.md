helm install --name prometheus-kubernetes --namespace monitoring stable/prometheus-operator -f helm-values.yaml
kubectl -n monitoring create secret generic prometheus-operator-thanos-sidecar-config --from-file=thanos.yaml=prometheus-operator-thanos-sidecar-config.yaml
helm upgrade prometheus-kubernetes stable/prometheus-operator -f helm-values-with-thanos.yaml
kubectl apply -f thanos/