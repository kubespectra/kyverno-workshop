---
runme:
  document:
    relativePath: Livedemo.md
  session:
    id: 01K4VPYM4025D5RX0RGH0BT3W8
    updated: 2025-09-11 18:14:29+02:00
---

# Writing the new policy

Check which JMESPath is needed:

```sh {"interactive":"false"}
kubectl get ns team-x -o json | kyverno jp query 'metadata.labels.projectId'

# Ran on 2025-09-11 09:55:03+02:00 for 746ms
Reading from terminal input.
Enter input object and hit Ctrl+D.
# metadata.labels.projectId
"pi****45"

```

# Testing the new policy (Manually and in Github)

```sh {"interactive":"true","terminalRows":"20"}
kyverno test tests/policy-namespace --detailed-results

# Ran on 2025-09-11 10:07:10+02:00 for 1.094s exited with 0
Loading test  ( tests/policy-namespace/kyverno-test.yaml ) ...
  Loading values/variables ...
  Loading policies ...
  Loading resources ...
  Loading exceptions ...
  Applying 1 policy to 5 resources ...
  Checking results ...

│────│──────────────────────────────│─────────────────────────────│───────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│
│ ID │ Y                       │ E                        │ E                  │ T │ N │ E                                                     │
│────│──────────────────────────────│─────────────────────────────│───────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│
│ 1  │ **************************id │ *************************id │ *******ce/****-x          │ **ss   │ Ok     │ validation rule 'validate-ns-label-projectid' passed.       │
│ 2  │ **************************id │ *************************id │ *******ce/****-y          │ **ss   │ Ok     │ Namespace: team-y with projectID: pi****45 ist not allowed. │
│ 3  │ **************************id │ *************************id │ *******ce/****-z          │ **ss   │ Ok     │ The label projectId is missing                              │
│ 4  │ **************************id │ *************************id │ *******ce/*************el │ **ss   │ Ok     │ The label projectId is missing                              │
│ 5  │ **************************id │ *************************id │ *******ce/*****lt         │ **ss   │ Ok     │ preconditions not met                                       │
│────│──────────────────────────────│─────────────────────────────│───────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│


Test Summary: 5 tests passed and 0 tests failed


```

```sh {"interactive":"false"}
kubectl apply -f policies/validate-namespace.yaml

# Ran on 2025-09-11 10:08:33+02:00 for 485ms
clusterpolicy.kyverno.io/validate-namespace-projectid configured

```

If a policy has already been created, dumpPayload can be set to true in the Admission Controller deployment to be able to read the admission.request.

```sh {"interactive":"true","terminalRows":"16"}
kubectl create namespace team-x-backend
kubectl logs ky*************************r-57************q8 -c kyverno | grep "admission.request" | grep "team-x-backend"

# Ran on 2025-09-11 10:09:20+02:00 for 974ms exited with 0
namespace/team-x-backend created
 F ********om/kyverno/kyverno/pkg/webhooks/handlers/du***go:44 > *******on request dump t={"cl********es":["cl*********in","sy**em:ba******er","sy**em:di*****ry","sy**em:pu**************er"],"dr**un":false,"kind":{"group":"","kind":"Na*****ce","ve***on":"v1"},"name":"te**********nd","na*****ce":"te**********nd","ob**ct":{"ap******on":"v1","kind":"Na*****ce","me****ta":{"cr*************mp":"20*********08:09:21Z","la**ls":{"ku*********io/me*********me":"te**********nd"},"ma*********ds":[{"ap******on":"v1","fi******pe":"Fi****V1","fi****V1":{"f:me****ta":{"f:la**ls":{".":{},"f:ku*********io/me*********me":{}}}},"ma***er":"ku**********te","op*****on":"Up**te","time":"20*********08:09:21Z"}],"name":"te**********nd","uid":"02********************************08"},"spec":{"fi******rs":["ku******es"]},"st**us":{"phase":"Ac**ve"}},"ol*****ct":null,"op*****on":"CR**TE","op***ns":{"ap******on":"me*******io/v1","fi********er":"ku**********te","fi***********on":"St**ct","kind":"Cr*********ns"},"re*******nd":{"group":"","kind":"Na*****ce","ve***on":"v1"},"re***********ce":{"group":"","re****ce":"na******es","ve***on":"v1"},"re****ce":{"group":"","re****ce":"na******es","ve***on":"v1"},"roles":null,"uid":"fb********************************65","userInfo":{"groups":["kubeadm:cluster-admins","system:authenticated"],"username":"kubernetes-admin"}} e={"al***ed":true,"uid":"fb********************************65"} s=["cluster-admin","system:basic-user","system:discovery","system:public-info-viewer"] k="/v1, Kind=Namespace" r={"group":"","re****ce":"na******es","ve***on":"v1"} r=webhooks/resource/validate e=************nd e=************nd n=****TE k="/v1, Kind=Namespace" s=[] d=**********************c-400134804465 r={"groups":["kubeadm:cluster-admins","system:authenticated"],"username":"kubernetes-admin"} v=0

```

