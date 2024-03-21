# KubeRay Accelerator

This is an accelerator that can be used to generate a Bitnami Kubernetes deployment for [Ray](https://www.ray.io/).

* Install App Accelerator: (see https://docs.vmware.com/en/Tanzu-Application-Platform/1.0/tap/GUID-cert-mgr-contour-fcd-install-cert-mgr.html)
```
tanzu package available list accelerator.apps.tanzu.vmware.com --namespace tap-install
tanzu package install accelerator -p accelerator.apps.tanzu.vmware.com -v 1.0.1 -n tap-install -f resources/app-accelerator-values.yaml
Verify that package is running: tanzu package installed get accelerator -n tap-install
Get the IP address for the App Accelerator API: kubectl get service -n accelerator-system
```

Publish Accelerators:
```
tanzu plugin install --local <path-to-tanzu-cli> all
tanzu acc create bitnami-ray --git-repository https://github.com/agapebondservant/ray-accelerator.git --git-branch bitnami
```

Publish Fragments:
```
tanzu acc fragment create bitnami-ray-fragment --git-repository https://github.com/agapebondservant/ray-accelerator.git --git-branch bitnami
```