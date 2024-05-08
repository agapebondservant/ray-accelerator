# KubeRay Accelerator

This is an accelerator that can be used to generate a Kubernetes deployment for [Ray](https://www.ray.io/).

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
tanzu acc create ray --git-repository https://github.com/agapebondservant/ray-accelerator.git --git-branch main
```

## Contents
1. [Install Ray with vanilla Kubernetes](#k8s)
1. [Integrate with TAP](#tap)

### Install Ray with vanilla Kubernetes<a name="k8s"/>

Fetch the KubeRay Helm repository:
```
helm repo add kuberay https://ray-project.github.io/kuberay-helm/
helm repo update
```
Deploy the KubeRay operator (NOTE: Wait until the pod(s) show "Ready" status - could take up to a minute):
```
helm install kuberay-operator kuberay/kuberay-operator -nray --create-namespace --version 1.0.0-rc.0
watch kubectl get pods -nray # Pod should show "Ready" status
```

Deploy a Ray cluster (Update properties as desired - see <a href="https://github.com/ray-project/kuberay-helm/blob/main/helm-chart/ray-cluster/values.yaml" target="_blank">Kuberay Helm Documentation</a>):
```
helm install raycluster kuberay/ray-cluster -nray --version 1.0.0-rc.0 \
    --set image.tag=2.6.3-py310 \
    --set head.resources.limits.memory=8G \
    --set head.resources.requests.memory=8G \
    --set worker.resources.limits.memory=8G \
    --set worker.resources.requests.memory=8G \
    --set service.type=LoadBalancer
watch kubectl get pods -nray --selector=ray.io/cluster=raycluster-kuberay # Wait till the pods are in "Running" state
```

Set up Ingress:
```
ytt -f resources/httpproxy.yaml -f resources/values.yaml | kubectl apply -nray -f -
kubectl annotate service raycluster-kuberay-head-svc -nray service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout=3600 # for AWS
```

Get the Ray Client Server endpoint:
```
export RAY_ADDRESS=ray://$(kubectl get svc raycluster-kuberay-head-svc -o jsonpath="{.status.loadBalancer.ingress[0]['hostname', 'ip']}" -nray):10001
```

To uninstall:
```
helm uninstall raycluster -nray
helm uninstall kuberay-operator -nray
kubectl delete crd rayclusters.ray.io
kubectl delete crd rayjobs.ray.io
kubectl delete crd rayservices.ray.io
kubectl delete ns ray
```

## Integrate with TAP<a name="tap"/>

* Deploy the app:
```
tanzu apps workload create rayserver-tap -f resources/tapworkloads/workload.yaml --yes
```

* Tail the logs of the main app:
```
tanzu apps workload tail rayserver-tap --since 64h
```

* Once deployment succeeds, get the URL for the main app:
```
tanzu apps workload get rayserver-tap    #should yield rayserver-tap.default.<your-domain>
```

* To delete the app:
```
tanzu apps workload delete rayserver-tap --yes
```