1. Create namespaces

- `baseline-without-kyverno`: kubectl apply -f baseline-without-kyverno.yml
  Enforces baseline PSS level without any Kyverno policy

- `privileged-with-kyverno`: kubectl apply -f baseline-without-kyverno.yml
  Enforces privileged PSS level with a Kyverno policy

2. Try to run a Pod with privileged containers in baseline-without-kyverno namespace

kubectl apply -f pod-privileged-container.yml -n baseline-without-kyverno

Expected result: Pod creation is forbidden

```
Error from server (Forbidden): error when creating "pod-privilged-container.yml": pods "nginx" is forbidden: violates PodSecurity "baseline:v1.23": privileged (containers "init-nodejs", "nginx" must not set securityContext.privileged=true)
```

3. Create a Kyverno policy with the new `validate.podSecurity` rule.

   kubectl apply -f enforce-baseline-exclude-all-privileged-containers.yml
   Exclude all restricted fields of `Privileged containers` control for selected containers

4. Try to run a Pod with privileged containers in baseline-without-kyverno namespace
   kubectl apply -f pod-privileged-container.yml -n privileged-with-kyverno

Expected result: Pod is created

kubectl delete -f pod-privileged-container.yml -n privileged-with-kyverno

5. Delete the Kyverno policy
   kubectl delete clusterpolicy enforce-baseline-exclude-all-privileged-containers

6. Create another Kyverno policy with the new `validate.podSecurity` rule.
   kubectl apply -f enforce-baseline-exclude-privileged-containers.yml
   Exclude the specified restricted fields of `Privileged containers` control for selected containers.

7. Try to run a Pod with privileged containers in baseline-without-kyverno namespace
   kubectl apply -f pod-privileged-container.yml -n privileged-with-kyverno

   Expected result: Pod is created

8. Try to run a Pod with privileged containers in baseline-without-kyverno namespace with non matching container
   kubectl apply -f pod-privileged-not-matching-container.yml -n privileged-with-kyverno

   Expected result: Pod is forbidden

9. Autogen: try to run a deployment
   kubectl apply -f deployment.yml -n privileged-with-kyverno

   Expected result: Pod is created
