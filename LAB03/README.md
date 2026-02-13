# LAB 02 – Deployments + YAML + Service

## Læringsmål

Etter denne laben skal du kunne:

* Deploye en app med **YAML** (Deployment)
* Forstå labels/selectors
* Skalere antall pods
* Opprette en **Service** som gir stabil tilgang
* Feilsøke når ting ikke fungerer

---

## Forutsetninger

* Du har tilgang til et Kubernetes-cluster (lokalt eller AKS)
* `kubectl` fungerer:

```bash
kubectl get nodes
```

---

# Del A — Deployment med YAML

## 1) Lag `deployment.yaml`

Lim inn:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Oppgave (forstå YAML)

Finn og forklar:

* Hvor settes antall replicas?
* Hvilken label får Podene?
* Hvilket image brukes?

---

## 2) Deploy

```bash
kubectl apply -f deployment.yaml -n <namespace>
```

Sjekk:

```bash
kubectl get deployment -n <namespace>
kubectl get pods -l app=web -n <namespace>
```

---

## 3) Inspiser deployment/pods

```bash
kubectl describe deployment web -n <namespace>
kubectl describe pod -l app=web -n <namespace> | head -n 40
```

👉 Oppgave:

* Hvilke events ser du på deployment/pod?
* Hvilken node kjører podene på?

---

# Del B — Skalering

Skaler opp til 4 replicas:

```bash
kubectl scale deployment web --replicas=4 -n <namespace>
kubectl get pods -l app=web -n <namespace>
```

👉 Oppgaver:

* Hvor mange pods kjører nå?
* Hva skjer hvis du sletter en pod?

```bash
kubectl delete pod -l app=web -n <namespace> --field-selector=status.phase=Running
kubectl get pods -l app=web -n <namespace>
```

(Det skal fortsatt ende med 4 pods – desired state)

---

# Del C — Rolling update (image-oppdatering)

Oppdater image med YAML (anbefalt workflow):

1. Endre i `deployment.yaml`:

```yaml
image: nginx:1.26
```

2. Apply:

```bash
kubectl apply -f deployment.yaml -n <namespace>
```

3. Følg rollout:

```bash
kubectl rollout status deployment/web -n <namespace>
kubectl get pods -l app=web -n <namespace>
```

👉 Oppgave:

* Ser du at nye pods blir laget og gamle fjernes gradvis?

Bonus:

```bash
kubectl rollout history deployment/web -n <namespace>
```

---

# Del D — Service (stabil tilgang)

## 1) Lag `service.yaml`

Lim inn:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Forklaring:

* Service velger pods via `selector`
* Gir fast DNS/IP internt i clusteret

---

## 2) Opprett service

```bash
kubectl apply -f service.yaml -n <namespace>
kubectl get svc -n <namespace>
```

👉 Oppgave:

* Hvilken `CLUSTER-IP` fikk `web-svc`?

---

## 3) Test service med port-forward

```bash
kubectl port-forward svc/web-svc 8080:80 -n <namespace>
```

Åpne:

* `http://localhost:8080`

Stop med `CTRL+C`.

---

# Del E — Label/selector (viktig konsept)

## 1) Se labels på pods

```bash
kubectl get pods --show-labels -l app=web -n <namespace>
```

## 2) Se “endpoints” service peker på

```bash
kubectl get endpoints web-svc -n <namespace>
```

👉 Oppgave:

* Hvor mange endpoints ser du?
* Matcher det antall replicas?

---

# Mini-quiz (muntlig)

1. Hva er forskjellen på Pod og Deployment?
2. Hvorfor trenger vi Service?
3. Hva kobler Deployment og Service sammen?
4. Hvorfor bruker vi `kubectl apply` og YAML i stedet for bare `kubectl run`?

---

# 🧹 Rydd opp

Når dere er ferdig:

```bash
kubectl delete -f service.yaml -n <namespace>
kubectl delete -f deployment.yaml -n <namespace>
```

---

# Bonus (valgfritt): Eksponer ut av clusteret

Bytt service type til NodePort:

I `service.yaml`:

```yaml
type: NodePort
```

Apply:

```bash
kubectl apply -f service.yaml -n <namespace>
kubectl get svc web-svc -n <namespace>
```

👉 Oppgave:

* Finn NodePort
* Diskuter: hvorfor er dette ofte “ok for demo”, men ikke alltid for prod?

---
