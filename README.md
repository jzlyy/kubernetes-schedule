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
### 2.Web Service Dependency and High Availability Deployment
#### Node Configurations:
##### All nodes:
  Add label env: prod
##### Worker1~2:
  Add label service-tier: frontend
##### Worker3~4:
  Add label service-tier: backend
##### Zone labels:
  Worker1/3: zone: zone-a  
  Worker2/4: zone: zone-b
#### Scheduling Requirements
#####Deploy a 2-replica frontend service with:
  Zone co-location: Must be co-located in the same zone as any backend service node.  
  Anti-affinity: Frontend Pods cannot be scheduled on the same node.  
  Node restriction: Scheduling to nodes without service-tier: frontend is prohibited.
##### For backend service:
  Each replica must be distributed across different availability zones (zone).

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


 
