# csvtoqxf Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [csvtoqxf](https://github.com/Gnucash/csvtoqxf). This tool is a CSV to QFX converter, often used for importing financial data into GnuCash or other financial software.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the csvtoqxf application. This typically runs a web server that provides the conversion utility.
- **`service.yml`**: Creates a Kubernetes Service to expose the csvtoqxf application internally within the cluster.

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured to interact with your cluster.
- An Ingress controller (like Nginx Ingress) installed in your cluster if you plan to expose csvtoqxf externally (note: no Ingress manifest is provided by default here, you would need to create one).

## Deployment

1.  **Review and Customize:**
    - **`deployment.yml`**: Check the container image version and any environment variables or arguments. The default deployment might run a simple web server; inspect the image's documentation for specific configurations.
    - **`service.yml`**: Note the `port` and `targetPort`.

2.  **Apply Manifests:**
    Deploy the manifests to your cluster, typically in your desired namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/csvtoqxf/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/csvtoqxf/service.yml
    ```
    *(Replace `<your-namespace>` with the namespace where you want to deploy csvtoqxf.)*

    Alternatively, if using ArgoCD, point it to the `k3s/csvtoqxf/` path in this repository.

## Accessing csvtoqxf

-   **Via Service (Internal):** You can access csvtoqxf internally using port-forwarding:
    ```bash
    kubectl port-forward -n <your-namespace> svc/csvtoqxf <local-port>:<service-port>
    ```
    (e.g., `kubectl port-forward -n default svc/csvtoqxf 8080:80` if the service port is 80)
    Then access it at `http://localhost:<local-port>`.

-   **Exposing Externally (Manual Ingress):**
    If you need to access csvtoqxf from outside the cluster, you will need to create an Ingress resource. An example might look like:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: csvtoqxf-ingress
      namespace: <your-namespace> # Match the service namespace
      annotations:
        # Add any necessary ingress annotations for your controller
        # nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - host: csvtoqxf.yourdomain.com # Replace with your desired domain
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: csvtoqxf # Match the service name from service.yml
                port:
                  number: <service-port> # Match the port from service.yml
    ```
    Apply this Ingress manifest using `kubectl apply -f your-ingress-file.yaml`.

## Further Information

-   **csvtoqxf GitHub Repository:** [https://github.com/Gnucash/csvtoqxf](https://github.com/Gnucash/csvtoqxf)
-   **GnuCash (related software):** [https://www.gnucash.org/](https://www.gnucash.org/)

Customize the configurations as needed for your environment.
