[[_TOC_]]

https://kubernetes.github.io/ingress-nginx/deploy/


### Instructions

Those are the [instructions](https://kubernetes.github.io/ingress-nginx/deploy/) to deploy the Kubernetes ingress.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml
```


### Modifications

After the installation modify the service to not use o loadbalancer and expose it to the external IP of the master VM

```yaml
spec:
   type: ClusterIP
   externalIPs:
   - 146.124.106.215
   ports:
   - name: http
     port: 80
     protocol: TCP
     targetPort: 80
   - name: https
     port: 443
     protocol: TCP
     targetPort: 443
```