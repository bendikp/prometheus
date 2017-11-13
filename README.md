# Prometheus

```
oc cluster up

# Make devloper cluster-admin
oc login -u system:admin
oc adm policy add-cluster-role-to-user cluster-admin developer

# Remove prometheus annotations from svc/router in default namespace
oc annotate svc/router -n default prometheus.io/scrape-
```

## Install Prometheus

```
# Login and create a new project for Prometheus
oc login -u developer
oc new-project prometheus

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

## Deploy simple Python application with Prometheus metrics

```
# Create a new project and deploy simple-app
oc new-project simple-app

oc create -f \
 https://raw.githubusercontent.com/bendikp/prometheus/master/simple_app/objects/list.yml
```

```
# Queries
request_processing_seconds_count{container_name="simple-app"}
```

## Install Grafana

```
# Create a new project and deploy Grafana
oc new-project grafana

oc create -f \
 https://raw.githubusercontent.com/bendikp/prometheus/master/grafana/objects/list.yml

oc adm policy add-scc-to-user anyuid -z grafana -n grafana
```

Create a datasource through the API
```
cd grafana

TOKEN=insert-token-here
curl -s -H "Authorization: Bearer $TOKEN" \
 -H "Content-Type: application/json" \
 --data @add_datasource.json http://grafana-grafana.127.0.0.1.nip.io/api/datasources \
 | jq .
```

[https://grafana.com/dashboards](https://grafana.com/dashboards)
* [Prometheus dashboard](https://grafana.com/dashboards/3662)
* [Kubernetes dashboard](https://grafana.com/dashboards/315)
