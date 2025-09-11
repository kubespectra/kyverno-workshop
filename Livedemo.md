# Writing the new policy

Check which JMESPath is needed:

```sh {"interactive":"false"}
kubectl get ns team-x -o json | kyverno jp query 'metadata.labels.projectId'
```

# Testing the new policy (Manually and in Github)

```sh {"interactive":"true","terminalRows":"20"}
kyverno test tests/policy-namespace --detailed-results
```

```sh {"interactive":"false"}
kubectl apply -f policies/validate-namespace.yaml
```

If a policy has already been created, dumpPayload can be set to true in the Admission Controller deployment to be able to read the admission.request.

```sh {"interactive":"true","terminalRows":"16"}
kubectl create namespace team-x-backend
kubectl logs kyverno-admission-controller-57b6666858-p25q8 -c kyverno | grep "admission.request" | grep "team-x-backend"
```

```sh {"interactive":"false","terminalRows":"789"}
jq . request.json
```

# Monitoring of the policy

```sh {"interactive":"false"}
kubectl get clusterpolicyreport
```

```sh {"interactive":"false"}
kubectl get policyreport -A
```

```sh
kubectl port-forward svc/policy-reporter-ui 8080:8080 -n policy-reporter
```

```sh

```