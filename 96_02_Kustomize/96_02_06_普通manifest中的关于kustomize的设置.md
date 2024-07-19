
https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/#setting-the-manifests-namespace

# 1 `kustomize build` Options/Parameters


To provide build options to `kustomize build` of default Kustomize version, use `kustomize.buildOptions` field of `argocd-cm` ConfigMap. Use `kustomize.buildOptions.<version>` to register version specific build options.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
    kustomize.buildOptions: --load-restrictor LoadRestrictionsNone
    kustomize.buildOptions.v4.4.0: --output /tmp
```

After modifying `kustomize.buildOptions`, you may need to restart ArgoCD for the changes to take effect.

# 2 Custom Kustomize versions

Argo CD supports using multiple Kustomize versions simultaneously and specifies required version per application. To add additional versions make sure required versions are [bundled](https://argo-cd.readthedocs.io/en/stable/operator-manual/custom_tools/) and then use `kustomize.path.<version>` fields of `argocd-cm` ConfigMap to register bundled additional versions.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
    kustomize.path.v3.5.1: /custom-tools/kustomize_3_5_1
    kustomize.path.v3.5.4: /custom-tools/kustomize_3_5_4
```

Once a new version is configured you can reference it in an Application spec as follows:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
spec:
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: kustomize-guestbook

    kustomize:
      version: v3.5.4
```

Additionally, the application kustomize version can be configured using the Parameters tab of the Application Details page, or using the following CLI command:

```
argocd app set <appName> --kustomize-version v3.5.4
```

# 3 Build Environment

Kustomize apps have access to the [standard build environment](https://argo-cd.readthedocs.io/en/stable/user-guide/build-environment/) which can be used in combination with a [config managment plugin](https://argo-cd.readthedocs.io/en/stable/operator-manual/config-management-plugins/) to alter the rendered manifests.

You can use these build environment variables in your Argo CD Application manifests. You can enable this by setting `.spec.source.kustomize.commonAnnotationsEnvsubst` to `true` in your Application manifest.

For example, the following Application manifest will set the `app-source` annotation to the name of the Application:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook-app
  namespace: argocd
spec:
  project: default
  destination:
    namespace: demo
    server: https://kubernetes.default.svc
  source:
    path: kustomize-guestbook
    repoURL: https://github.com/argoproj/argocd-example-apps
    targetRevision: HEAD
    kustomize:
      commonAnnotationsEnvsubst: true
      commonAnnotations:
        app-source: ${ARGOCD_APP_NAME}
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```




