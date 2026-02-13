
---

# LAB: Lag din første Pod i AKS (med autentisering)

## Læringsmål

Etter laben skal du kunne:

* Koble `kubectl` til et AKS-cluster (autentisering)
* Lage en Pod
* Sjekke status, logs og detaljer
* Kjøre kommandoer inni Poden (`exec`)
* Rydde opp igjen

---

## Forutsetninger

Du må ha installert:

* **Azure CLI** (`az`)
* **kubectl**
* **kubelogin** (ofte nødvendig i AKS med Microsoft Entra ID / Azure RBAC) ([Microsoft Learn][1])

Installer Azure CLI:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

---

# Del A — Autentisering mot AKS (koble kubectl til clusteret)

## 1) Logg inn i Azure

Kjør:

```bash
az login
```

Hvis du har flere subscriptions, velg riktig:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

## 2) Hent kubeconfig fra AKS

Kjør:

```bash
az aks get-credentials -g "<RESOURCE_GROUP>" -n "<AKS_CLUSTER_NAME>" --overwrite-existing
```

Dette legger cluster-context inn i `~/.kube/config` (kubeconfig). ([Microsoft Learn][2])

## 3) Hvis clusteret bruker Entra/Azure RBAC: konverter kubeconfig med kubelogin

Mange AKS-oppsett krever `kubelogin` for innlogging med Entra. ([Microsoft Learn][1])

Vanligste og enkleste:

```bash
kubelogin convert-kubeconfig -l azurecli
```

(Dette gjør at `kubectl` bruker din Azure CLI-login.) ([Microsoft Learn][1])

## 4) Test at du har tilgang

```bash
kubectl cluster-info
kubectl get nodes
```

---

# Del B — Lag en Pod (enkelt og konkret)

## 1) Lag et namespace for laben

```bash
kubectl create namespace lab01
```

Sjekk:

```bash
kubectl get ns
```

## 2) Lag en Pod (nginx)

```bash
kubectl run nginx --image=nginx --restart=Never -n lab01
```

> `--restart=Never` gjør dette til en “ren” Pod (ikke Deployment).

## 3) Se status

```bash
kubectl get pods -n lab01
```

Vent til du ser `Running`.

## 4) Inspiser Poden (viktig!)

```bash
kubectl describe pod nginx -n lab01
```

👉 Oppgaver:

* Finn **Pod IP**
* Finn **Node** den kjører på
* Se **Events** nederst (her ser du ofte feilen hvis noe stopper)

---

# Del C — Test at Poden fungerer

## 1) Port-forward til Poden

```bash
kubectl port-forward pod/nginx 8080:80 -n lab01
```

Åpne i nettleser:

* `http://localhost:8080`

Stop port-forward med `CTRL+C`.

## 2) Kjør kommando inni Poden

```bash
kubectl exec -it nginx -n lab01 -- /bin/sh
```

Inne i Poden, kjør:

```sh
uname -a
ls -la
exit
```

👉 Oppgave:

* Hva er forskjellen på å “se på” Poden og å “gå inn i” Poden?

## 3) Sjekk logs

Nginx logger lite uten trafikk, men prøv:

```bash
kubectl logs nginx -n lab01
```

---

# Del D — Rydd opp

Slett Pod + namespace:

```bash
kubectl delete pod nginx -n lab01
kubectl delete namespace lab01
```

---

# 🧠 Mini-quiz (2 minutter)

1. Hva er en Pod?
2. Hvorfor endrer Pod IP seg hvis den blir erstattet?
3. Hvor finner du feil hvis Pod ikke starter?
4. Hva gjør `kubectl describe` som `kubectl get` ikke gjør?

---

# 🛠️ Feilsøking (vanlige problemer)

## 1) “You must be logged in …” / blir bedt om å autentisere hele tiden

Dette skyldes ofte Entra/Azure RBAC-oppsett og at kubeconfig må konverteres med kubelogin.

Kjør:

```bash
kubelogin convert-kubeconfig -l azurecli
```

Prøv så:

```bash
kubectl get nodes
```

Dette er en vanlig løsning i AKS med Entra-integrasjon. ([Microsoft Learn][1])

---

## 2) “Forbidden” på `kubectl get nodes`

Da er du autentisert, men mangler **rettigheter** i clusteret (RBAC).
Typisk: du er ikke i riktig Entra-gruppe/rolle.

Hva du kan gjøre i lab:

* Prøv `kubectl get pods -A` og se om alt er forbidden
* Kontakt instruktør for å få tildelt riktig tilgang

(AKS kan styres via Azure RBAC for Kubernetes Authorization.) ([Microsoft Learn][3])

---

## 3) Pod står i `ImagePullBackOff` / `ErrImagePull`

Sjekk:

```bash
kubectl describe pod nginx -n lab01
```

Se under **Events** for feilmelding (f.eks. registry/blokkering).

---

## 4) Pod står i `Pending`

Vanlig årsak: ingen kapasitet / scheduling-problem.

Sjekk:

```bash
kubectl describe pod nginx -n lab01
```

Se under **Events** (scheduler forklarer hvorfor).

---

## 5) Context-feil: “no configuration has been provided”

Da mangler kubeconfig eller du peker på feil context.

Sjekk:

```bash
kubectl config get-contexts
kubectl config current-context
```

Hent credentials på nytt:

```bash
az aks get-credentials -g "<RG>" -n "<CLUSTER>" --overwrite-existing
```

---


[1]: https://learn.microsoft.com/en-us/azure/aks/kubelogin-authentication?utm_source=chatgpt.com "Use kubelogin to authenticate in Azure Kubernetes Service (AKS) - Azure ..."
[2]: https://learn.microsoft.com/en-us/azure/aks/control-kubeconfig-access?utm_source=chatgpt.com "Limit access to kubeconfig in Azure Kubernetes Service (AKS) - Azure ..."
[3]: https://learn.microsoft.com/en-us/azure/aks/manage-azure-rbac?utm_source=chatgpt.com "Use Azure RBAC for Kubernetes Authorization - Azure Kubernetes Service"
