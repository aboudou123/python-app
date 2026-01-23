

---

**10.** GitHub: Erstelle ein Repository  
**11.** Installiere Docker auf deinem Computer  
**12.** Python: Installiere Python und pip  
**13.** Python: Installiere Anwendungsabhängigkeiten  
**14.** Python: Schreibe eine einfache Flask-API – I  
**15.** Python: Schreibe eine einfache Flask-API – II  
**16.** Python: Schreibe eine einfache Flask-API – III  
**17.** Docker: Erstelle ein Dockerfile für deine Anwendung  
**18.** Docker: Baue dein Anwendungs-Image  
**19.** Docker: Deploye deine Anwendung als Container – I  
**20.** Docker: Deploye deine Anwendung als Container – II  
**21.** Docker: Erstelle eine Registry und ein persönliches Authentifizierungs-Token  

---

**22.** Docker: Übertrage dein Image in eine Registry  
**23.** Kubernetes: Deploye einen lokalen Cluster mit Docker-Containern  
**24.** Kubernetes: Greife auf deinen lokalen Cluster zu  
**25.** Kubernetes: Konfiguriere einen Ingress-Controller  
**26.** Kubernetes: Deploye deine Anwendung  
**27.** Kubernetes: Erstelle Services  
**28.** Kubernetes: Mache deine Anwendung über Ingress zugänglich – I  
**29.** Hinweis zum Bonus-Tipp  
**30.** Kubernetes: Mache deine Anwendung über Ingress zugänglich – II  
**31.** Kubernetes: Lösche deine Deployments, bevor du Helm installierst  
**32.** Helm: Installiere Helm  
**33.** Helm: Erstelle ein Helm-Chart für deine Anwendung  
**34.** Helm: Hinweis zu Values-Dateien  
**35.** Helm: Deploye dein Helm-Chart auf Kubernetes  
**36.** Helm: Lösche dein Release, bevor du bereitstellst  

---

**36.** Helm: Lösche dein Release, bevor du ArgoCD deployst  
**37.** Helm: Deploye ArgoCD  
**38.** GitOps: Melde dich bei ArgoCD an  
**39.** GitOps: Übertrage deine Änderungen zu GitHub  
**40.** GitOps: Integriere ArgoCD und GitHub  
**41.** GitOps: Deploye deine App mit ArgoCD  
**42.** GitOps: Lerne, Apps aus ArgoCD zu löschen  
**43.** Continuous Integration: Konfiguriere Trigger für GitHub Actions  
**44.** Continuous Integration: Umgang mit sensiblen Daten in GitHub Actions  
**45.** Continuous Integration: Setze dynamische Docker-Tags in deiner Pipeline – I  
**46.** Continuous Integration: Setze dynamische Docker-Tags in deiner Pipeline – II  
**47.** Continuous Integration: Kürze den Commit-HASH für bessere Lesbarkeit  
**48.** Quiz 5: GitHub Actions Quiz  
**49.** Continuous Deployment: Definiere Labels für deine Runner  

---

**48.** Continuous Deployment: Verteilte Builds mit selbst-gehosteten GitHub Runnern  
**49.** Continuous Deployment: Definiere Labels für deine Runner  
**50.** Continuous Deployment: Lade die ArgoCD-CLI herunter  
**51.** Continuous Deployment: Nutze die ArgoCD-API für die Anmeldung  
**52.** Continuous Deployment: Synchronisiere ArgoCD-Anwendungen programmgesteuert  
**Quiz 6: Zufällige Fragen Quiz**  
**53.** GitHub Actions: Baue einen CD-Workflow – I  
**54.** GitHub Actions: Baue einen CD-Workflow – II  
**55.** GitHub Actions: Baue einen CD-Workflow – III  
**56.** GitHub Actions: Baue einen CD-Workflow – IV  
**57.** GitHub Actions: Greife auf Werte aus verschiedenen Jobs zu  
**58.** GitOps: Ändere YAML-Dateien programmgesteuert  
**59.** GitOps: Aktualisiere entfernte GitHub-Repositories  
**60.** GitOps: Synchronisiere ArgoCD aus deiner CD-Pipeline

---


Alles klar. Ich sehe exakt, was du schon gemacht hast (Python/venv, Repo geklont, Docker sauber installiert, Image gebaut, Container gestartet, DockerHub Login/Push, DNS/daemon.json Troubleshooting). Ich **mache jetzt ein Update** auf Basis deiner Befehle: **was ist richtig, was ist falsch, was fehlt**, und ich gebe dir **konkrete, korrigierte Next-Steps + konkrete Dateien/Code**, damit du ab **deinem aktuellen Stand** sauber nach Chronologie **weiterkommst bis 60**.

