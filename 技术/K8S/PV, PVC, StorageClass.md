
PV、PVC是[K8S]用来做存储管理的资源对象，它们让存储资源的使用变得**可控**，从而保障系统的稳定性、可靠性。
StorageClass则是为了减少人工的工作量而去**自动化创建**_PV的组件。
所有Pod使用存储只有一个原则：**先规划**_ → _**后申请**_ → _**再使用**_


## 概念
### 1.PV (PersistentVolume)
PV是外部存储系统中的一块存储空间，由管理员创建和维护。
与 Volume (存储卷)一样，PV 具有持久性，生命周期独立于 Pod。

### 2.PVC (PersistentVolumeClaim)
(PVC) 是对 PV 的申请 (Claim)。PVC 通常由普通用户创建和维护。
需要为 Pod 分配存储资源时，用户可以创建一个 PVC，指明存储资源的容量大小和访问模式（比如只读）等信息，Kubernetes 会查找并提供满足条件的 PV

### 3.StorageClass
K8S有两种存储资源的供应模式：静态模式和动态模式，资源供应的最终目的就是将适合的PV与PVC绑定：
- 静态模式：管理员预先创建许多各种各样的PV，等待PVC申请使用。
- 动态模式：管理员无须预先创建PV，通过StorageClass自动完成PV的创建以及与PVC的绑定。

StorageClass就是动态模式，根据PVC的需求动态创建合适的PV资源，从而实现存储卷的按需创建。
- 当Pod通过PVC申请存储资源时，直接通过StorageClass去动态的创建对应大小的PV，然后与PVC绑定


## PV与PVC关系
PV相当于对磁盘的分区，PVC相当于APP（应用程序）向某个分区申请多少空间

一旦 PV 与PVC绑定，Pod就可以使用这个 PVC 了。如果在系统中没有满足 PVC 要求的 PV，PVC则一直处于 Pending 状态，直到系统里产生了一个合适的 PV


## PV,PVC使用前后对比
使用前:
![[Pasted image 20250917102148.png|450]]
使用后:
![[Pasted image 20250917102328.png|450]]
