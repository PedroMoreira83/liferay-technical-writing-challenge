# Optimizing Ehcache Cluster Replication in Liferay EE

Liferay Enterprise Edition (EE) features a refined ehcache cluster replication mechanism designed to resolve scalability and resource issues found in standard deployments.

## The Problem: Resource Bottlenecks
Standard ehcache uses RMI replication, which relies on a point-to-point communication graph. This structure does not scale for large clusters; as node counts increase, the N-1 event traffic creates significant network congestion.

Additionally, ehcache creates a dedicated replication thread for each cache entity. In a Liferay Portal environment with over 100 cache entities, these threads consume substantial memory. With a default stack size of 2MB, 100+ threads can consume up to 500MB per node. These mostly idle threads also contribute to frequent context-switch overhead.

## The Solution: ClusterLink and Coalescing
Liferay addresses these bottlenecks using **ClusterLink**, an abstract communication channel that utilizes JGroups' UDP multicast to streamline network traffic. 

To optimize CPU and memory usage, we replaced individual entity threads with a specialized pool of **dispatching threads**. These threads manage all cache cluster events in one place, enabling **coalescing**. If a cache object is modified multiple times in rapid succession, the system only notifies remote peers once, significantly reducing total network load.

*EE customers interested in this feature should contact Liferay Support for further details.*