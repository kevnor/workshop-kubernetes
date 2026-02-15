# LAB 04 – ConfigMaps & Secrets

## Læringsmål

Etter denne laben skal du kunne:

* Opprette en ConfigMap
* Opprette en Secret
* Bruke dem i en Deployment
* Se miljøvariabler i en Pod
* Forstå forskjellen på ConfigMap og Secret

---

# Del 1 – ConfigMap

## 1. Opprett ConfigMap via YAML

Lag `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "development"
  LOG_LEVEL: "debug"
```

Apply:

```bash
kubectl apply -f configmap.yaml
```

Sjekk:

```bash
kubectl get configmap -n <namespace>
kubectl describe configmap app-config -n <namespace>
```

---

# Del 2 – Secret

## 1. Opprett Secret via YAML

Lag `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: <namespace>
type: Opaque
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=
```

🔎 Forklaring:
`c3VwZXJzZWNyZXQ=` er base64 for `supersecret`.

Du kan lage base64 selv med:

```bash
echo -n "supersecret" | base64
```

Apply:

```bash
kubectl apply -f secret.yaml
```

Sjekk:

```bash
kubectl get secret -n <namespace>
kubectl describe secret app-secret -n <namespace>
```

---

# Del 3 – Bruke ConfigMap og Secret i Deployment

Lag `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web-container
        image: nginx:1.25
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Sjekk:

```bash
kubectl get pods -n lab03
```

---

# Del 4 – Verifiser inne i Pod

Finn pod-navn:

```bash
kubectl get pods -n lab03
```

Gå inn i poden:

```bash
kubectl exec -it <podnavn> -n lab03 -- /bin/sh
```

Sjekk miljøvariabler:

```bash
env | grep APP_ENV
env | grep LOG_LEVEL
env | grep DB_PASSWORD
```

Du skal se:

```
APP_ENV=development
LOG_LEVEL=debug
DB_PASSWORD=supersecret
```

Avslutt:

```bash
exit
```

---

# Refleksjonsspørsmål

1. Hva er forskjellen på ConfigMap og Secret?
2. Hvorfor er Secret base64-enkodet?
3. Er Secret automatisk kryptert?
4. Må vi bygge nytt image for å endre konfig?

---

# Del 6 – Endre ConfigMap

Endre `APP_ENV` til:

```yaml
APP_ENV: "production"
```

Apply igjen:

```bash
kubectl apply -f configmap.yaml
```

⚠️ Viktig:

Eksisterende Pods får ikke automatisk oppdatert env-vars.

Restart deployment:

```bash
kubectl rollout restart deployment web -n lab03
```

Sjekk igjen med `exec`.

👉 Diskusjon:
Hvorfor må vi restarte når vi bruker env vars?

---

# Viktigste læringspoeng

* ConfigMap = vanlig konfig
* Secret = sensitiv konfig
* Image skal være miljø-uavhengig
* Konfig injiseres via Kubernetes

---
