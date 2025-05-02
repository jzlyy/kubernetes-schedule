# kubernetes-schedule
Based on taint, toleration, node affinity, pod affinity, and pod anti-affinity for subtle scenario scheduling

## Scheme Design (2Master+4Wokerer)
### 1.GPU Resource Isolation and Multi-Tenant Scheduling
#### Node Configuration:
##### Worker1~2:
  label:  gpu-type: a100  
  taint:  gpu-reserved=true:NoSchedule
##### Worker3~4:
  label:  gpu-type: none  
  No taints
##### All nodes:
  zone label:  zone: zone-a (Worker1/3), zone: zone-b (Worker2/4)
#### Scheduling Requirements
  Deploy a 3-replica AI inference service with:
  1. Exclusive use of GPU nodes (Worker1~2).  
  2. Toleration for GPU taint required by all Pods.  
  3. Anti-affinity: Pods from the same tenant (labeled with tenant: team-x) must not be scheduled on the same node.  
  4. Preferred topology spread: Prioritize distributing Pods across multiple availability zones (zone).
### 2.Web Service Dependency and High Availability Deployment
#### Node Configurations:
##### All nodes:
  label:  env: prod
##### Worker1~2:
  label:  service-tier: frontend
##### Worker3~4:
  label:  service-tier: backend
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

### 3.Multi-Availability Zone-Sensitive Database Cluster
#### Node Configurations:
  Worker-node1: Labels: zone: zone-a, disk: ssd  
  Worker-node2: Labels: zone: zone-b, disk: hdd
  Worker-node3: Labels: zone: zone-c, disk: ssd
  Worker-node3: Labels: zone: zone-a, disk: nvme
#### Scheduling Requirements
##### Deploy a 3-replica database service with:
  Zone exclusivity: Each Pod must occupy a unique availability zone (3 replicas spanning 3 distinct zone labels).  
  Preferred disk types: Prioritize scheduling Pods on nodes labeled with disk: ssd or disk: nvme.  
  Node anti-affinity: No more than one database Pod may run on the same node.  
  Fallback tolerance: If resources are insufficient, allow partial Pods to use nodes labeled with disk: hdd.

### 4.Tenant-Level Resource Preemption Defense
### Node Configuration
#### Worker1~2:
  taint:  shared-resource=limited:NoSchedule  
  label:  capacity: high (Worker1), capacity: medium (Worker2)
#### Worker3~4:
  No taints  
  label:  capacity: high (Worker3), capacity: medium (Worker4)
### Scheduling Requirements
#### Deploy a 4-replica Tenant-A batch job with:
Taint tolerance: Pods must explicitly tolerate the shared-resource=limited:NoSchedule taint.  
Per-node limit: No more than 2 Tenant-A Pods may run on the same node.  
Node preference: Prioritize scheduling to nodes labeled with capacity: high.
#### For Tenant-B Pods:  
Strict anti-cohabitation: Pods must not coexist with Tenant-A Pods on the same node.  
Node restriction: Scheduling is allowed only on taint-free nodes (Worker3~4).

 
