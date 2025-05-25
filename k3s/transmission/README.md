# Transmission Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Transmission](https://transmissionbt.com/). Transmission is a popular, lightweight, and cross-platform BitTorrent client.

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Transmission application.
- **`service.yml`**: Creates a Kubernetes Service to expose Transmission internally within the cluster (for Web UI and peer communication).
- **`ingress.yml`**: Defines an Ingress resource to expose the Transmission Web UI externally. You will likely need to customize the `host` and TLS settings.
- **`PersistentVolumeClaim.yml`**: Defines a PersistentVolumeClaim (PVC) for storing Transmission's configuration, torrent files, and downloaded data.

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured to interact with your cluster.
- An Ingress controller (like Nginx Ingress) installed in your cluster if you plan to use `ingress.yml`.
- A StorageClass available that can provision PersistentVolumes for `PersistentVolumeClaim.yml`.
- Ensure your network/firewall allows BitTorrent traffic if you want to download/upload effectively.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` to your desired domain for the Web UI. Configure TLS settings as needed.
    - **`PersistentVolumeClaim.yml`**:
        - Review `metadata.name` (e.g., `transmission-config-pvc`, `transmission-downloads-pvc`). You might have one PVC for config and another for downloads, or a single one. The provided listing shows one, which would typically be for `/config` and `/downloads` as subpaths or combined. Adjust based on your setup.
        - Ensure the `storageClassName` matches an available StorageClass, and adjust `spec.resources.requests.storage` for config and downloads.
    - **`deployment.yml`**:
        - Verify the container image version (e.g., `lscr.io/linuxserver/transmission`).
        - **Volume Mounts**: Critically review `volumeMounts`. Transmission typically needs:
            - `/config`: For its settings.json and resume files.
            - `/downloads`: Where completed torrents are stored.
            - `/watch`: (Optional) A directory where Transmission can automatically pick up .torrent files.
        - **Environment Variables**:
            - `TZ`: Set your timezone.
            - `PUID` & `PGID`: Set the user and group ID for file permissions on your volumes.
            - `TRANSMISSION_RPC_USERNAME`, `TRANSMISSION_RPC_PASSWORD`: For Web UI authentication. Consider managing these via Kubernetes Secrets rather than plain text in the deployment if possible.
            - Other variables as per the image documentation (e.g., `PEER_PORT`).
    - **`service.yml`**:
        - Note the ports. Typically, Transmission uses:
            - `9091/tcp` for the Web UI (RPC port).
            - A specific port for peer communication (e.g., `51413/tcp` and `51413/udp`). Ensure this peer port is also exposed in the Deployment if it's not the same as a port in the Service. Some images configure this via `PEER_PORT` env var.

2.  **Apply Manifests:**
    Deploy the manifests to your cluster:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/transmission/PersistentVolumeClaim.yml
    # Consider creating Kubernetes secrets for username/password here
    kubectl apply -n <your-namespace> -f k3s/transmission/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/transmission/service.yml
    kubectl apply -n <your-namespace> -f k3s/transmission/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>` with the namespace where you want to deploy Transmission.)*

    For ArgoCD, point it to the `k3s/transmission/` path.

## Accessing Transmission

-   **Web UI (Via Ingress):** If you configured Ingress, access the Transmission Web UI at the host specified (e.g., `http://transmission.yourdomain.com`). You'll be prompted for the username and password.
-   **Web UI (Via Service Port-Forward):**
    ```bash
    kubectl port-forward -n <your-namespace> svc/transmission 9091:9091
    ```
    Then access it at `http://localhost:9091`.
-   **Peer Communication:** The `service.yml` should also expose the peer communication port (e.g., 51413). If this service is of type `LoadBalancer` or `NodePort` and your firewall/router is configured, this allows external peers to connect to your Transmission client.

## Important Considerations

-   **Permissions:** Ensure the `PUID`/`PGID` used in `deployment.yml` have read/write access to the volumes/paths specified in `volumeMounts`.
-   **Security:**
    - Use strong, unique credentials for `TRANSMISSION_RPC_USERNAME` and `TRANSMISSION_RPC_PASSWORD`.
    - Be mindful of the content you download and share.
-   **Port Forwarding (for optimal BitTorrent performance):** For best peer connectivity, you may need to configure port forwarding on your router to forward the `PEER_PORT` (e.g., 51413) to the Kubernetes node(s) running Transmission, especially if the service is `NodePort` or if using `hostPort` (not generally recommended over a Service). If the service is `LoadBalancer`, this is usually handled by the load balancer.

## Further Information

-   **Transmission Official Website:** [https://transmissionbt.com/](https://transmissionbt.com/)
-   **Commonly used Docker Image (LinuxServer.io):** [https://docs.linuxserver.io/images/docker-transmission](https://docs.linuxserver.io/images/docker-transmission)

Customize the configurations to fit your specific environment and storage setup.
