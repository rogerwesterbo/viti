<p align="center">
  <img src="public/viti-logo.svg" alt="Vitistack" height="120" />
</p>

<h1 align="center">viti</h1>

<p align="center">
  A presentation about <a href="https://vitistack.io">Vitistack</a>, an infrastructure platform for running Kubernetes clusters on multiple machine and Kubernetes providers.<br/>
  Built with <a href="https://sli.dev">Slidev</a>, shipped as a container image and a Helm chart.
</p>

<p align="center">
  <a href="https://github.com/rogerwesterbo/viti/actions/workflows/ci.yml"><img src="https://github.com/rogerwesterbo/viti/actions/workflows/ci.yml/badge.svg" alt="CI" /></a>
  <a href="https://github.com/rogerwesterbo/viti/actions/workflows/release.yml"><img src="https://github.com/rogerwesterbo/viti/actions/workflows/release.yml/badge.svg" alt="Release" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/rogerwesterbo/viti" alt="License" /></a>
</p>

---

## What's in the deck

- Why Vitistack was built
- What Vitistack is
- Operators and the Kubernetes control loop
- Creating a cluster with a `NetworkNamespace` and a `KubernetesCluster`
- Demo and architecture diagrams
- Experiences so far, and what's next

A PDF export is checked in as [`Vitistack - Slidev.pdf`](Vitistack%20-%20Slidev.pdf).

More about Vitistack: [vitistack.io](https://vitistack.io) · [docs.vitistack.io](https://docs.vitistack.io) · [github.com/vitistack](https://github.com/vitistack)

## Run it locally

Requires Node.js 22 or newer.

```bash
npm install
npm run dev       # http://localhost:3030  (presenter mode: /presenter)
npm run build     # static site in dist/
npm run export    # PDF export (uses playwright-chromium)
```

## Run it with Docker

The image builds the deck and serves it with [nginx-unprivileged](https://hub.docker.com/r/nginxinc/nginx-unprivileged) on port **8080**. It runs as a non-root user and works with a read-only root filesystem.

```bash
# released image
docker run --rm -p 8080:8080 ghcr.io/rogerwesterbo/viti:latest

# or build it yourself
docker build -t viti .
docker run --rm -p 8080:8080 viti
```

Then open <http://localhost:8080>.

## Deploy to Kubernetes with Helm

The chart is published as an OCI artifact to `oci://ghcr.io/rogerwesterbo/helm/viti`.

```bash
helm install viti oci://ghcr.io/rogerwesterbo/helm/viti \
  --version <version> \
  --namespace viti --create-namespace
```

By default it creates a `ClusterIP` service only. To try it out:

```bash
kubectl -n viti port-forward svc/viti 8080:8080
```

### Exposing it

Gateway API (e.g. Envoy Gateway):

```yaml
# values-gateway.yaml
httpRoute:
  enabled: true
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
      sectionName: https-viti
  hostnames:
    - viti.example.com
```

Ingress:

```yaml
# values-ingress.yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: viti.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: viti-tls
      hosts:
        - viti.example.com
```

```bash
helm upgrade --install viti oci://ghcr.io/rogerwesterbo/helm/viti \
  --version <version> -n viti -f values-gateway.yaml
```

### Common values

| Key | Default | Description |
| --- | --- | --- |
| `image.repository` | `ghcr.io/rogerwesterbo/viti` | Container image |
| `image.tag` | chart `appVersion` | Image tag override |
| `imagePullSecrets` | `[]` | Set this if the ghcr package is private, e.g. `[{name: ghcr-image-pull-secret}]` |
| `replicaCount` | `1` | Replicas (ignored when autoscaling is enabled) |
| `service.port` | `8080` | Service and container port |
| `httpRoute.enabled` | `false` | Create a Gateway API `HTTPRoute` |
| `ingress.enabled` | `false` | Create an `Ingress` |
| `autoscaling.enabled` | `false` | Create a `HorizontalPodAutoscaler` |
| `resources` | 10m/32Mi requests, 100m/128Mi limits | Pod resources |

See [`charts/viti/values.yaml`](charts/viti/values.yaml) for every option.

## Releasing

Push a semver tag:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The [release workflow](.github/workflows/release.yml) then:

1. Builds a multi-arch (`linux/amd64`, `linux/arm64`) image and pushes it to `ghcr.io/rogerwesterbo/viti` with the tags `1.0.0`, `1.0`, `1` and `sha-<commit>`.
2. Packages the Helm chart with `version` and `appVersion` set to `1.0.0` and pushes it to `oci://ghcr.io/rogerwesterbo/helm`.

The chart's `image.tag` defaults to its `appVersion`, so chart `1.0.0` always deploys image `1.0.0`.

[CI](.github/workflows/ci.yml) runs on every push and pull request to `main`. It builds the deck, lints and renders the chart, and builds the image without pushing it.

## Repository layout

| Path | Contents |
| --- | --- |
| `slides.md` | The deck |
| `components/`, `global-top.vue`, `styles/` | Vue components, the logo overlay and custom styles |
| `public/` | Images and exported diagrams |
| `diagrams/*.excalidraw` | Diagram sources. Open them in [excalidraw.com](https://excalidraw.com) and export SVGs to `public/` |
| `manifests/` | `NetworkNamespace` and `KubernetesCluster` YAML used in the demo |
| `Dockerfile`, `nginx/` | Container image and nginx config |
| `charts/viti/` | Helm chart |
| `.github/workflows/` | CI and release pipelines |

## License

[Apache 2.0](LICENSE)
