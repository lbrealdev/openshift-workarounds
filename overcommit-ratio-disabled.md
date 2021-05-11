# Disable overcommit and maxLimitRequestRatio

Remove maxLimitRequestRatio in all projects:
```
oc get limits -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\n"}{end}' | xargs -n1 oc patch limits limits --type=json -p '[{"op":"remove", "path": "/spec/limits/1/maxLimitRequestRatio/cpu", "value": 40}]' -n
```

Disable overcommit in all projects with labels "$PROJECT_LABEL":
```
oc get ns -l "$PROJECT_LABEL" -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | xargs -n1 oc patch --patch '{"metadata":{"annotations": {"quota.openshift.io/cluster-resource-override-enabled": "false" }}}' ns
```

[overcommit](https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-overcommit.html)
