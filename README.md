# GitLab Omnibus with Istio Helm Chart

This Helm chart deploys GitLab Omnibus with Istio integration, providing a comprehensive solution for managing your Git repositories, CI/CD pipelines, and other DevOps functionalities within a Kubernetes cluster. It leverages Istio for advanced traffic management, security, and observability.

### Features
* **GitLab Omnibus Deployment**: Deploys a single instance of GitLab Community Edition (CE).
* **Persistent Storage**: Configurable PersistentVolumeClaims for GitLab data (`/var/opt/gitlab`) and configuration (`/etc/gitlab`).
* **Configurable GitLab Settings**: Allows customization of `external_url`, initial root password, SMTP settings, PostgreSQL, and Redis connection details via `values.yaml`.
* **Istio Integration**:
    * **HTTP/HTTPS Gateway**: Exposes GitLab via Istio Gateway for HTTP and HTTPS traffic.
    * **SSH Gateway**: Exposes GitLab SSH via a dedicated Istio TCP Gateway.
    * **Virtual Services**: Configures Istio Virtual Services for routing HTTP/HTTPS and SSH traffic to the GitLab service.
* **Initial Data Copy**: An `initContainer` is used to initialize the GitLab data directory (`/var/opt/gitlab`) and backup `/etc/gitlab` if the respective persistent volumes are empty.
* **Shared Memory Volume**: Configurable `/dev/shm` emptyDir volume for performance optimization.

### Prerequisites
* Kubernetes cluster (version 1.19+ recommended)
* Helm (version 3+)
* Istio installed and configured in your cluster
* A pre-existing Kubernetes Secret for TLS, specified by `gitlab.tls_credential_name` in `values.yaml`, if HTTPS is desired.
* Access to an SMTP server if email notifications are required.
* External PostgreSQL and Redis instances are assumed for production deployments. The chart is configured to connect to external services.

### Installation

1.  **Clone the Repository**:
    ```bash
    git clone https://your-repo-link/gitlab-omnibus-with-Istio.git
    cd gitlab-omnibus-with-Istio
    ```

2.  **Configure `values.yaml`**:
    Edit the `values.yaml` file with your specific configurations, such as:
    * `gitlab.domain`: Your domain for GitLab access.
    * `gitlab.initial_root_password`: Initial password for the root user.
    * `gitlab.tls_credential_name`: Name of your TLS secret.
    * `smtp.user`, `smtp.password`, `smtp.server`: SMTP server details.
    * `postgresql.host`, `postgresql.port`, `postgresql.username`, `postgresql.password`, `postgresql.database`: External PostgreSQL connection details.
    * `redis.host`, `redis.port`: External Redis connection details.

    **Example `values.yaml` snippet:**
    ```yaml
    gitlab:
      domain: "your-domain.com"
      protocol: "https://"
      ssh_sub_domain: "services"
      initial_root_password: "your-secret-root-password"
      tls_credential_name: "your-tls-secret" # Ensure this secret exists in the namespace
    smtp:
      server: "smtp.example.com"
      user: "smtp-user"
      password: "smtp-password"
    postgresql:
      host: "your-postgresql.svc.cluster.local"
      port: "5432"
      username: "gitlab"
      password: "db-password"
      database: "gitlabhq_production"
    redis:
      host: "your-redis.svc.cluster.local"
      port: "6379"
    ```

3.  **Install the Chart**:
    ```bash
    helm install gitlab . --namespace gitlab --create-namespace
    ```
    (Replace `gitlab` with your desired release name and namespace)

### Configuration
The following table outlines the configurable parameters in `values.yaml`:

| Parameter | Description | Default |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| `gitlab.domain` | The base domain for GitLab (e.g., `example.com`). Used for `external_url` and Istio hosts. | `{example.com}` |
| `gitlab.protocol` | The protocol for `external_url`. | `https://` |
| `gitlab.ssh_sub_domain` | Subdomain for SSH access (e.g., `services`). | `services` |
| `gitlab.initial_root_password` | The initial password for the GitLab root user. **Crucial to change after initial deployment.** | `{your-first-root-password}` |
| `gitlab.tls_credential_name` | Name of the Kubernetes Secret containing TLS certificates for HTTPS. | `{your-tls-secret-name}` |
| `gitlab.persistence.enabled` | If `true`, enables persistent storage for `/var/opt/gitlab`. | `true` |
| `gitlab.persistence.volumeName` | Name of the PersistentVolumeClaim for GitLab data. | `data` |
| `gitlab.persistence.accessMode` | Access mode for the data PVC. | `ReadWriteOnce` |
| `gitlab.persistence.size` | Storage size for the data PVC. | `64Gi` |
| `gitlab.persistence.storageClass` | StorageClass for the data PVC. | `local-path` |
| `gitlab.etcPersistence.volumeName` | Name of the PersistentVolumeClaim for `/etc/gitlab`. | `gitlab-etc-claim` |
| `gitlab.etcPersistence.accessMode` | Access mode for the `etc` PVC. | `ReadWriteOnce` |
| `gitlab.etcPersistence.size` | Storage size for the `etc` PVC. | `100Mi` |
| `gitlab.etcPersistence.storageClass` | StorageClass for the `etc` PVC. | `local-path` |
| `gitlab.shmVolume.enabled` | If `true`, enables an `emptyDir` volume for `/dev/shm`. | `true` |
| `gitlab.shmVolume.sizeLimit` | Size limit for the `/dev/shm` volume. | |
| `smtp.server` | SMTP server address. | `smtp.gmail.com` |
| `smtp.user` | SMTP username. | `s{mtp outmail user name}` |
| `smtp.password` | SMTP password. | `{smtp password}` |
| `postgresql.host` | PostgreSQL host. | `internal-postgresql.internal-postgresql.svc.cluster.local` |
| `postgresql.port` | PostgreSQL port. | `5432` |
| `postgresql.username` | PostgreSQL username. | `""` |
| `postgresql.password` | PostgreSQL password. | `""` |
| `postgresql.database` | PostgreSQL database name. | `""` |
| `redis.host` | Redis host. | `redis-master.gitlab.svc.cluster.local` |
| `redis.port` | Redis port. | `6379` |

### Upgrading
To upgrade the chart:
```bash
helm upgrade gitlab . --namespace gitlab
```

### Uninstallation
To uninstall the chart:
```bash
helm uninstall gitlab --namespace gitlab
```
---
Note: PersistentVolumeClaims are retained by default to prevent data loss. You may need to manually delete them if you wish to remove all data.