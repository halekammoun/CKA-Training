# QUESTION 13 hale
Migrate an existing web application from Ingress to Gateway API. You must maintain HTTPS access.
First, create a Gateway named web-gateway with hostname gateway.web.k8s.local that maintains the existing TLS and listener configuration from the existing ingress resource named web.
Next, create an HTTPRoute named web-route with hostname gateway.web.k8s.local that maintains the existing routing rules from the current Ingress resource named web.
Note - A GatewayClass named nginx is installed in the cluster. -->