# kubernetes-schedule
Based on taint, toleration, node affinity, pod affinity, and pod anti-affinity for subtle scenario scheduling

## Scheme Design
### GPU-Intensive Service Disaster Recovery Deployment
#### Node Configuration:
Node1 & Node2: Taint gpu=true:NoSchedule, label hardware=gpu
Node3 & Node4: Taint disk=ssd:NoSchedule, label hardware=ssd
#### Requirements:
> Deploy a Deployment with 3 replicas running a GPU-intensive service.
> Pods must tolerate the GPU taint and only be scheduled to GPU nodes (Node1/Node2).
> Strict pod anti-affinity: No two pods can run on the same node.
> All pods must be distributed across different availability zones (assume Node1 & Node3 belong to zone=a, Node2 & Node4 to zone=b).
#### Challenge:
Only 2 GPU nodes are available, but 3 replicas must be deployed across zones. How to resolve this?
