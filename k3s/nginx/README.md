# Nginx Ingress Controller Kubernetes Configuration

This directory contains Kubernetes manifest files for deploying Nginx, likely as an Ingress controller for your k3s cluster. Using Nginx as an Ingress controller allows you to route external traffic to services within your cluster based on rules like hostname and path.

## Manifest Files

- **`namespace.yml`**: Defines a dedicated namespace for the Nginx Ingress controller (e.g., `nginx-ingress`). It's good practice to deploy Ingress controllers in their own namespace.
- **`service-account.yml`**: Creates a ServiceAccount for Nginx Ingress, which the controller will use to interact with the Kubernetes API.
- **`cluster-role.yml`**: Defines a ClusterRole with necessary permissions for the Nginx Ingress controller to watch and manage Ingress, Service, Endpoint, Secret, and ConfigMap resources across the cluster.
- **`cluster-role-binding.yml`**: Binds the ClusterRole to the ServiceAccount created, granting it the defined permissions.
- **`configMap.yml`**: Contains the main Nginx Ingress controller configuration (e.g., `nginx-configuration`). This ConfigMap allows you to customize global Nginx settings.
- **`custom-snippets.configmap.yml`**: (If present) Likely a ConfigMap for adding custom Nginx configuration snippets (e.g., `custom-nginx-snippets`) that can be referenced in Ingress annotations or global settings.
- **`deplyment.yml`**: Defines the Kubernetes Deployment for the Nginx Ingress controller pods themselves. (Note: filename might be a typo and intended as `deployment.yml`).
- **`service.yml`**: Creates a Service of type LoadBalancer or NodePort to expose the Nginx Ingress controller to external traffic. This is the entry point for traffic into your cluster via Ingress.

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured to interact with your cluster.
- Understanding of Kubernetes RBAC (Roles, ClusterRoles, Bindings, ServiceAccounts).
- If your `service.yml` is of type `LoadBalancer`, your cluster environment must have a way to provide an external IP for it (e.g., MetalLB on bare-metal k3s, or cloud provider integration).

## Deployment

1.  **Review Namespace:**
    - Check `namespace.yml` to confirm or change the namespace for the Ingress controller (e.g., `nginx-ingress` or `ingress-nginx`). Ensure all other namespaced resources in this directory are targeted for this namespace.

2.  **Review RBAC:**
    - The `cluster-role.yml` and `cluster-role-binding.yml` are typically standard for Nginx Ingress. Review if you have specific security constraints.

3.  **Review ConfigMaps:**
    - **`configMap.yml`**: Check for any default settings you might want to change (e.g., SSL protocols, HSTS settings, proxy timeouts).
    - **`custom-snippets.configmap.yml`**: If you plan to use custom snippets, review this file.

4.  **Review Deployment (`deplyment.yml`):**
    - Check the Nginx Ingress controller image version.
    - Verify resource requests and limits.
    - Ensure arguments passed to the controller are correct (e.g., `--configmap`, `--ingress-class`).

5.  **Review Service (`service.yml`):**
    - Determine if `LoadBalancer` or `NodePort` is appropriate for your environment.
    - If `LoadBalancer`, ensure your cluster can provision it.
    - Note the ports (usually 80 and 443).

6.  **Apply Manifests:**
    Apply the manifests in a logical order, typically starting with the namespace and RBAC, then configurations, and finally the deployment and service:
    ```bash
    kubectl apply -f k3s/nginx/namespace.yml
    kubectl apply -f k3s/nginx/service-account.yml
    kubectl apply -f k3s/nginx/cluster-role.yml
    kubectl apply -f k3s/nginx/cluster-role-binding.yml
    kubectl apply -f k3s/nginx/configMap.yml
    # kubectl apply -f k3s/nginx/custom-snippets.configmap.yml # If you use it
    kubectl apply -f k3s/nginx/deplyment.yml # Ensure correct filename
    kubectl apply -f k3s/nginx/service.yml
    ```
    *It's crucial to apply these to the cluster in an order that respects dependencies (e.g., ServiceAccount before RoleBinding).*

    For ArgoCD, you can point it to the `k3s/nginx/` path. ArgoCD can usually handle inter-resource dependencies within a sync operation, but applying foundational resources like Namespaces and CRDs first is a common practice if setting up manually or troubleshooting.

## Using the Nginx Ingress Controller

Once deployed, this Nginx Ingress controller will watch for Ingress resources created in any namespace (unless restricted by its configuration). You can then create Ingress resources for your applications, specifying an `ingressClassName` that matches the one this controller is configured to serve (often found in the Deployment args of the Nginx Ingress controller).

Example Ingress resource for an application:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: my-app-namespace
  annotations:
    kubernetes.io/ingress.class: "nginx" # Or your specific Ingress class
spec:
  rules:
  - host: myapp.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

## Further Information

- **Nginx Ingress Controller Documentation (Kubernetes Community):** [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

This setup provides a powerful way to manage external access to your applications. Make sure to consult the official Nginx Ingress documentation for detailed configuration options.
