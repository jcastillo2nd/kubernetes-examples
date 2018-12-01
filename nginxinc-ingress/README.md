# Nginx ingress Examples #

An example using the [Nginx Kubernetes Ingress][1] controller.

## single-ingress-multiple-namespaces ##

An [example][2] using Deployments in an isolated Nginx Ingress namespace,
with [test apps][4] running in individual namespaces leveraging Endpoints.

## single-ingress-multiple-namespaces-daemonset ##

Exactly like [single-ingress-multiple-namespaces][2] only [this][3] leverages
DaemonSet for the Ingress controller.

[1]: https://github.com/nginxinc/kubernetes-ingress
[2]: ./single-ingress-multiple-namespaces
[3]: ./single-ingress-multiple-namespaces-daemonset
[4]: ./test-apps
