# prometheus

```
oc cluster up
```

```
iptables -A IN_dockerc_allow -p tcp -m tcp --dport 10250 -m conntrack --ctstate NEW -j ACCEPT
iptables -A IN_dockerc_allow -p tcp -m tcp --dport 1936 -m conntrack --ctstate NEW -j ACCEPT
```

```
oc env dc/router --list -n default | grep STATS_PASSWORD
```
