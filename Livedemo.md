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
kubectl create namespace team-z-frontend
kubectl logs kyverno-admission-controller-57b6666858-sw769 -c kyverno -n kyverno | grep "admission.request" | grep "team-z-frontend"
```

```sh {"interactive":"false","terminalRows":"789"}
jq . request.json
```

```sh {"interactive":"false"}
# Policy to check that the tag is not 'latest'
kubectl apply -f policies/validate-image-tag.yaml
```

# Monitoring of the policy

```sh {"interactive":"false"}
# reports for cluster scoped resources
kubectl get clusterpolicyreport
```

```sh {"interactive":"false"}
# reports for namespace scoped resources
kubectl get policyreport -A
```

```sh
kubectl port-forward svc/policy-reporter-ui 8080:8080 -n policy-reporter
```

```sh

```