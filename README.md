# prometheus

```
oc cluster up
oc login -u system:admin
oc adm policy add-cluster-role-to-user cluster-admin developer
oc login -u developer
oc new-project prometheus
```

```
oc process -f https://raw.githubusercontent.com/bendikp/prometheus/master/openshift/objects/template-prometheus.yaml \
  -p NAMESPACE=prometheus \
  -p HAPROXY_PASS=$(oc env dc/router --list -n default | grep STATS_PASSWORD | awk -F= '{print $2}') \
  -p HAPROXY_USER=admin \
  -p IMAGE_PROMETHEUS=prom/prometheus:v2.0.0 \
  | oc create -f -
```


```
sudo iptables -A IN_dockerc_allow -p tcp -m tcp --dport 10250 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A IN_dockerc_allow -p tcp -m tcp --dport 1936 -m conntrack --ctstate NEW -j ACCEPT
```
