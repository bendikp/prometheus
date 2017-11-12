# prometheus

```
oc cluster up
oc login -u system:admin
oc adm policy add-cluster-role-to-user cluster-admin developer
oc login -u developer
oc new-project prometheus
```

```
# Remove prometheus annotations from svc/router in default namespace.
oc annotate svc/router -n default prometheus.io/scrape-

# Install Prometheus server
oc process -f \
   https://raw.githubusercontent.com/bendikp/prometheus/master/openshift/objects/template-prometheus.yaml \
  -p NAMESPACE=prometheus \
  -p HAPROXY_PASS=$(oc env dc/router --list -n default | grep STATS_PASSWORD | awk -F= '{print $2}') \
  -p HAPROXY_USER=admin \
  -p IMAGE_PROMETHEUS=prom/prometheus:v2.0.0 \
  | oc create -f -
```


```
# Open in firewall to be able to scrape router and kubelet
sudo iptables -A IN_dockerc_allow -p tcp -m tcp --dport 10250 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A IN_dockerc_allow -p tcp -m tcp --dport 1936 -m conntrack --ctstate NEW -j ACCEPT
```

```
# Create a new project to deploy simple-app in
oc new-project simple-app

oc create -f \
 https://raw.githubusercontent.com/bendikp/prometheus/master/simple_app/objects/list.yml
```


```
# Queries
container_memory_usage_bytes{container_name="simple-app"}
```

```
# Create a new project to deploy Grafana in
oc new-project grafana

oc create -f \
 https://raw.githubusercontent.com/bendikp/prometheus/master/grafana/objects/list.yml

oc adm policy add-scc-to-user anyuid -z grafana -n grafana
```

```
cd grafana

curl -s -H "Authorization: Bearer eyJrIjoiZ3E3dGdRQTk2Slg1eVM0dHl5QjJpZVJMRkk0T0s2dEEiLCJuIjoiQWRtaW4iLCJpZCI6MX0=" \
 -H "Content-Type: application/json" \
 --data @add_datasource.json http://grafana-grafana.127.0.0.1.nip.io/api/datasources \
 | jq .

curl -s -H "Authorization: Bearer eyJrIjoiZ3E3dGdRQTk2Slg1eVM0dHl5QjJpZVJMRkk0T0s2dEEiLCJuIjoiQWRtaW4iLCJpZCI6MX0=" \
 -H "Content-Type: application/json" \
 --data @add_dashboard.json http://grafana-grafana.127.0.0.1.nip.io/api/dashboards/db \
 | jq .
```
