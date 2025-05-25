# Open-WebUI Kubernetes Configuration

This directory contains the Kubernetes manifest files for deploying [Open-WebUI](https://open-webui.com/). Open-WebUI is a user-friendly web interface for LLMs (Large Language Models).

## Manifest Files

- **`deployment.yml`**: Defines the Kubernetes Deployment for the Open-WebUI application. This manages the application pods, replicas, and update strategy.
- **`service.yml`**: Creates a Kubernetes Service to expose the Open-WebUI application internally within the cluster.
- **`ingress.yml`**: (Optional) Defines an Ingress resource to expose Open-WebUI externally via a domain name. You will likely need to customize the `host` and TLS settings in this file.
- **`persistentvolumeclaim.yml`**: Defines a PersistentVolumeClaim (PVC) for storing Open-WebUI's persistent data. This ensures data like user configurations and chat history are retained across pod restarts.
- **`secret.yml`**: Placeholder for Kubernetes Secrets. You should create your own secrets for any sensitive data Open-WebUI might require (e.g., API keys if integrated with external services). **Do not commit actual secret values to Git.**

## Prerequisites

- A running Kubernetes cluster (k3s recommended).
- `kubectl` configured to interact with your cluster.
- An Ingress controller (like Nginx Ingress) installed in your cluster if you plan to use the `ingress.yml`.
- A StorageClass available that can provision PersistentVolumes for the `persistentvolumeclaim.yml`.

## Deployment

1.  **Review and Customize:**
    - **`ingress.yml`**: Update `spec.rules[0].host` to your desired domain. Configure TLS settings as needed.
    - **`persistentvolumeclaim.yml`**: Ensure the `storageClassName` matches an available StorageClass in your cluster, and adjust `spec.resources.requests.storage` if needed.
    - **`secret.yml`**: Create the actual Kubernetes Secret object in your cluster with the necessary values before deploying. For example:
        ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
          name: open-webui-secrets
          namespace: default # Or your target namespace
        type: Opaque
        stringData:
          OPENAI_API_KEY: "your-api-key-if-needed"
          # Add other sensitive values here
        ```
        Then, ensure the `deployment.yml` correctly references this secret.
2.  **Apply Manifests:**
    Deploy the manifests to your cluster, typically in your desired namespace:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/Open-WebUI/persistentvolumeclaim.yml
    kubectl apply -n <your-namespace> -f k3s/Open-WebUI/secret.yml # Apply your actual secret
    kubectl apply -n <your-namespace> -f k3s/Open-WebUI/deployment.yml
    kubectl apply -n <your-namespace> -f k3s/Open-WebUI/service.yml
    kubectl apply -n <your-namespace> -f k3s/Open-WebUI/ingress.yml # If using Ingress
    ```
    *(Replace `<your-namespace>` with the namespace where you want to deploy Open-WebUI.)*

    Alternatively, if using ArgoCD, point it to the `k3s/Open-WebUI/` path in this repository.

## Accessing Open-WebUI

-   **Via Ingress:** If you configured Ingress, you should be able to access Open-WebUI at the host you specified (e.g., `http://open-webui.yourdomain.com`).
-   **Via Service (Internal):** If not using Ingress, you can access Open-WebUI internally using port-forwarding:
    ```bash
    kubectl port-forward -n <your-namespace> svc/open-webui 8080:80
    ```
    Then access it at `http://localhost:8080`.

## Further Information

-   **Open-WebUI Official Website:** [https://open-webui.com/](https://open-webui.com/)
-   **Open-WebUI GitHub:** [https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui)

Remember to manage your secrets securely and customize the configurations to fit your specific environment.
