---
runme:
  document:
    relativePath: Livedemo.md
  session:
    id: 01KAG6NKNHSV31GPY74AZX4C3V
    updated: 2025-11-20 12:10:35+01:00
---

# Writing the new policy

Check which JMESPath is needed:

```sh {"interactive":"false"}
kubectl get ns team-x -o json | kyverno jp query 'metadata.labels.projectId'

# Ran on 2025-11-20 10:12:40+01:00 for 517ms
Reading from terminal input.
Enter input object and hit Ctrl+D.
# metadata.labels.projectId
"pi****45"

```

# Testing the new policy (Manually and in Github)

```sh {"interactive":"true","terminalRows":"20"}
kyverno test tests/policy-namespace --detailed-results

# Ran on 2025-11-20 10:27:08+01:00 for 1.133s exited with 0
Loading test  ( tests/policy-namespace/kyverno-test.yaml ) ...
  Loading values/variables ...
  Loading policies ...
  Loading resources ...
  Loading exceptions ...
  Applying 1 policy to 5 resources with 0 exceptions ...
  Checking results ...

│────│──────────────────────────────│─────────────────────────────│──────────────────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│
│ ID │ Y                       │ E                        │ E                             │ T │ N │ E                                                     │
│────│──────────────────────────────│─────────────────────────────│──────────────────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│
│ 1  │ **************************id │ *************************id │ v1/Namespace/de***lt/****-x          │ **ss   │ Ok     │ validation rule 'validate-ns-label-projectid' passed.       │
│ 2  │ **************************id │ *************************id │ v1/Namespace/de***lt/****-y          │ **ss   │ Ok     │ Namespace: team-y with projectID: pi****45 ist not allowed. │
│ 3  │ **************************id │ *************************id │ v1/Namespace/de***lt/****-z          │ **ss   │ Ok     │ The label projectId is missing                              │
│ 4  │ **************************id │ *************************id │ v1/Namespace/de***lt/*************el │ **ss   │ Ok     │ The label projectId is missing                              │
│ 5  │ **************************id │ *************************id │ v1/Namespace/de***lt/*****lt         │ **ss   │ Ok     │ preconditions not met                                       │
│────│──────────────────────────────│─────────────────────────────│──────────────────────────────────────│────────│────────│─────────────────────────────────────────────────────────────│


Test Summary: 5 tests passed and 0 tests failed


```

```sh {"interactive":"false"}
kubectl apply -f policies/validate-namespace.yaml

# Ran on 2025-11-20 10:28:13+01:00 for 480ms
clusterpolicy.kyverno.io/validate-namespace-projectid configured

```

If a policy has already been created, dumpPayload can be set to true in the Admission Controller deployment to be able to read the admission.request.

```sh {"interactive":"true","terminalRows":"16"}
kubectl create namespace team-z-frontend
kubectl logs ky*************************r-57************69 -c kyverno -n kyverno | grep "admission.request" | grep "team-z-frontend"

# Ran on 2025-11-20 10:28:50+01:00 for 844ms exited with 0
namespace/team-z-frontend created
 F ********om/kyverno/kyverno/pkg/webhooks/handlers/du***go:44 > *******on request dump t={"cl********es":["cl*********in","sy**em:ba******er","sy**em:di*****ry","sy**em:pu**************er"],"dr**un":false,"kind":{"group":"","kind":"Na*****ce","ve***on":"v1"},"name":"te***********nd","na*****ce":"te***********nd","ob**ct":{"ap******on":"v1","kind":"Na*****ce","me****ta":{"cr*************mp":"20*********09:28:50Z","la**ls":{"ku*********io/me*********me":"te***********nd"},"ma*********ds":[{"ap******on":"v1","fi******pe":"Fi****V1","fi****V1":{"f:me****ta":{"f:la**ls":{".":{},"f:ku*********io/me*********me":{}}}},"ma***er":"ku**********te","op*****on":"Up**te","time":"20*********09:28:50Z"}],"name":"te***********nd","uid":"72********************************03"},"spec":{"fi******rs":["ku******es"]},"st**us":{"phase":"Ac**ve"}},"ol*****ct":null,"op*****on":"CR**TE","op***ns":{"ap******on":"me*******io/v1","fi********er":"ku**********te","fi***********on":"St**ct","kind":"Cr*********ns"},"re*******nd":{"group":"","kind":"Na*****ce","ve***on":"v1"},"re***********ce":{"group":"","re****ce":"na******es","ve***on":"v1"},"re****ce":{"group":"","re****ce":"na******es","ve***on":"v1"},"roles":null,"uid":"05********************************26","userInfo":{"groups":["kubeadm:cluster-admins","system:authenticated"],"username":"kubernetes-admin"}} e={"al***ed":true,"uid":"05********************************26"} s=["cluster-admin","system:basic-user","system:discovery","system:public-info-viewer"] k="/v1, Kind=Namespace" r={"group":"","re****ce":"na******es","ve***on":"v1"} r=webhooks/resource/validate e=*************nd e=*************nd n=****TE k="/v1, Kind=Namespace" s=[] d=**********************1-62********26 r={"groups":["kubeadm:cluster-admins","system:authenticated"],"username":"kubernetes-admin"} v=0

```

