# Deploy Bookinfo sample app for Red Hat OpenShift Service Mesh operator using Helm Chart

- Test the helm chart:
```
$ helm template . --debug
```

- Pre requisites:
    - Service Mesh operator is already installed.

    - istio-system namespace exists and Service Mesh Control Plane (SMCP) has "Ready" as condition status.

      ```
      kind: ServiceMeshControlPlane
      apiVersion: maistra.io/v2
      metadata:
        name: basic
        namespace: istio-system
      spec:
        version: v2.3
        tracing:
          type: Jaeger
          sampling: 10000
        policy:
          type: Istiod
        telemetry:
          type: Istiod
        addons:
          jaeger:
            install:
              storage:
                type: Memory
          prometheus:
            enabled: true
          kiali:
            enabled: true
          grafana:
            enabled: true
      ```

    - Namespace bookinfo exits and it is included in the Service Mesh Member Roll (SMMR).

      ```
      apiVersion: maistra.io/v1
      kind: ServiceMeshMemberRoll
      metadata:
        name: default
        namespace: istio-system
      spec:
        members:
          - bookinfo
      ```
    
-  If you are willing to deploy bookinfo the GitOps way using ArgoCD, create an application: 

    ```
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: bookinfo-app
    spec:
      destination:
        namespace: bookinfo
        server: 'https://kubernetes.default.svc'
      source:
        path: .
        repoURL: 'https://github.com/jtovarro/helm-bookinfo-app.git'
        targetRevision: dev
      sources: []
      project: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```

- ArgoCD managed namespaces are labeled:

  ```
  $ oc label namespace bookinfo argocd.argoproj.io/managed-by=openshift-gitops
    
  $ oc label namespace istio-system argocd.argoproj.io/managed-by=openshift-gitops
  ```

## Author

Juan Carlos Tovar @RedHat
