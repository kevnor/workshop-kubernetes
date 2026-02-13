# 🐳 LAB 01 – Fra Dockerfile til kjørende container

## 🎯 Mål

Etter denne laben skal du kunne:

* Forstå hva et Docker image er
* Lage en enkel Dockerfile
* Bygge et image
* Starte en container
* Forstå forskjellen på image og container
* Gjøre enkel feilsøking

---

# Hva vi skal lage

Vi skal:

1. Lage en enkel Python-webserver
2. Lage en Dockerfile
3. Bygge et image
4. Starte en container
5. Åpne applikasjonen i nettleser

---

# Steg 1 – Se på applikasjonen

Åpne `app.py`.

Den starter en enkel HTTP-server på port `8000`.

👉 Spørsmål:

* Hvilken port kjører appen på?
* Hva skjer når vi starter den lokalt?

Test lokalt (valgfritt):

```bash
python3 app.py
```

Gå til:

```
http://localhost:8000
```

Avslutt med `CTRL + C`.

---

# Steg 2 – Se på Dockerfile

Åpne `Dockerfile`.

Gå gjennom linjene og prøv å forstå:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python", "app.py"]
```

👉 Oppgave:
Forklar hva disse gjør:

* `FROM`
* `COPY`
* `CMD`
* `EXPOSE`

---

# Steg 3 – Bygg imaget

Kjør:

```bash
docker build -t min-webapp .
```

Forklaring:

* `-t` = navn på imaget
* `.` = bygg fra denne mappen

Sjekk at imaget finnes:

```bash
docker images
```

👉 Oppgave:

* Ser du `min-webapp`?
* Hvilket base-image ble brukt?

---

# Steg 4 – Start container

Kjør:

```bash
docker run -p 8000:8000 min-webapp
```

Forklaring:

```
-p 8000:8000
host-port : container-port
```

Åpne i nettleser:

```
http://localhost:8000
```

👉 Oppgave:

* Hva skjer hvis du stopper containeren?
* Hva skjer hvis du starter den igjen?

---

# Steg 5 – Se hva som kjører

Åpne en ny terminal og kjør:

```bash
docker ps
```

Stopp containeren:

```bash
docker stop <container-id>
```

Sjekk igjen:

```bash
docker ps
```

---

# 🧠 Refleksjonsspørsmål

1. Hva er forskjellen på et image og en container?
2. Kan vi starte flere containere fra samme image?
3. Hvorfor må vi bruke `-p`?
4. Hva skjer hvis vi prøver å bruke samme port to ganger?

---

# 🧪 Bonusoppgave

Start en ny container på en annen port:

```bash
docker run -p 8001:8000 min-webapp
```

Åpne:

```
http://localhost:8001
```

👉 Hva betyr det at containeren fortsatt bruker port 8000 internt?

---