```sh {"interactive":"false","terminalRows":"789"}
jq . request.json

# Ran on 2025-11-20 10:29:17+01:00 for 221ms
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
  "name": "team-z-frontend",
  "namespace": "team-z-frontend",
  "object": {
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
      "creationTimestamp": "2025-11-20T09:28:50Z",
      "labels": {
        "kubernetes.io/metadata.name": "team-z-frontend"
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
          "time": "2025-11-20T09:28:50Z"
        }
      ],
      "name": "team-z-frontend",
      "uid": "72db47a5-d829-48b9-9b70-efdf467cf403"
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
  "uid": "05ba3cdb-db20-42af-8c11-62dd7d593f26",
  "userInfo": {
    "groups": [
      "kubeadm:cluster-admins",
      "system:authenticated"
    ],
    "username": "kubernetes-admin"
  }
}

```

```sh {"interactive":"false"}
# Policy to check that the tag is not 'latest'
kubectl apply -f policies/validate-image-tag.yaml

# Ran on 2025-11-20 10:29:56+01:00 for 470ms
clusterpolicy.kyverno.io/validate-image-tag configured

```

# Monitoring of the policy

```sh {"interactive":"false"}
# reports for cluster scoped resources
kubectl get clusterpolicyreport

# Ran on 2025-11-20 10:31:49+01:00 for 385ms
NAME                                   KIND        NAME                 PASS   FAIL   WARN   ERROR   SKIP   AGE
10********************************02   Namespace   team-x-frontend      0      1      0      0       0      22h
34********************************d7   Namespace   local-path-storage   0      1      0      0       0      45h
46********************************43   Namespace   team-y-frontend      0      1      0      0       0      11h
4d********************************40   Namespace   team-z               0      1      0      0       0      45h
69********************************48   Namespace   team-x               1      0      0      0       0      45h
72********************************03   Namespace   team-z-frontend      0      1      0      0       0      2m38s
9a********************************e6   Namespace   kube-public          0      1      0      0       0      45h
a9********************************25   Namespace   policy-reporter      0      1      0      0       0      15h
ae********************************07   Namespace   team-x-backend       0      1      0      0       0      45h
b1********************************98   Namespace   kube-node-lease      0      1      0      0       0      45h
b4********************************1a   Namespace   kyverno              0      1      0      0       0      45h
be********************************10   Namespace   team-z-no-label      0      1      0      0       0      45h
cf********************************86   Namespace   team-y               0      1      0      0       0      45h
e9********************************5e   Namespace   kube-system          0      1      0      0       0      45h
f8********************************5d   Namespace   default              0      0      0      0       1      45h

```

```sh {"interactive":"false"}
# reports for namespace scoped resources
kubectl get policyreport -A

# Ran on 2025-11-20 10:32:09+01:00 for 380ms
NAMESPACE            NAME                                   KIND   NAME                                               PASS   FAIL   WARN   ERROR   SKIP   AGE
kube-system          04********************************f3   Pod    ku************tv                                   0      0      0      0       1      14h
kube-system          0b********************************2a   Pod    ki*********qw                                      0      0      0      0       1      14h
kube-system          1f********************************b1   Pod    etcd-my-cluster-control-plane                      0      0      0      0       1      14h
kube-system          2e********************************bb   Pod    kube-controller-manager-my-cluster-control-plane   0      0      0      0       1      14h
kube-system          36********************************d0   Pod    co********************b4                           0      0      0      0       1      14h
kube-system          72********************************e8   Pod    co********************2d                           0      0      0      0       1      14h
kube-system          9f********************************8e   Pod    kube-scheduler-my-cluster-control-plane            0      0      0      0       1      14h
kube-system          f8********************************c2   Pod    kube-apiserver-my-cluster-control-plane            0      0      0      0       1      14h
kyverno              0b********************************5b   Pod    ky**************************r-67************nd     0      0      0      0       1      14h
kyverno              4a********************************82   Pod    ky*************************r-57************69      0      0      0      0       1      14h
kyverno              d9********************************13   Pod    ky***********************r-bf***********v7         0      0      0      0       1      14h
kyverno              ed********************************97   Pod    ky***********************r-7c************8s        0      0      0      0       1      14h
local-path-storage   e7********************************91   Pod    lo*******************r-57************nv            1      0      0      0       0      14h
policy-reporter      07********************************59   Pod    po****************************7p                   1      0      0      0       0      14h
policy-reporter      29********************************d8   Pod    po**************************6-qr279                1      0      0      0       0      14h
team-x               3c********************************ac   Pod    ng*****************6n                              0      1      0      0       0      14h

```

```sh
kubectl port-forward svc/policy-reporter-ui 8080:8080 -n policy-reporter

# Ran on 2025-11-20 10:33:33+01:00 for 1h 36m 17.082s exited with 1
Forwarding from 12*****.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
^C
```

```sh

```