# Workshop Setup – Kubernetes Lab Miljø (Windows)

## Mål

Etter dette oppsettet skal du ha:

* ✅ PowerShell 7
* ✅ WSL2
* ✅ Debian i WSL
* ✅ Azure CLI (`az`)
* ✅ kubectl
* ✅ kubelogin

Alt vi gjør videre i workshopen skjer i **WSL (Debian)**.

---

# 1️ Installer PowerShell 7

## Sjekk om du allerede har det:

Åpne Start → skriv:

```
pwsh
```

Hvis det starter → ferdig 🎉

Hvis ikke:

## Installer via winget:

Åpne vanlig PowerShell (ikke admin nødvendig):

```powershell
winget install --id Microsoft.PowerShell --source winget
```

Test:

```powershell
pwsh
$PSVersionTable
```

Du skal se versjon 7.x

---

# 2️ Installer WSL2

Åpne PowerShell som Administrator og kjør:

```powershell
wsl --install
```

Start maskinen på nytt hvis du blir bedt om det.

---

## Verifiser at WSL2 brukes:

```powershell
wsl -l -v
```

Du skal se:

```
VERSION 2
```

Hvis ikke:

```powershell
wsl --set-default-version 2
```

---

# 3️ Installer Debian i WSL

Installer Debian:

```powershell
wsl --install -d Debian
```

Start Debian fra Start-menyen.

Første gang:

* Sett brukernavn
* Sett passord

Dersom du har flere distro:

```powershell
wsl -s Debian
```

Dette setter "default" distro til å være Debian

---

# 4️ Oppdater Debian

Inne i Debian (WSL):

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 5️ Installer Azure CLI (az)

Kjør i Debian:

```bash
sudo apt install curl -y
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Test:

```bash
az version
```

---

# 6️ Installer kubectl

Installer med Azure CLI:

```bash
sudo az aks install-cli
```

Test:

```bash
kubectl version --client
```

---

# 7️ Installer kubelogin

Dette trengs for AKS med Entra ID / Azure RBAC.

Installer via Azure CLI:

```bash
sudo az aks install-cli
```

(Installerer både kubectl og kubelogin)

Alternativ metode:

```bash
curl -LO https://github.com/Azure/kubelogin/releases/latest/download/kubelogin-linux-amd64.zip
unzip kubelogin-linux-amd64.zip
sudo mv bin/linux_amd64/kubelogin /usr/local/bin/
```

Test:

```bash
kubelogin --version
```

# 8 Installer Docker

Dette trengs for å kunne lage iamges og kjøre containere lokalt.

Kjør hver kommando under (en og en) for å sette opp apt-repoet:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Installer Docker:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Sørg for at du har rett tilganger:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Restart WSL for at permissions skal oppdatere seg:

```bash
exit
wsl --shutdown
wsl
```

Gå tilbake til mappenN

```bash
cd ~/workshop/workshop-kubernetes
```


---

# 9 Test at alt fungerer

Test i WSL:

```bash
az login
```

Logg inn i nettleser.

Deretter:

```bash
az account show
```

---

# 10 Koble til AKS (gjøres under workshop)

```bash
az aks get-credentials -g <RESOURCE_GROUP> -n <CLUSTER_NAME> --overwrite-existing
kubelogin convert-kubeconfig -l azurecli
kubectl get nodes
```

---

# 11 Hent workshop-repoet fra GitHub

Alle øvelser og YAML-filer ligger i repoet:

👉 [https://github.com/kevnor/workshop-kubernetes](https://github.com/kevnor/workshop-kubernetes)

---

## 1) Gå til hjemmemappen i WSL

I Debian (WSL):

```bash
cd ~
```

Opprett en workshop-mappe (hvis du ikke allerede har):

```bash
mkdir workshop
cd workshop
```

---

## 2) Klon repoet

```bash
sudo apt install git
git clone https://github.com/kevnor/workshop-kubernetes.git
```

Gå inn i repoet:

```bash
cd workshop-kubernetes
```

---

## 3) Åpne i VS Code

For å åpne prosjektet i VS Code fra WSL:

```bash
code .
```

Hvis dette fungerer:

* VS Code åpner
* Nederst til venstre står det: **WSL: Debian**

Hvis ikke fungerer:

Installer VS Code fra:
[https://code.visualstudio.com/](https://code.visualstudio.com/)

---

# ⚠️ Viktig

Du skal jobbe i:

```
/home/<brukernavn>/workshop/workshop-kubernetes
```

* ❌ Ikke jobb i `/mnt/c/...`
* ❌ Ikke klon repoet i OneDrive

---

# ✅ Ferdig

Når dette fungerer skal du kunne:

```bash
cd ~/workshop/workshop-kubernetes
code .
```

---

# Når du er ferdig

Du skal kunne kjøre:

```bash
az version
kubectl version --client
kubelogin --version
```

Og være klar for lab 🚀

---