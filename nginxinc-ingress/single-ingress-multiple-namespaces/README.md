# Single Ingress Multiple Namespaces #

This example covers an Nginx inc ingress controller, in its own namespace that
routes to applications in other namespaces.

## Things to consider ##

### Crossing namespaces requires known Cluster IP for service ###

Due to the way that the ingress controller operates, it's not possible to
create ExternalName services, as this results in the [default upstream][1]
object which won't work as expected.

This means that we have to create our app services first, then reference the
cluster IP in an headless service with an endpoint.

### Paths may require rewrites ###

When using the Ingress paths, the applications may be deployed not expecting to
handle the path. In this case, the [nginx.org/rewrites][2] annotation is used.

## Objects ##

These are the objects defined.

* [Namespace](./manifests/Namespace.yaml)

  Defines the namespace the rest of the objects will be in.

* [ConfigMap](./manifests/ConfigMap.yaml)

  For testing, disabled SSL redirects.

* [Secret](./manifests/Secret.yaml)

  Define two TLS crt/key secrets for TLS/SSL support.

* [ClusterRole](./manifests/ClusterRole.yaml)

  Taken from the [rbac deployment files][3].

* [ServiceAccount](./manifests/ServiceAccount.yaml)

  Service account ingress controller operates with.

* [ClusterRoleBinding](./manifests/ClusterRoleBinding.yaml)

  Applies the rbac taken from [the deployment files][3].

* [Deployment](./manifests/Deployment.yaml)

  Based on the [nginxinc deployment manifest][4].

* [Ingress](./manifests/Ingress.yaml)

  Defines the hosts, paths and designates the two tls hosts and secrets.

* [Service](./manifests/Service.yaml)

  Specifies the ingress service and internamespaces services.

## Services and Endpoints ##

The Services will fail as is, since the ip is mandatory. In order to complete
the manifest, you have to have the [test applications][5] running with
services. You would plug in the cluster IP into the Endpoints definitions::

    [jcastillo2nd@host single-ingress-multiple-namespaces]$ kubectl get services --all-namespaces
    NAMESPACE                NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
    default                  kubernetes             ClusterIP      10.245.0.1       <none>           443/TCP                      3h14m
    kube-system              kube-dns               ClusterIP      10.245.0.10      <none>           53/UDP,53/TCP                3h14m
    nginxingress-namespace   nginxingress-service   LoadBalancer   10.245.197.37    1.2.3.4          80:30988/TCP,443:32094/TCP   157m
    nginxingress-namespace   test-app-api-service   ClusterIP      10.245.106.16    <none>           80/TCP                       33m
    nginxingress-namespace   test-app-one-service   ClusterIP      10.245.130.21    <none>           80/TCP                       157m
    nginxingress-namespace   test-app-two-service   ClusterIP      10.245.220.22    <none>           80/TCP                       157m
    testappapi-namespace     testappapi-service     ClusterIP      10.245.180.64    <none>           80/TCP                       36m
    testappone-namespace     testappone-service     ClusterIP      10.245.213.91    <none>           80/TCP                       163m
    testapptwo-namespace     testapptwo-service     ClusterIP      10.245.231.135   <none>           80/TCP                       160m

The CLUSTER-IP value for the testappapi-service, testappone-service and
testapptwo-service services are used in the Endpoints

    apiVersion: v1
    kind: Service
    metadata:
      name: test-app-api-service
      namespace: nginxingress-namespace
    spec:
      ports:
      - name: http
        port: 80
        targetPort: 80
    ---
    apiVersion: v1
    kind: Endpoints
    metadata:
      name: test-app-api-service
      namespace: nginxingress-namespace
    subsets:
    - addresses:
      - ip: 10.245.180.64
      ports:
      - port: 80

Once all the manifests are applied, the nginxingress-service EXTERNAL-IP can be
updated in public DNS or the hosts file and tested for http:

    [jcastillo2nd@thor kubernetes-examples]$ curl http://test.do-testing.com/app1
    <html>
    <head><title>Test App One</title></head>
    <body><h1>Test App One!</h1><p>This is test app one</p></body></html>
    [jcastillo2nd@thor kubernetes-examples]$ curl http://test.do-testing.com/app2
    <html>
    <head><title>Test App Two</title></head>
    <body><h1>Test App Two!</h1><p>This is test app two</p></body></html>
    [jcastillo2nd@thor kubernetes-examples]$ curl http://api.test.do-testing.com/
    <html>
    <head><title>API Service</title></head>
    <body><h1>Test API Service!</h1><p>This is test app for an API</p></body></html>

And for TLS ( using -k to skip certificate check ):

    [jcastillo2nd@thor kubernetes-examples]$ curl -k https://test.do-testing.com/app1
    <html>
    <head><title>Test App One</title></head>
    <body><h1>Test App One!</h1><p>This is test app one</p></body></html>
    [jcastillo2nd@thor kubernetes-examples]$ curl -k https://test.do-testing.com/app2
    <html>
    <head><title>Test App Two</title></head>
    <body><h1>Test App Two!</h1><p>This is test app two</p></body></html>
    [jcastillo2nd@thor kubernetes-examples]$ curl -k https://api.test.do-testing.com/
    <html>
    <head><title>API Service</title></head>
    <body><h1>Test API Service!</h1><p>This is test app for an API</p></body></html>


[1]: https://github.com/nginxinc/kubernetes-ingress/blob/release-1.4/internal/nginx/nginx.go#L182
[2]: https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/configmap-and-annotations.md#summary-of-configmap-and-annotations
[3]: https://github.com/nginxinc/kubernetes-ingress/blob/master/deployments/rbac/rbac.yaml
[4]: https://github.com/nginxinc/kubernetes-ingress/blob/master/deployments/deployment/nginx-ingress.yaml
[5]: ../test-apps/