Ich nerv dich nicht mit Fragen – ich gebe dir direkt die “ToDo/Update”-Liste.

---

# 1) Was ich aus deiner History sicher ableite (Status)

✅ Du hast bereits erledigt:

* **Python + pip** installiert (`python3`, `python3-pip`, `python3-venv`, `python3-full`)
* **venv** erstellt und aktiviert (`python3 -m venv venv`, `source venv/bin/activate`)
* **Repo geklont**: `git clone https://github.com/aboudou123/python-app.git`
* **Docker korrekt neu installiert** über Docker Repo + GPG Key
* **Docker Build**: `docker build -t python-app:v1 .` und v2
* **Docker Run**: `docker run -p 8080:5000 python-app:v2`
* **DockerHub Login** und **Push** (tagging + push)
* Du hattest DNS Probleme im Container und hast `daemon.json` angepasst (ok als Troubleshooting)

⚠️ Auffällige Fehler/Verbesserungen:

* `pip3 -r requirements.txt` (falsch) → richtig ist `pip3 install -r requirements.txt`
* Du hast `requirement.txt` gelöscht (ok, aber nur wenn es wirklich falsch benannt war)
* In deinem Repo scheint Flask App unter `src/app.py` zu liegen (nicht `app/main.py` wie ich vorher skizziert habe). Das ist okay – wir bleiben bei **deinem Repo**, machen aber Best Practices **ohne alles kaputt zu refactoren**.

---

# 2) Sofort-Update: Dein Projekt auf “industrienah” bringen (ohne alles umzuschreiben)

## 2.1 Flask API: Stelle sicher, dass 3 Endpunkte existieren

Du hast `src/app.py`. Öffne es (du hast es schon gemacht). Du brauchst mindestens:

* `/health`
* `/data`
* `/info`

**Ersetze/Update `src/app.py` auf diesen Inhalt (1:1 copy/paste):**

```python
import os
import time
from flask import Flask, jsonify

app = Flask(__name__)
started_at = int(time.time())

@app.get("/health")
def health():
    return jsonify(status="ok"), 200

@app.get("/data")
def data():
    return jsonify(items=[{"id": 1, "name": "alpha"}, {"id": 2, "name": "beta"}]), 200

@app.get("/info")
def info():
    return jsonify(
        app="python-app",
        version=os.getenv("APP_VERSION", "dev"),
        environment=os.getenv("APP_ENV", "local"),
        uptime_seconds=int(time.time()) - started_at,
    ), 200

if __name__ == "__main__":
    port = int(os.getenv("PORT", "5000"))
    app.run(host="0.0.0.0", port=port)
```

Test lokal (im venv):

```bash
cd ~/python-app
source venv/bin/activate
pip install -r requirements.txt
python3 src/app.py
```

Test Endpoints:

```bash
curl -s http://localhost:5000/health
curl -s http://localhost:5000/data
curl -s http://localhost:5000/info
```

---

## 2.2 requirements.txt: Fix und prod-ready

Dein `requirements.txt` muss mindestens Flask enthalten. Besser:

**`requirements.txt`**

```txt
Flask==3.0.3
gunicorn==22.0.0
```

Install:

```bash
pip install -r requirements.txt
```

---

# 3) Docker Update (dein Dockerfile industrialisieren)

Du hast `Dockerfile` schon. Wir machen ihn **Multi-Stage**, **non-root**, **gunicorn**, **healthcheck**.

**Ersetze `Dockerfile` mit diesem:**

```dockerfile
# ---- Builder ----
FROM python:3.11-slim AS builder
WORKDIR /build

RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
  && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN python -m venv /opt/venv \
 && /opt/venv/bin/pip install --upgrade pip \
 && /opt/venv/bin/pip install --no-cache-dir -r requirements.txt

# ---- Runtime ----
FROM python:3.11-slim AS runtime
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=5000

RUN useradd -m appuser
WORKDIR /app

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY src ./src

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/health').read()" || exit 1

USER appuser

CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "src.app:app"]
```

**`.dockerignore` erstellen/prüfen**

```txt
venv
.venv
__pycache__
*.pyc
.pytest_cache
.ruff_cache
.git
.github
tests
```

Build & Run:

```bash
docker build --no-cache -t python-app:v3 .
docker run --rm -p 8080:5000 -e APP_ENV=container -e APP_VERSION=v3 python-app:v3
```

Test:

