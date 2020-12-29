# ocp workaround `LeaderElection`

Considering the following scenario, that a cluster OCP 4.x is power off and when you comeback power on all the nodes in `NotReady`.

### Troubleshooting

Get events in all namespaces search for reason `LeaderElection`, this event:
```
oc get ev -A | grep LeaderElection
```

NOTE: This event occurs every 5 minutes

Get numbers of csr in pending state:
```
oc get csr --no-headers | grep Pending | wc -l
```

If there is a CSR in a pending state, it will be necessary to approve it.

Approve csr in pending state:
```
oc get csr -o name | xargs oc adm certificate approve
```

After all certificates are approved, the cluster automatically goes into Ready state.

Run this command to follow the procedure:
```
watch 'oc get no'
```
