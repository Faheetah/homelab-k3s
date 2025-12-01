# Prometheus Setup

Deploy the Prometheus helm chart[^1]. Use the [prometheus.yaml](prometheus.yaml) file that references the k3s helm resource[^2].

```bash
kubectl apply -f prometheus.yaml
```

Create a secret for the Grafana dashboard[^3].

```bash
kubectl -n monitoring create secret generic grafana-admin-secret --from-literal=grafana-user=$(echo -n admin | base64) --from-literal=grafana-password=$(echo -n admin | base64)
```

Port forward Grafana to verify it is responding on http://localhost:3000.

```bash
kubectl -n monitoring port-forward svc/prometeheus-grafana 3000:80
```

# Enable Cluster Metrics

k3s includes a metrics endpoint for monitoring cluster health[^4]. Enable the supervisor metrics by adding it to the k3s configuration and restart the service[^5].

<sub>/etc/rancher/k3s/config.yaml</sub>
```yaml
supervisor-metrics: true
```

Test that the metrics endpoint is available. A lot of metrics outputs should be emitted with `key value` format.

```bash
kubectl get --raw /metrics
```

# References

[^1]: https://github.com/prometheus-community/helm-charts

[^2]: https://docs.k3s.io/add-ons/helm

[^3]: secrets are best managed for production environments by using a template and dotenv.

[^4]: https://docs.k3s.io/reference/metrics

[^5]: https://support.scc.suse.com/s/kb/How-to-enable-and-query-the-supervisor-loadbalancer-metrics-in-an-RKE2-or-K3s-cluster
