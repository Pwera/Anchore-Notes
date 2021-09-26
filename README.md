<img src=img/Anchore-K8s.png> 


<table>
<tr>
<td>
Install anchor-engine with<br>helm
<td>

``` bash
helm install anchore -f anchore_values.yaml anchore/anchore-engine
```

<tr>
<td>
Configure <br>anchore-cli
<td>

``` bash
ANCHORE_CLI_USER=admin
ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)
ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/

```

<tr>
<td>
Run anchore-cli
<td>

``` bash
kubectl run -it anchore-cli --restart=Always --image anchore/engine-cli  --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=${ANCHORE_CLI_URL}

```


<tr>
<td>Check current policies
<td>

``` bash
anchore-cli policy list
anchore-cli policy get 2c53a13c-1765-11e8-82ef-23527761d060 --detail

```

<tr>
<td>List analyzed images
<td>

``` bash
anchore-cli image list

```


<tr>
<td>
Scan container image
<td>

``` bash
anchore-cli image add ubuntu
anchore-cli image list
```

<tr>
<td>
Describe Policies
<td>

``` bash
anchore-cli policy describe
anchore-cli policy describe --gate=licenses
anchore-cli policy describe --gate=dockerfile --trigger=exposed_ports

```


<tr>
<td>
Evaluating Images Against Policies
<td>

``` bash
anchore-cli evaluate check docker.io/kaizheh/apache-struts2-cve-2017-5638:latest --detail

```




<tr>
<td>
Evaluating status based on Digest or ID
<td>

``` bash
anchore-cli evaluate check docker.io/library/centos@sha256:191c883e479a7da2362b2d54c0840b2e8981e5ab62e11ab925abf8808d3d5d44 --tag=latest

```



<tr>
<td> Install admission controller
<td>

``` bash

sed 's@{{ANCHORE_CLI_URL}}@'"${ANCHORE_CLI_URL}"'@; s@{{ANCHORE_CLI_USER}}@'"${ANCHORE_CLI_USER}"'@; s@{{ANCHORE_CLI_PASS}}@'"${ANCHORE_CLI_PASS}"'@' image-scanning-admission-controller.yaml > image-scanning-admission-controller.yaml

kubectl apply -f image-scanning-admission-controller.yaml

kubectl get all -n image-scan-k8s-webhook-system

CA_BUNDLE=$(kubectl -n image-scan-k8s-webhook-system get secret image-scan-k8s-webhook-webhook-server-secret -o jsonpath='{.data.ca-cert\.pem}')
```




<tr>
<td>Register webhook
<td>

``` bash
sed 's@{{CA_BUNDLE}}@'"${CA_BUNDLE}"'@' generic-validatingewebhookconfig.yaml > generic-validatingewebhookconfig.yaml

kubectl apply -f generic-validatingewebhookconfig.yaml

kubectl get ValidatingWebhookConfiguration
```



<tr>
<td>Test default policies
<td>

``` bash
kubectl run --image=kaizheh/apache-struts2-cve-2017-5638 --restart=Never apache-struts2 

Error from server (Image failed policy check: kaizheh/apache-struts2-cve-2017-5638 ): admission webhook "validating-create-pods.k8s.io" denied the request: Image failed policy check: kaizheh/apache-struts2-cve-2017-5638

```
<!--<tr>
<td>
<td>
``` bash

``` -->

</table>