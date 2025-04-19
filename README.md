# kubernetes-schedule
Based on taint, toleration, node affinity, pod affinity, and pod anti-affinity for subtle scenario scheduling

## Scheme Design (2Master+4Wokerer)
### 1.GPU Resource Isolation and Multi-Tenant Scheduling
#### Node Configuration:
##### Worker1~2:
  Add label gpu-type: a100  
  Apply taint gpu-reserved=true:NoSchedule
##### Worker3~4:
  Add label gpu-type: none  
  No taints
##### All nodes:
  Add zone label zone: zone-a (Worker1/3), zone: zone-b (Worker2/4)
#### Scheduling Requirements
  Exclusive use of GPU nodes (Worker1~2).  
  Toleration for GPU taint required by all Pods.  
  Anti-affinity: Pods from the same tenant (labeled with tenant: team-x) must not be scheduled on the same node.  
  Preferred topology spread: Prioritize distributing Pods across multiple availability zones (zone).
### 2.Cache-Dependent Web Service
#### Node Configurations:
Node1: Taint cache=redis:NoSchedule, Label app=redis  
Node2: Taint cache=memcached:NoSchedule, Label app=memcached  
Node3 & Node4: No taints, Label app=web  
#### Requirements:
Deploy a Deployment with 4 replicas running a web service.  
Each web pod must co-locate with a Redis or Memcached pod on the same node.  
Web pods must have anti-affinity among themselves (avoid multiple replicas on the same node).  
Web pods cannot be scheduled to taint-free nodes (Node3/Node4).
#### Challenge:
Only Node1 and Node2 have cache services, but 4 web pods need to be scheduled. How to resolve this?

### 3.Multi-Region Deployment 
#### Node Configurations:
Node1 & Node2: Labels region=us-east, topology=edge  
Node3 & Node4: Labels region=us-west, topology=core  
All nodes have taint reserved=true:NoSchedule  
#### Requirements:
Deploy a Deployment with 5 replicas for a global monitoring service.  
Pods must tolerate the reserved taint.  
Pods must run on nodes with topology=core .  
No more than 2 Pods per region .  
At least 1 Pod should run on a node with topology=edge.
#### Challenge:
Only Node3 and Node4 (in us-west) are labeled topology=core, but the deployment must distribute replicas across regions while adhering to the per-region limit. How to resolve this?


 