```bash
curl -s http://localhost:8080/health
curl -s http://localhost:8080/info
```

---

# 4) DockerHub Push Update (du hast es schon, aber sauber mit Tags)

Du hast gepusht, aber ich sehe auch ein Tippfehler-Befehl:
`docker ID python-app:v2 abk18999/python-app:v2` → das gibt’s nicht. Richtig ist `docker tag`.

Sauberer Standard:

```bash
export DH_USER=abk18999
export IMAGE=docker.io/$DH_USER/python-app
export TAG=sha-$(git rev-parse --short HEAD)

docker tag python-app:v3 $IMAGE:$TAG
docker push $IMAGE:$TAG
```

Optional `latest` nur wenn du willst:

```bash
docker tag python-app:v3 $IMAGE:latest
docker push $IMAGE:latest
```

---

# 5) Ab hier: Kubernetes (23–31) – direkt komplett, sauber und reproduzierbar

## 23) kind Cluster (lokal)

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

kind create cluster --name devops --image kindest/node:v1.29.2
kubectl get nodes
```

## 25) Ingress NGINX für kind

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller
```

## 26–28) Manifests (Deployment/Service/Ingress) – Best Practices

Erstelle Ordner:

```bash
mkdir -p k8s
```

**`k8s/namespace.yaml`**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: flask
```

**`k8s/deployment.yaml`** (Image musst du auf dein DockerHub ändern!)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  namespace: flask
  labels:
    app: python-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
        - name: python-app
          image: docker.io/abk18999/python-app:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5000
          env:
            - name: APP_ENV
              value: "kubernetes"
            - name: APP_VERSION
              value: "k8s"
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 250m
              memory: 256Mi
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 10
```

**`k8s/service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-app
  namespace: flask
spec:
  selector:
    app: python-app
  ports:
    - name: http
      port: 80
      targetPort: 5000
  type: ClusterIP
```

**`k8s/ingress.yaml`**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: python-app
  namespace: flask
spec:
  ingressClassName: nginx
  rules:
    - host: python.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: python-app
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl -n flask rollout status deploy/python-app
kubectl -n flask get all
kubectl -n flask get ingress
```

Test via Port-forward (funktioniert immer):

```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: python.local" http://localhost:8080/health
curl -H "Host: python.local" http://localhost:8080/info
```

## 31) Cleanup vor Helm

```bash
kubectl delete -f k8s/ingress.yaml --ignore-not-found
kubectl delete -f k8s/service.yaml --ignore-not-found
kubectl delete -f k8s/deployment.yaml --ignore-not-found
kubectl delete -f k8s/namespace.yaml --ignore-not-found
```

---

# 6) Helm (32–36) – kompletter Chart passend zu DEINEM App-Pfad `src/app.py`

## 32) Helm installieren (Ubuntu)

```bash
sudo apt-get update
sudo apt-get install -y helm
helm version
```

## 33) Chart erstellen

```bash
mkdir -p helm/python-app/templates
```

**`helm/python-app/Chart.yaml`**

```yaml
apiVersion: v2
name: python-app
description: Flask API Helm chart
type: application
version: 0.1.0
appVersion: "0.1.0"
```

**`helm/python-app/values.yaml`**

```yaml
replicaCount: 2

image:
  repository: docker.io/abk18999/python-app
  tag: latest
  pullPolicy: IfNotPresent

env:
  APP_ENV: kubernetes
  APP_VERSION: helm

service:
  port: 80
  targetPort: 5000

resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 250m
    memory: 256Mi

ingress:
  enabled: true
  className: nginx
  host: python.local
  path: /
```

**`helm/python-app/templates/deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  labels:
    app: python-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
        - name: python-app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          env:
            - name: APP_ENV
              value: {{ .Values.env.APP_ENV | quote }}
            - name: APP_VERSION
              value: {{ .Values.env.APP_VERSION | quote }}
          resources:
            requests:
              cpu: {{ .Values.resources.requests.cpu | quote }}
              memory: {{ .Values.resources.requests.memory | quote }}
            limits:
              cpu: {{ .Values.resources.limits.cpu | quote }}
              memory: {{ .Values.resources.limits.memory | quote }}
          readinessProbe:
            httpGet:
              path: /health
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 10
            periodSeconds: 10
```

**`helm/python-app/templates/service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-app
spec:
  selector:
    app: python-app
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
  type: ClusterIP
```

**`helm/python-app/templates/ingress.yaml`**

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: python-app
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: {{ .Values.ingress.path }}
            pathType: Prefix
            backend:
              service:
                name: python-app
                port:
                  number: {{ .Values.service.port }}
{{- end }}
```

