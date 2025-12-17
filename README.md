# Nginx TLS Proxy Helm Chart

This Helm chart deploys an Nginx instance configured to act as a TLS termination proxy or sidecar. It routes traffic to a backend service (like Nextcloud) while handling SSL/TLS certificates and health checks.

## Architecture

* **Port HTTP**: Provides a `/healthz` endpoint for Kubernetes liveness/readiness probes.
* **Port HTTPS**: Handles HTTPS traffic, terminates TLS, and proxies requests to the backend.
* **Configuration**: Managed via a ConfigMap mounted at `/etc/nginx/conf.d/default.conf`.
* **Certificates**: Managed via a Kubernetes Secret mounted at `/etc/nginx/ssl/`.

---

## Prerequisites

1. **SSL Certificates**: You must place your certificate and key in the `certs/` directory of this chart before deploying:
* `certs/tls.crt`
* `certs/tls.key`


2. **Backend Service**: Ensure the `proxyPass` value in `values.yaml` points to a reachable service in your cluster.

---

## Configuration

The following table lists the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
| --- | --- | --- |
| `service.httpPort` | Port for HTTP/Healthchecks | `80` |
| `service.httpsPort` | Port for HTTPS/TLS | `443` |
| `nginxConfig.clientMaxBodySize` | Max upload size (useful for Nextcloud/File sharing) | `5g` |
| `nginxConfig.proxyPass` | The backend destination URL | `http://nextcloud:8080` |
| `secret.create` | Whether to create the TLS secret from local files | `true` |
| `secret.name` | Name of the secret to create/use | `nginx-ssl-certs` |

---

## Usage

### 1. Prepare Certificates

Add your certs:

```bash
cd  certs
cp /path/to/your/certificate.crt ./certs/tls.crt
cp /path/to/your/private.key ./certs/tls.key

```

### Create self signed certificate

```bash
export MSYS_NO_PATHCONV=1
openssl req -x509 -newkey rsa:2048 -sha256 -days 365 -nodes -keyout ./certs/tls.key -out ./certs/tls.crt -subj "/CN=<your app>.local" -addext "subjectAltName=DNS:<your app>.local"

  ```

### 3. Install

```bash
helm install my-release .

```

### 4. Update

If you update your certificates in the `files/` folder or change the `values.yaml`, run:

```bash
helm upgrade my-release .

```

*Note: The deployment includes a checksum annotation that will automatically trigger a rolling restart of the Nginx pods when the configuration or certificates change.*

---

## Troubleshooting

**Nginx fails to start:**
Check the logs of the pod: `kubectl logs deployment/my-release-nginx`. Common issues include:

* Missing `tls.crt` or `tls.key` in the secret.
* Syntax error in `proxyPass` (ensure it's a valid URL).

**SSL Handshake errors:**
Ensure the client trusts the certificate provided in `files/tls.crt`.

---

Would you like me to add a **Service** manifest to the chart so you can access this Nginx deployment from outside the pod?