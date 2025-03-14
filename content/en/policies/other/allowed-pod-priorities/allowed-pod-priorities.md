---
title: "Allowed Pod Priorities"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    A Pod PriorityClass is used to provide a guarantee on the scheduling of a Pod relative to others. In certain cases where not all users in a cluster are trusted, a malicious user could create Pods at the highest possible priorities, causing other Pods to be evicted/not get scheduled. This policy checks the defined `priorityClassName` in a Pod spec to a dictionary of allowable PriorityClasses for the given Namespace stored in a ConfigMap. If the `priorityClassName` is not among them, the Pod is blocked.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/allowed-pod-priorities/allowed-pod-priorities.yaml" target="-blank">/other/allowed-pod-priorities/allowed-pod-priorities.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-podpriorities
  annotations:
    policies.kyverno.io/title: Allowed Pod Priorities
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      A Pod PriorityClass is used to provide a guarantee on the scheduling of a Pod relative to others.
      In certain cases where not all users in a cluster are trusted, a malicious user could create Pods
      at the highest possible priorities, causing other Pods to be evicted/not get scheduled. This policy
      checks the defined `priorityClassName` in a Pod spec to a dictionary of allowable
      PriorityClasses for the given Namespace stored in a ConfigMap. If the `priorityClassName` is not
      among them, the Pod is blocked.
spec:
  validationFailureAction: audit
  background: true
  rules:
  - name: validate-pod-priority
    context:
      - name: podprioritydict
        configMap:
          name: allowed-pod-priorities
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - Deployment
          - DaemonSet
          - StatefulSet
          - Job
    validate:
      message: >-
        The Pod PriorityClass {{ request.object.spec.template.spec.priorityClassName }} is not in the list
        of the following PriorityClasses allowed in this Namespace: {{ podprioritydict.data.{{request.namespace}} }}.
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.template.spec.priorityClassName }}"
            operator: AnyNotIn
            value:  "{{ podprioritydict.data.{{request.namespace}} }}"
  - name: validate-pod-priority-pods
    context:
      - name: podprioritydict
        configMap:
          name: allowed-pod-priorities
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: >-
        The Pod PriorityClass {{ request.object.spec.priorityClassName }} is not in the list
        of the following PriorityClasses allowed in this Namespace: {{ podprioritydict.data.{{request.namespace}} }}.
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.priorityClassName }}"
            operator: AnyNotIn
            value:  "{{ podprioritydict.data.{{request.namespace}} }}"
  - name: validate-pod-priority-cronjob
    context:
      - name: podprioritydict
        configMap:
          name: allowed-pod-priorities
          namespace: default
    match:
      any:
      - resources:
          kinds:
          - CronJob
    validate:
      message: >-
        The Pod PriorityClass {{ request.object.spec.jobTemplate.spec.template.spec.priorityClassName }} is not in the list
        of the following PriorityClasses allowed in this Namespace: {{ podprioritydict.data.{{request.namespace}} }}.
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.jobTemplate.spec.template.spec.priorityClassName }}"
            operator: AnyNotIn
            value:  "{{ podprioritydict.data.{{request.namespace}} }}"

```