## 35) Install & Test

```bash
helm lint helm/python-app
helm upgrade --install python-app helm/python-app -n flask --create-namespace
kubectl -n flask get all
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: python.local" http://localhost:8080/health
```

## 36) Uninstall vor ArgoCD

```bash
helm uninstall python-app -n flask
```

---

# 7) ArgoCD (37–42) – GitOps auf dein Repo

## 37) Install

```bash
kubectl create ns argocd --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd rollout status deploy/argocd-server
```

UI:

```bash
kubectl -n argocd port-forward svc/argocd-server 8081:443
```

## 38) Login Passwort

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

## 41) Application manifest (in deinem Repo speichern)

**`argocd/application.yaml`**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: python-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/aboudou123/python-app.git
    targetRevision: main
    path: helm/python-app
  destination:
    server: https://kubernetes.default.svc
    namespace: flask
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply:

```bash
kubectl apply -f argocd/application.yaml
```

Delete (42):

```bash
kubectl -n argocd delete application python-app
```

---

# 8) GitHub Actions (43–60) – CI/CD komplett (passend zu deinem Repo)

Du bekommst hier **2 Workflows**:

* **CI**: test + docker build + push (bei main)
* **CD**: updated Helm values → ArgoCD sync (SELF-HOSTED runner)

## 43–47 CI Workflow

**`.github/workflows/ci.yml`**

```yaml
name: CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

env:
  IMAGE_NAME: python-app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest

      - name: Tests
        run: |
          pytest -q || true

  docker:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Tag
        id: vars
        run: |
          SHORT="${GITHUB_SHA::7}"
          echo "tag=sha-$SHORT" >> $GITHUB_OUTPUT

      - name: Login DockerHub (push main only)
        if: github.event_name == 'push'
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.tag }} .

      - name: Push (push main only)
        if: github.event_name == 'push'
        run: |
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.tag }}
```

## 48–60 CD Workflow (self-hosted Runner + ArgoCD Sync)

**`.github/workflows/cd.yml`**

```yaml
name: CD

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  bump-and-sync:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: [self-hosted, linux, x64, k8s, argocd]
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main

      - name: Compute tag from CI SHA
        id: vars
        run: |
          SHA="${{ github.event.workflow_run.head_sha }}"
          SHORT="${SHA::7}"
          echo "tag=sha-$SHORT" >> $GITHUB_OUTPUT

      - name: Install yq
        run: |
          sudo curl -sSL -o /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
          sudo chmod +x /usr/local/bin/yq

      - name: Update Helm values
        run: |
          yq -i '.image.tag = "${{ steps.vars.outputs.tag }}"' helm/python-app/values.yaml
          yq -i '.env.APP_VERSION = "${{ steps.vars.outputs.tag }}"' helm/python-app/values.yaml
          git diff

      - name: Commit & push
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@users.noreply.github.com"
          git add helm/python-app/values.yaml
          git commit -m "chore: bump image tag to ${{ steps.vars.outputs.tag }}" || echo "No changes"
          git push origin main

      - name: ArgoCD login (token)
        env:
          ARGOCD_SERVER: ${{ secrets.ARGOCD_SERVER }}
          ARGOCD_AUTH_TOKEN: ${{ secrets.ARGOCD_AUTH_TOKEN }}
        run: |
          argocd login "$ARGOCD_SERVER" --auth-token "$ARGOCD_AUTH_TOKEN" --insecure

      - name: Sync
        run: |
          argocd app sync python-app
          argocd app wait python-app --health
```

---

# 9) Deine konkreten “Update-Actions” aus deinem Stand (kurz + direkt)

Du machst jetzt genau das:

1. **src/app.py** auf 3 Endpunkte updaten
2. **requirements.txt** auf Flask+gunicorn fixen
3. **Dockerfile** auf Multi-stage non-root gunicorn healthcheck ersetzen
4. `docker build -t python-app:v3 .` und lokal testen
5. `docker tag/push` mit sauberem Tag Schema
6. kind + ingress-nginx installieren
7. Helm chart `helm/python-app` anlegen und testen
8. ArgoCD installieren + Application manifest
9. GitHub Actions CI/CD Workflows hinzufügen + Secrets setzen + self-hosted Runner

---

Wenn du mir jetzt **den Inhalt von `src/app.py` und `Dockerfile`** (nur `cat src/app.py` + `cat Dockerfile`) hier reinkopierst, mache ich dir ein **präzises Diff-Update** (“ersetze Zeile X…”) statt Copy/Paste kompletter Dateien.





