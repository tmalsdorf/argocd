# Sonarr Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Sonarr](https://sonarr.tv/). Sonarr is a PVR for Usenet and BitTorrent users that can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Sonarr application.
- **`Service.yaml`**: Creates a Kubernetes Service to expose Sonarr internally. (Note potential capitalization).
- **`ingress.yml`**: Defines an Ingress resource to expose Sonarr externally. Customize `host` and TLS.
- **`PersistentVolumeClaim.yaml`**: Defines a PVC for Sonarr's configuration data. (Note potential capitalization).

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured.
- Ingress controller if using `ingress.yml`.
- StorageClass for `PersistentVolumeClaim.yaml`.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` and TLS settings.
    - **`PersistentVolumeClaim.yaml`**: Check `storageClassName` and storage size.
    - **`deployment.yml`**: Verify image version, volume mounts (especially for `/config`, `/tv` (your series library), and any download directories you might map), and environment variables (e.g., `TZ`, `PUID`, `PGID`).

2.  **Apply Manifests:**
    Deploy to your namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/sonarr/PersistentVolumeClaim.yaml
    kubectl apply -n <your-namespace> -f k3s/sonarr/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/sonarr/Service.yaml
    kubectl apply -n <your-namespace> -f k3s/sonarr/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>`.)*

    For ArgoCD, point it to `k3s/sonarr/`.

## Accessing Sonarr

-   **Via Ingress:** Access Sonarr at the host in `ingress.yml` (e.g., `http://sonarr.yourdomain.com`).
-   **Via Service (Internal):** Use port-forwarding. Sonarr's default port is 8989.
    ```bash
    kubectl port-forward -n <your-namespace> svc/sonarr 8989:8989
    ```
    Access at `http://localhost:8989`. Check `metadata.name` in `Service.yaml` for the correct service name (usually `sonarr`).

## Important Considerations

-   **Paths:** Sonarr needs access to your TV series library and download client's completed download directory. Ensure your volume mounts in `deployment.yml` correctly map these paths from your host/NFS to the paths Sonarr expects inside the container.
-   **Permissions:** The user ID (`PUID`) and group ID (`PGID`) used by Sonarr (often set via environment variables in `deployment.yml`) must have read/write access to the configuration volume and your TV/download directories.

## Further Information

-   **Sonarr Website:** [https://sonarr.tv/](https://sonarr.tv/)
-   **Sonarr GitHub:** [https://github.com/Sonarr/Sonarr](https://github.com/Sonarr/Sonarr)
