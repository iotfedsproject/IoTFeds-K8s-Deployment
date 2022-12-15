[[_TOC_]]


### Installation
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install --atomic --create-namespace prometheus prometheus-community/kube-prometheus-stack -f values.yml -n monitoring
```

Default username and password credentials are:
```txt
username: admin  
password: prom-operator
```


Visit [GitHub page](https://github.com/prometheus-operator/kube-prometheus) for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

This is the [GitHub page](https://github.com/prometheus-community/helm-charts) for the helm chart.

