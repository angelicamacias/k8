# Determining Service Port Numbers

[Back to the study notes](README.md#services)

To write a Service you need to answer **three questions**, based on where the traffic is **coming from** and where it is **going**:

```text
  Outside the cluster          Inside the cluster              Inside the pod
┌───────────────────┐      ┌──────────────────────┐      ┌───────────────────────┐
│  <Node-IP>:30082  │ ───► │   my-service:80      │ ───► │  container :8080      │
│  (every node)     │      │   (ClusterIP / DNS)  │      │  (the app process)    │
└───────────────────┘      └──────────────────────┘      └───────────────────────┘
       nodePort                     port                        targetPort
```

| Port | Lives on | Used by |
| :--- | :--- | :--- |
| `targetPort` | The container | The Service, to forward traffic to the app |
| `port` | The Service (ClusterIP / DNS name) | Other pods inside the cluster |
| `nodePort` | Every node in the cluster | Clients outside the cluster |

---

## 1. `targetPort` — the container's application port

**What it is:** the actual port the application process inside the container is bound to.

**How to find it:**

- Look at the Pod/Deployment definition under `spec.containers[*].ports[*].containerPort`.
- Check the application configuration or the Docker image default:

  | Application | Default port |
  | :--- | :--- |
  | Nginx | `80` |
  | Redis | `6379` |
  | PostgreSQL | `5432` |
  | Custom Node / Java / Python apps | often `8080`, `3000` or `5000` |

> [!TIP]
> If an exam question says *"The web app listens on port 8080"* → `targetPort: 8080`.

---

## 2. `port` — the Service's internal virtual port

**What it is:** the port that other pods inside the cluster use to reach the Service through its ClusterIP or DNS name (`my-service:port`).

**How to find it:**

- In exam questions it is stated explicitly: *"Expose the service on port 80"* or *"Port: 6379"*.
- In real architectures, standard protocol ports are usually chosen (`80` for HTTP, `443` for HTTPS, `6379` for Redis), even when the container underneath uses a non-standard port like `8080`.

> [!NOTE]
> If `targetPort` is omitted in the YAML, Kubernetes sets it to the same value as `port`.

---

## 3. `nodePort` — the node's external entry port

**What it is:** a dedicated port opened on the network interface of **every node** in the cluster, so users outside the cluster can reach the workload at `<Node-IP>:<nodePort>`.

**How to find it:**

- It must fall within the default Kubernetes NodePort range: **`30000–32767`**.
- In exam tasks, when it's required, the question states it explicitly: *"accessible on port 30082 on the nodes of the cluster"*.

> [!NOTE]
> If you set `type: NodePort` without a `nodePort`, Kubernetes picks a random unused port between `30000` and `32767`.

---

## Port Decision Matrix for the Exam

| Scenario | Service Type | `port` | `targetPort` | `nodePort` |
| :--- | :--- | :--- | :--- | :--- |
| **Internal only** (e.g., Database, Cache) | `ClusterIP` | Given in task (e.g., `6379`) | Port app runs on (e.g., `6379`) | *None* |
| **Internal app** with custom container port | `ClusterIP` | Standard port (e.g., `80`) | App port (e.g., `8080`) | *None* |
| **External access** via cluster nodes | `NodePort` | Service port (e.g., `8080`) | App port (e.g., `8080`) | High port (`30000–32767`, e.g., `30082`) |

---

## Putting it together

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80          # other pods call webapp-service:80
      targetPort: 8080  # the app listens on 8080 inside the container
      nodePort: 30082   # outside users call <Node-IP>:30082
```
