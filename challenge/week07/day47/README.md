# Day 47 of #66DaysOfK8s

_Last update: 2021-02-26_

---
Today, I have read about RBAC.

#kubernetes #learning #K8s #66DaysChallenge

---

## Takeaways

* Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization (_Kubernetes official definition_); Check Wikipedia RBAC reference in this [link](https://en.wikipedia.org/wiki/Role-based_access_control).

* It uses the rbac.authorization.k8s.io [API group](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning) to handle authorization.

* The RBAC API defines 4 Kubernetes object types: _Role_, _ClusterRole_, _RoleBinding_ and _ClusterRoleBinding_.

* In Role types (_Role_ and _ClusteRole_) you define which resources and verbs a user (or a set of users) will have access to. In _RoleBinding_ and _ClusterRoleBinding_ you bind users and roles.

* Permissions are purely additive (there are no "deny" rules).

* A Role always sets permissions within a particular namespace.

* ClusterRole, by contrast, is a non-namespaced resource. The resources have different names (Role and ClusterRole) because a Kubernetes object always has to be either namespaced or not namespaced; it can't be both.

---

## Role

A typical ```Role``` manifest is like the following, where in this example a user in the default namespace can read pods:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

The same can be achieved imperatively:

```bash
kubectl create role pod-reader --verb=get --verb=watch --verb=list --resource=pods --dry-run -o yaml
```

---

## ClusterRole

A ```ClusterRole``` can be used to grant the same permissions as a Role. ClusterRoles are cluster-scoped, so it allows to grant access to nodes, pods across all namespaces, or secrets like in the following example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

---

## RoleBinding and ClusterRoleBinding

A role binding grants the permissions defined in a role to a user (or set of users). It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. A RoleBinding grants permissions within a specific namespace whereas a ClusterRoleBinding grants that access cluster-wide.

An example to bind the ```pod-reader``` role to the user ```jane``` is:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

---

## References

* [Kubernetes RBAC 101: authorization](https://www.cncf.io/blog/2020/08/28/kubernetes-rbac-101-authorization/)

* [Using RBAC Authorization (official site)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)