```sh {"interactive":"false","terminalRows":"789"}
jq . request.json

# Ran on 2025-09-11 10:09:47+02:00 for 138ms
{
  "clusterRoles": [
    "cluster-admin",
    "system:basic-user",
    "system:discovery",
    "system:public-info-viewer"
  ],
  "dryRun": false,
  "kind": {
    "group": "",
    "kind": "Namespace",
    "version": "v1"
  },
  "name": "team-x-backend",
  "namespace": "team-x-backend",
  "object": {
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
      "creationTimestamp": "2025-09-11T08:09:21Z",
      "labels": {
        "kubernetes.io/metadata.name": "team-x-backend"
      },
      "managedFields": [
        {
          "apiVersion": "v1",
          "fieldsType": "FieldsV1",
          "fieldsV1": {
            "f:metadata": {
              "f:labels": {
                ".": {},
                "f:kubernetes.io/metadata.name": {}
              }
            }
          },
          "manager": "kubectl-create",
          "operation": "Update",
          "time": "2025-09-11T08:09:21Z"
        }
      ],
      "name": "team-x-backend",
      "uid": "02072ca2-6881-4786-82e4-fd14a2dda708"
    },
    "spec": {
      "finalizers": [
        "kubernetes"
      ]
    },
    "status": {
      "phase": "Active"
    }
  },
  "oldObject": null,
  "operation": "CREATE",
  "options": {
    "apiVersion": "meta.k8s.io/v1",
    "fieldManager": "kubectl-create",
    "fieldValidation": "Strict",
    "kind": "CreateOptions"
  },
  "requestKind": {
    "group": "",
    "kind": "Namespace",
    "version": "v1"
  },
  "requestResource": {
    "group": "",
    "resource": "namespaces",
    "version": "v1"
  },
  "resource": {
    "group": "",
    "resource": "namespaces",
    "version": "v1"
  },
  "roles": null,
  "uid": "fb38f5f6-d64d-42f7-b67c-400134804465",
  "userInfo": {
    "groups": [
      "kubeadm:cluster-admins",
      "system:authenticated"
    ],
    "username": "kubernetes-admin"
  }
}

```

# Monitoring of the policy

```sh {"interactive":"false"}
kubectl get clusterpolicyreport

# Ran on 2025-09-11 10:12:09+02:00 for 336ms
NAME                                   KIND        NAME                 PASS   FAIL   WARN   ERROR   SKIP   AGE
02********************************08   Namespace   team-x-backend       0      1      0      0       0      2m29s
08********************************86   Namespace   kube-public          0      1      0      0       0      2d20h
09********************************a5   Namespace   kube-system          0      1      0      0       0      2d20h
12********************************9d   Namespace   local-path-storage   0      1      0      0       0      2d20h
13********************************14   Namespace   munich               0      1      0      0       0      32h
45********************************e3   Namespace   psa-test             0      1      0      0       0      2d20h
75********************************15   Namespace   policy-reporter      0      1      0      0       0      2d20h
78********************************43   Namespace   kube-node-lease      0      1      0      0       0      2d20h
7a********************************e5   Namespace   default              0      0      0      0       1      2d20h
a5********************************26   Namespace   hamburg              0      1      0      0       0      34h
b7********************************7a   Namespace   team-z               0      1      0      0       0      2d20h
bd********************************b5   Namespace   kyverno              0      1      0      0       0      2d20h
df********************************df   Namespace   team-x               1      0      0      0       0      2d20h

```

