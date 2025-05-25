# Jackett Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Jackett](https://github.com/Jackett/Jackett). Jackett works as a proxy server: it translates queries from applications (like Sonarr, Radarr, Lidarr) into tracker-site-specific http queries, parses the html or json response, and then sends results back to the requesting software.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Jackett application.
- **`Service.yaml`**: Creates a Kubernetes Service to expose the Jackett application internally within the cluster. (Note the capitalization if the file is indeed `Service.yaml` and not `service.yaml`).
- **`ingress.yml`**: Defines an Ingress resource to expose Jackett externally. You'll likely need to customize the `host` and TLS settings.
- **`PersistentVolumeClaim.yaml`**: Defines a PersistentVolumeClaim (PVC) for storing Jackett's configuration and any downloaded tracker definitions. (Note the capitalization if the file is indeed `PersistentVolumeClaim.yaml`).

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured to interact with your cluster.
- An Ingress controller (like Nginx Ingress) installed if using `ingress.yml`.
- A StorageClass available for the `PersistentVolumeClaim.yaml`.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` to your domain and configure TLS.
    - **`PersistentVolumeClaim.yaml`**: Check `storageClassName` and `spec.resources.requests.storage`.
    - **`deployment.yml`**: Verify the image version, volume mounts, and any environment variables (e.g., for timezone `TZ`).

2.  **Apply Manifests:**
    Deploy to your chosen namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/jackett/PersistentVolumeClaim.yaml
    kubectl apply -n <your-namespace> -f k3s/jackett/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/jackett/Service.yaml
    kubectl apply -n <your-namespace> -f k3s/jackett/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>` with your target namespace.)*

    For ArgoCD, point it to the `k3s/jackett/` path.

## Accessing Jackett

-   **Via Ingress:** Access Jackett at the host defined in `ingress.yml` (e.g., `http://jackett.yourdomain.com`).
-   **Via Service (Internal):** Use port-forwarding:
    ```bash
    kubectl port-forward -n <your-namespace> svc/jackett 9117:9117
    ```
    Then access at `http://localhost:9117`. The service is typically named `jackett` (lowercase) based on standard conventions, even if the YAML filename is `Service.yaml`. Double-check the `metadata.name` in `Service.yaml`.

## Further Information

-   **Jackett GitHub:** [https://github.com/Jackett/Jackett](https://github.com/Jackett/Jackett)

Ensure your PVC has adequate permissions and the user/group context in the deployment matches what Jackett expects for its configuration directory.
