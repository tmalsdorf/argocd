# Lidarr Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Lidarr](https://lidarr.audio/). Lidarr is a music collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new albums from your favorite artists and will grab, sort and rename them.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Lidarr application.
- **`Service.yaml`**: Creates a Kubernetes Service to expose Lidarr internally. (Note potential capitalization).
- **`ingress.yml`**: Defines an Ingress resource to expose Lidarr externally. Customize `host` and TLS.
- **`PersistentVolumeClaim.yaml`**: Defines a PVC for Lidarr's configuration data. (Note potential capitalization).

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured.
- Ingress controller if using `ingress.yml`.
- StorageClass for `PersistentVolumeClaim.yaml`.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` and TLS settings.
    - **`PersistentVolumeClaim.yaml`**: Check `storageClassName` and storage size.
    - **`deployment.yml`**: Verify image version, volume mounts (especially for `/config` and any media download directories you might map), and environment variables (e.g., `TZ`, `PUID`, `PGID`).

2.  **Apply Manifests:**
    Deploy to your namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/lidarr/PersistentVolumeClaim.yaml
    kubectl apply -n <your-namespace> -f k3s/lidarr/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/lidarr/Service.yaml
    kubectl apply -n <your-namespace> -f k3s/lidarr/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>`.)*

    For ArgoCD, point it to `k3s/lidarr/`.

## Accessing Lidarr

-   **Via Ingress:** Access Lidarr at the host in `ingress.yml` (e.g., `http://lidarr.yourdomain.com`).
-   **Via Service (Internal):** Use port-forwarding. Lidarr's default port is 8686.
    ```bash
    kubectl port-forward -n <your-namespace> svc/lidarr 8686:8686
    ```
    Access at `http://localhost:8686`. Check `metadata.name` in `Service.yaml` for the correct service name (usually `lidarr`).

## Important Considerations

-   **Paths:** Lidarr needs access to your music library and download client's completed download directory. Ensure your volume mounts in `deployment.yml` correctly map these paths from your host/NFS to the paths Lidarr expects inside the container.
-   **Permissions:** The user ID (`PUID`) and group ID (`PGID`) used by Lidarr (often set via environment variables in `deployment.yml`) must have read/write access to the configuration volume and your music/download directories.

## Further Information

-   **Lidarr Website:** [https://lidarr.audio/](https://lidarr.audio/)
-   **Lidarr GitHub:** [https://github.com/lidarr/Lidarr](https://github.com/lidarr/Lidarr)