```sh {"interactive":"false"}
kubectl get policyreport -A

# Ran on 2025-09-11 10:12:37+02:00 for 322ms
NAMESPACE            NAME                                   KIND         NAME                                             PASS   FAIL   WARN   ERROR   SKIP   AGE
default              a6********************************5d   Pod          nginx-x                                          1      1      0      0       0      2d17h
kube-system          10********************************26   Deployment   coredns                                          0      0      0      2       0      2d17h
kube-system          1a********************************31   Pod          co********************gm                         0      0      0      0       2      2d17h
kube-system          1e********************************ea   Pod          kube-controller-manager-kind-control-plane       0      0      0      0       2      2d17h
kube-system          53********************************6b   ReplicaSet   co**************8f                               0      0      0      2       0      2d17h
kube-system          71********************************57   Pod          etcd-kind-control-plane                          0      0      0      0       2      2d17h
kube-system          79********************************cd   Pod          ki*********hb                                    0      0      0      0       2      2d17h
kube-system          aa********************************e4   Pod          kube-scheduler-kind-control-plane                0      0      0      0       2      2d17h
kube-system          cb********************************d7   DaemonSet    kube-proxy                                       0      0      0      2       0      2d17h
kube-system          df********************************8e   Pod          kube-apiserver-kind-control-plane                0      0      0      0       2      2d17h
kube-system          e0********************************5c   Pod          kube-proxy-xnxjd                                 0      0      0      0       2      2d17h
kube-system          e2********************************4e   DaemonSet    kindnet                                          0      0      0      2       0      2d17h
kube-system          e8********************************1a   Pod          co********************cw                         0      0      0      0       2      2d17h
kyverno              1f********************************a8   ReplicaSet   ky***************************68******************************* 0      2       0      2d17h
kyverno              2d********************************a3   ReplicaSet   ky***********************r-bf*****c8             0      0      0      2       0      2d17h
kyverno              47********************************f5   Pod          ky**************************r-67************2c   0      0      0      0       2      2d17h
kyverno              53********************************2e   ReplicaSet   ky**************************r-67******57         0      0      0      2       0      2d17h
kyverno              54********************************9f   Pod          ky***********************r-7c************vq      0      0      0      0       2      2d17h
kyverno              6b********************************f7   Pod          ky*************************r-57************q8    0      0      0      0       2      2d17h
kyverno              84********************************9d   Deployment   kyverno-background-controller                    0      0      0      2       0      2d17h
kyverno              a2********************************17   Deployment   kyverno-admission-controller                     0      0      0      2       0      2d17h
kyverno              bd********************************7e   ReplicaSet   ky*************************r-57******58          0      0      0      2       0      2d17h
kyverno              c1********************************3c   Deployment   kyverno-reports-controller                       0      0      0      2       0      2d17h
kyverno              cb********************************ab   ReplicaSet   ky***********************r-7c******df            0      0      0      2       0      2d17h
kyverno              eb********************************0c   Deployment   kyverno-cleanup-controller                       0      0      0      2       0      2d17h
kyverno              ef********************************6e   Pod          ky***********************r-bf***********68       0      0      0      0       2      2d17h
kyverno              f8********************************67   ReplicaSet   ky***********************r-6c******4d            0      0      0      2       0      2d17h
local-path-storage   5e********************************92   Deployment   local-path-provisioner                           0      0      0      2       0      2d17h
local-path-storage   b4********************************d4   Pod          lo*******************r-57************56          2      0      0      0       0      2d17h
local-path-storage   e3********************************41   ReplicaSet   lo*******************r-57******d4                0      0      0      2       0      2d17h
policy-reporter      06********************************44   ReplicaSet   po**********************7f                       0      0      0      2       0      2d17h
policy-reporter      20********************************36   ReplicaSet   po*************************c9                    0      0      0      2       0      2d17h
policy-reporter      28********************************e2   ReplicaSet   po************************56                     0      0      0      2       0      2d17h
policy-reporter      31********************************32   Pod          po**************************7-khg8m              2      0      0      0       0      2d17h
policy-reporter      31********************************84   ReplicaSet   po*************************58                    0      0      0      2       0      2d17h
policy-reporter      3b********************************38   ReplicaSet   po*************************d8                    0      0      0      2       0      2d17h
policy-reporter      3e********************************d9   ReplicaSet   po*************************6f                    0      0      0      2       0      2d17h
policy-reporter      57********************************a0   ReplicaSet   po**********************69                       0      0      0      2       0      2d17h
policy-reporter      90********************************42   Pod          po***************************4f                  2      0      0      0       0      2d17h
policy-reporter      95********************************88   ReplicaSet   po*************************b4                    0      0      0      2       0      2d17h
policy-reporter      a1********************************e9   Deployment   policy-reporter-ui                               0      0      0      2       0      2d17h
policy-reporter      bb********************************ef   ReplicaSet   po*********************bd                        0      0      0      2       0      2d17h
policy-reporter      bc********************************ca   ReplicaSet   po*************************47                    0      0      0      2       0      2d17h
policy-reporter      bd********************************3a   Deployment   policy-reporter                                  0      0      0      2       0      2d17h
psa-test             a3********************************a6   Pod          nginx                                            1      1      0      0       0      2d17h
team-x               85********************************ce   Pod          nginx-x                                          1      1      0      0       0      2d17h

```

```sh
kubectl port-forward svc/policy-reporter-ui 8080:8080 -n policy-reporter

# Ran on 2025-09-11 10:12:49+02:00 for 7h 59m 55.267s exited with 1
Forwarding from 12*****.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
^C
```

```sh

```