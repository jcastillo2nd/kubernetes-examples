# Single Ingress Multiple Namespaces #

This example covers an Nginx inc ingress controller, in its own namespace that
routes to applications in other namespaces.

This only differs from the [Deployment based][1] example by using a DaemonSet
instead of Deployment. The DaemonSet results in a pod for the Ingress
Controller running on every node in the Cluster.

[1]: ../single-ingress-multiple-namespaces/

