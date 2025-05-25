# Kubernetes (k3s) Application Configurations

This repository stores Kubernetes manifest files for deploying various applications on a k3s cluster. It is primarily designed to be used with ArgoCD for GitOps-style management.

## Repository Structure

The Kubernetes configurations are organized by application under the `k3s/` directory. Each subdirectory within `k3s/` corresponds to a specific application and contains the necessary YAML manifest files for its deployment, such as:

- `deployment.yml`
- `service.yml`
- `ingress.yml` (if applicable)
- `persistentvolumeclaim.yml` (if applicable)
- `secret.yml` (placeholders or encrypted secrets, ensure you manage sensitive data securely)

Example: `k3s/nginx/` contains the manifests for deploying Nginx.

## Prerequisites

- A running k3s cluster.
- `kubectl` configured to interact with your cluster.
- (Recommended) ArgoCD installed and configured in your cluster for managing these applications.
- Familiarity with Kubernetes concepts (Deployments, Services, Ingress, PVCs, etc.).

## Getting Started

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    ```
2.  **Explore Application Configurations:**
    Navigate to the `k3s/<application_name>/` directory for the application you wish to deploy. Review the manifest files and customize them as needed for your environment (e.g., domain names in Ingress, storage class in PVCs, resource requests/limits).
3.  **Manual Deployment (Optional):**
    You can manually deploy an application using `kubectl`:
    ```bash
    kubectl apply -n <your-namespace> -f k3s/<application_name>/
    ```
    *(Note: Replace `<your-namespace>` and `<application_name>` accordingly. It's generally recommended to deploy each manifest file individually in the correct order if there are dependencies, or manage them via a tool like ArgoCD or Helm.)*

## ArgoCD Integration

This repository is structured optimally for use with [ArgoCD](https://argo-cd.readthedocs.io/), a declarative, GitOps continuous delivery tool for Kubernetes.

### Why ArgoCD?

-   **GitOps:** ArgoCD enables you to define your desired application states in Git (this repository) and automatically syncs these states to your Kubernetes cluster.
-   **Automation:** Changes pushed to this repository can be automatically deployed.
-   **Visibility:** ArgoCD provides a UI to visualize the state of your applications and troubleshoot deployment issues.
-   **Rollbacks:** Easily revert to previous application states.

### Setting up Applications in ArgoCD

You will typically create an ArgoCD `Application` CRD for each application or group of applications you want to manage from this repository.

1.  **Define an ArgoCD Application:**
    Below is an example of how you might define an ArgoCD Application resource to deploy `radarr` from this repository. You would typically apply this YAML using `kubectl apply -f your-argocd-app.yaml` to the namespace where ArgoCD is running.

    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: radarr
      namespace: argocd # The namespace where ArgoCD is running
    spec:
      project: default # Or your specific ArgoCD project
      source:
        repoURL: '<YOUR_GIT_REPOSITORY_URL>' # Replace with the URL of this Git repository
        path: 'k3s/radarr' # Path to the Radarr manifests
        targetRevision: HEAD # Or a specific branch/tag/commit
      destination:
        server: 'https://kubernetes.default.svc' # Target cluster URL
        namespace: 'media' # The namespace where you want Radarr to be deployed
      syncPolicy:
        automated: # Optional: enables auto-sync
          prune: true # Optional: deletes resources not defined in Git
          selfHeal: true # Optional: allows ArgoCD to correct drift from the desired state
        syncOptions:
        - CreateNamespace=true # Optional: creates the destination namespace if it doesn't exist
    ```

