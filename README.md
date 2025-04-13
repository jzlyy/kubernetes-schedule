# kubernetes-schedule
Based on taint, toleration, node affinity, pod affinity, and pod anti-affinity for subtle scenario scheduling

## Scheme Design (2Master+4Wokerer)
### 1.GPU-Intensive Service Disaster Recovery Deployment
#### Node Configuration:
Node1 & Node2: Taint gpu=true:NoSchedule, label hardware=gpu  
Node3 & Node4: Taint disk=ssd:NoSchedule, label hardware=ssd
#### Requirements:
  Deploy a Deployment with 3 replicas running a GPU-intensive service.   
  Pods must tolerate the GPU taint and only be scheduled to GPU nodes (Node1/Node2).  
  Strict pod anti-affinity: No two pods can run on the same node.  
  All pods must be distributed across different availability zones (assume Node1 & Node3 belong to zone=a, Node2 & Node4 to zone=b).
#### Challenge:
Only 2 GPU nodes are available, but 3 replicas must be deployed across zones. How to resolve this?

### 2.Cache-Dependent Web Service
#### Node Configurations:
Node1: Taint cache=redis:NoSchedule, Label app=redis  
Node2: Taint cache=memcached:NoSchedule, Label app=memcached  
Node3 & Node4: No taints, Label app=web  
#### Requirements:
Deploy a Deployment with 4 replicas running a web service.  
Each web pod must co-locate with a Redis or Memcached pod on the same node (using Pod Affinity).  
Web pods must have anti-affinity among themselves (avoid multiple replicas on the same node).  
Web pods cannot be scheduled to taint-free nodes (Node3/Node4).
#### Challenge:
Only Node1 and Node2 have cache services, but 4 web pods need to be scheduled. How to resolve this?


 
