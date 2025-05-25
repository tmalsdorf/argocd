# Radarr Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Radarr](https://radarr.video/). Radarr is an independent fork of Sonarr reworked for automatically downloading movies via Usenet and BitTorrent.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Radarr application.
- **`Service.yaml`**: Creates a Kubernetes Service to expose Radarr internally. (Note potential capitalization).
- **`ingress.yml`**: Defines an Ingress resource to expose Radarr externally. Customize `host` and TLS.
- **`PersistentVolumeClaim.yaml`**: Defines a PVC for Radarr's configuration data. (Note potential capitalization).

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured.
- Ingress controller if using `ingress.yml`.
- StorageClass for `PersistentVolumeClaim.yaml`.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` and TLS settings.
    - **`PersistentVolumeClaim.yaml`**: Check `storageClassName` and storage size.
    - **`deployment.yml`**: Verify image version, volume mounts (especially for `/config` and any movie/download directories you might map), and environment variables (e.g., `TZ`, `PUID`, `PGID`).

2.  **Apply Manifests:**
    Deploy to your namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/radarr/PersistentVolumeClaim.yaml
    kubectl apply -n <your-namespace> -f k3s/radarr/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/radarr/Service.yaml
    kubectl apply -n <your-namespace> -f k3s/radarr/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>`.)*

    For ArgoCD, point it to `k3s/radarr/`.

## Accessing Radarr

-   **Via Ingress:** Access Radarr at the host in `ingress.yml` (e.g., `http://radarr.yourdomain.com`).
-   **Via Service (Internal):** Use port-forwarding. Radarr's default port is 7878.
    ```bash
    kubectl port-forward -n <your-namespace> svc/radarr 7878:7878
    ```
    Access at `http://localhost:7878`. Check `metadata.name` in `Service.yaml` for the correct service name (usually `radarr`).

## Important Considerations

-   **Paths:** Radarr needs access to your movie library and download client's completed download directory. Ensure your volume mounts in `deployment.yml` correctly map these paths from your host/NFS to the paths Radarr expects inside the container.
-   **Permissions:** The user ID (`PUID`) and group ID (`PGID`) used by Radarr (often set via environment variables in `deployment.yml`) must have read/write access to the configuration volume and your movie/download directories.

## Further Information

-   **Radarr Website:** [https://radarr.video/](https://radarr.video/)
-   **Radarr GitHub:** [https://github.com/Radarr/Radarr](https://github.com/Radarr/Radarr)