2.  **Key fields to customize:**
    *   `metadata.name`: A unique name for your ArgoCD application (e.g., `radarr`, `sonarr`, `nginx-ingress`).
    *   `metadata.namespace`: Should be `argocd` or the namespace where your ArgoCD instance is installed.
    *   `spec.project`: The ArgoCD project to group this application under.
    *   `spec.source.repoURL`: **Crucially, change this** to the actual Git URL of this repository.
    *   `spec.source.path`: The directory within this repository for the specific application (e.g., `k3s/radarr`, `k3s/nginx`).
    *   `spec.source.targetRevision`: The branch, tag, or commit hash to sync from (e.g., `main`, `v1.0.0`, specific commit SHA). `HEAD` tracks the tip of the default branch.
    *   `spec.destination.namespace`: The namespace in your Kubernetes cluster where the application should be deployed (e.g., `media`, `downloads`, `ingress-nginx`).
    *   `spec.syncPolicy`:
        *   `automated`: Configure if you want ArgoCD to automatically sync changes from Git.
        *   `CreateNamespace=true`: Useful if the target namespace might not exist yet.

3.  **ApplicationSets (Advanced):**
    For managing multiple applications from this repository, especially if they follow a similar pattern, consider using ArgoCD's [ApplicationSet controller](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/). This allows you to use generators (e.g., Git directory generator) to automatically create multiple ArgoCD Applications based on the directory structure in this `k3s/` path.

    Example snippet for an ApplicationSet using the Git directory generator:
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: ApplicationSet
    metadata:
      name: k3s-apps
      namespace: argocd
    spec:
      generators:
      - git:
          repoURL: '<YOUR_GIT_REPOSITORY_URL>' # Replace with your Git repo URL
          revision: HEAD
          directories:
          - path: k3s/* # This will find all subdirectories in k3s/
      template:
        metadata:
          name: '{{path.basename}}' # Application name will be the directory name (e.g., radarr)
        spec:
          project: default
          source:
            repoURL: '<YOUR_GIT_REPOSITORY_URL>' # Replace with your Git repo URL
            targetRevision: HEAD
            path: '{{path}}' # Path will be the full path to the directory (e.g., k3s/radarr)
          destination:
            server: 'https://kubernetes.default.svc'
            namespace: '{{path.basename}}' # Deploys each app to a namespace matching its name
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            syncOptions:
            - CreateNamespace=true
    ```
    **Note:** Using `{{path.basename}}` for the destination namespace means each app (radarr, sonarr, etc.) would go into its own namespace. Adjust `namespace` in the template as needed if you want them in a shared namespace or a different naming convention.

### Best Practices

-   **Secrets:** Manage Kubernetes secrets outside of this Git repository, or use a sealed secrets solution. The application manifests can reference these pre-existing secrets. The `secret.yml` files in this repo are often placeholders.
-   **Customization:** While ArgoCD syncs the manifests, you might still need to customize values (e.g., domain names in Ingress, resource limits). You can do this by:
    -   Forking this repository and modifying the YAML files directly.
    -   Using ArgoCD's parameters override feature if manifests are parameterized (though these are not by default).
    -   For more complex scenarios, consider using tools like Kustomize or Helm with ArgoCD, which provide more advanced templating and overlay capabilities. This repository currently provides plain YAML manifests.

Refer to the [official ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/) for more detailed information on setup, usage, and advanced features.

## Applications

This repository includes configurations for the following applications:

- Open-WebUI
- csvtoqxf
- Jackett
- Lidarr
- Nginx (as an Ingress controller or standalone)
- Radarr
- Sonarr
- Transmission

Detailed README files with application-specific notes can be found within each application's directory (e.g., `k3s/Open-WebUI/README.md`).

## Secrets Management

Secrets (like API keys, passwords) should be managed securely. The `secret.yml` files in this repository might be templates or placeholders. Consider using tools like:

- Kubernetes Secrets (ensure they are created securely, possibly outside of Git or encrypted)
- HashiCorp Vault
- Sealed Secrets
- External Secrets Operator

Choose a method that aligns with your security policies.

## Contributing

Contributions to improve or add new application configurations are welcome! Please follow these general guidelines:

1.  Fork the repository.
2.  Create a new branch for your changes.
3.  Ensure your manifest files are well-formatted and follow Kubernetes best practices.
4.  Add or update the application-specific README.md.
5.  Submit a pull request with a clear description of your changes.
