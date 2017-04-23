
---

#### Kubernetes Cluster Installation

> 详细的介绍了如何一步一步搭建kubernetes cluster [follow-me-install-kubernetes-cluster](https://github.com/opsnull/follow-me-install-kubernetes-cluster)

---

#### Authentication & Authorization & Admission Control

通常，一个kubernetes的请求需要经历三个阶段，才会被kubernetes cluster接受并处理。流程如图所示：

![](images/3-steps-auth.jpg =150*50)

##### Authentication

##### Authorization

##### Admission Control

Admission Control主要工作在于，验证用户的请求是否满足所有的Admission Control plugins。

Admission Control是一组plugins，类似于OpenStack `api-paste.ini`中的filter, 用户请求会
  依次通过plugins，只有通过所有的plugins的验证，请求才会真正被kubernetes cluster处理。

Admission Control plugins通过`kube-apiserver`的参数`--admission-control`进行指定，形如

```
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota
```

更多的plugins介绍，请参照官方文档[Using Admission Controllers](https://kubernetes.io/docs/admin/admission-controllers/)

---