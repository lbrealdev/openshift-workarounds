
# cachefix openshift4

This workaround solves the problem of degradation for `clusterOperators`.

Export $NEWIMAGE from pod:
```
export NEW_IMAGE=`oc get po -n openshift-network-operator -o=jsonpath='{.items[*].spec.containers[].image}'`
```

Create daemonset cachefix:
```
cat <<EOF > cachefix.yaml
kind: DaemonSet                                                                                                                                                         
apiVersion: apps/v1                                                                                                                                                     
metadata:                                                                                                                                                               
  name: cachefix                                                                                                                                                        
  namespace: openshift-network-operator                                                                                                                                 
  annotations:                                                                                                                                                          
    kubernetes.io/description: |                                                                                                                                        
      This daemonset will flush route cache entries created with mtu of 1450.  See https://bugzilla.redhat.com/show_bug.cgi?id=1825219                                  
    release.openshift.io/version: "{{.ReleaseVersion}}"                                                                                                                 
spec:                                                                                                                                                                   
  selector:                                                                                                                                                             
    matchLabels:                                                                                                                                                        
      app: cachefix                                                                                                                                                     
  template:                                                                                                                                                             
    metadata:                                                                                                                                                           
      labels:                                                                                                                                                           
        app: cachefix                                                                                                                                                   
        component: network                                                                                                                                              
        type: infra                                                                                                                                                     
        openshift.io/component: network                                                                                                                                 
        kubernetes.io/os: "linux"                                                                                                                                       
    spec:                                                                                                                                                               
      hostNetwork: true                                                                                                                                                 
      priorityClassName: "system-cluster-critical"                                                                                                                      
      containers:                                                                                                                                                       
      #                                                                                                                                                                 
      - name: cachefix                                                                                                                                                  
        image: $NEW_IMAGE                                   
        command:                                                                                                                                                        
        - /bin/bash                                                                                                                                                     
        - -c                                                                                                                                                            
        - |                                                                                                                                                             
          set -xe                                                                                                                                                       
          echo "I$(date "+%m%d %H:%M:%S.%N") - cachefix - start cachefix ${K8S_NODE}"                                                                                   
          for ((;;))                                                                                                                                                    
            do                                                                                                                                                          
              if ip route show cache | grep -q 'mtu 14'; then                                                                                                           
                 ip route show cache                                                                                                                                    
                 ip route flush cache                                                                                                                                   
              fi                                                                                                                                                        
              sleep 60                                                                                                                                                  
            done                                                                                                                                                        
        lifecycle:                                                                                                                                                      
          preStop:                                                                                                                                                      
            exec:                                                                                                                                                       
              command: ["/bin/bash", "-c", "echo cachefix done"]                                                                                                        
        securityContext:                                                                                                                                                
          privileged: true                                                                                                                                              
        env:                                                                                                                                                            
        - name: K8S_NODE                                                                                                                                                
          valueFrom:                                                                                                                                                    
            fieldRef:                                                                                                                                                   
              fieldPath: spec.nodeName                                                                                                                                  
      nodeSelector:                                                                                                                                                     
        beta.kubernetes.io/os: "linux"                                                                                                                                  
      tolerations:                                                                                                                                                      
        - operator: "Exists"                                                                                                                                            
          effect: "NoExecute"                                                                                                                                           
        - operator: "Exists"                                                                                                                                            
          effect: "NoSchedule"                                                                                                                                          
EOF
```

Apply DaemonSet:
```
oc apply -f cachefix.yaml
```

Watch the pods:
```
watch 'oc get po'
```
