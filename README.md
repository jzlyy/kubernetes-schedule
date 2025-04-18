# kubernetes-schedule
Based on taint, toleration, node affinity, pod affinity, and pod anti-affinity for subtle scenario scheduling

## Scheme Design (2Master+4Wokerer)
### 1.GPU Inference Service Multi-Availability Zone High Availability Deployment
#### Node Configuration:
##### 4 Worker nodes distributed across 3 availability zones:
3 GPU nodes (each in a distinct zone: zone=zone1, zone=zone2, zone=zone3), labeled with gpu=true and tainted with gpu:NoSchedule.
1 non-GPU node (no GPU, no gpu=true label or taint).
##### All GPU nodes are labeled with app=gpu-inference
#### Requirements:
##### Deploy a 3-replica inference service (inference-pod) with the following constraints:
  Each replica must be exclusively placed in a different availability zone.  
  Pods must be scheduled to GPU nodes only.
##### Taint Tolerance:
  Add a toleration for the gpu:NoSchedule taint to allow scheduling on GPU nodes.
##### Node Affinity:
  se nodeAffinity with a requiredDuringSchedulingIgnoredDuringExecution rule to enforce scheduling only on nodes labeled with gpu=true.
##### Pod Anti-Affinity:
  Use podAntiAffinity with a requiredDuringSchedulingIgnoredDuringExecution rule to ensure no two replicas are scheduled in the same availability zone (based on the zone label)
##### Explicit Handling for Insufficient GPU Nodes
  If GPU nodes are insufficient (e.g., fewer than 3 available), do not schedule Pods to non-GPU nodes (let them remain pending until GPU nodes meet requirements).

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


 
