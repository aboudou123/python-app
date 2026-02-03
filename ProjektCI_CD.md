
---

# 0) Zielbild

# Am Ende dieser Arbeit soll ein Repository auf GitHub vorliegen, konkret mit dem Namen „flask-devops-pipeline“, das folgende Inhalte umfasst:

* `app/` Flask API (3 Endpunkte)
* `Dockerfile` (Multi-Stage, non-root, gunicorn)
* `k8s/` Kubernetes Manifests (Deployment/Service/Ingress + Best Practices)
* `helm/` Helm Chart (templates, values, helpers)
* `argocd/` ArgoCD Application (GitOps)
* `.github/workflows/` CI und CD Workflows
* `scripts/` Build/Deploy/Cleanup/Cluster-Setup-Skripte
* `README.md` komplette Doku




<img width="697" height="274" alt="image" src="https://github.com/user-attachments/assets/e164fba5-5034-4c36-b434-c1b1e1d337c3" />

---

# 10) GitHub: Erstelle ein Repository

## 10.1 GitHub Repo anlegen

* GitHub → New repository
* Name: `flask-devops-pipeline`
* Branch: `main`
* Kein Auto-README (wir machen lokal)

## 10.2 Lokal klonen oder initialisieren

```bash
mkdir flask-devops-pipeline && cd flask-devops-pipeline
git init
git branch -M main
```

## 10.3 Ordnerstruktur erstellen

```bash
mkdir -p app tests k8s helm/flask-api/templates argocd scripts .github/workflows
touch app/__init__.py app/main.py
touch tests/test_api.py
touch requirements.txt requirements-dev.txt
touch Dockerfile .dockerignore
touch k8s/namespace.yaml k8s/deployment.yaml k8s/service.yaml k8s/ingress.yaml
touch helm/flask-api/Chart.yaml helm/flask-api/values.yaml
touch helm/flask-api/templates/_helpers.tpl
touch helm/flask-api/templates/deployment.yaml helm/flask-api/templates/service.yaml helm/flask-api/templates/ingress.yaml
touch argocd/application.yaml
touch scripts/00_prereqs.sh scripts/01_local_run.sh scripts/02_docker_build_run.sh scripts/03_push_registry.sh
touch scripts/10_kind_create.sh scripts/11_ingress_install.sh scripts/12_k8s_apply.sh scripts/13_k8s_cleanup.sh
touch scripts/20_helm_install.sh scripts/21_helm_uninstall.sh
touch scripts/30_argocd_install.sh scripts/31_argocd_login.sh scripts/32_argocd_app_apply.sh scripts/33_argocd_cleanup.sh
touch README.md .gitignore
```
<img width="501" height="386" alt="2026-01-18 10_08_12-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_requirements txt (python-app)" src="https://github.com/user-attachments/assets/8ed35964-eb77-4f6f-8aae-962ca7223875" />




<img width="658" height="358" alt="2026-01-18 10_08_53-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_requirements txt (python-app)" src="https://github.com/user-attachments/assets/dedefa23-d4b0-41b0-a022-8265350d92c2" />

---

# 11) Installation Docker

## 11.1 Test

```bash
docker --version
docker run --rm hello-world
```

---

# 12) Python: Installiere Python und pip

## 12.1 Test (Python 3.9+)

```bash
python3 --version
pip3 --version
```

<img width="570" height="387" alt="2026-01-18 21_45_11-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_Dockerfile (python-app) - Sub" src="https://github.com/user-attachments/assets/c711586e-755b-426b-8d80-7263bb5f6bb4" />



<img width="935" height="529" alt="2026-01-18 21_03_41-" src="https://github.com/user-attachments/assets/62e78ef7-43a1-4445-852f-1955c2b4a241" />



---

# 13) Python: Installiere Anwendungsabhängigkeiten

## 13.1 venv erstellen

```bash
python3 -m venv .venv
# Linux/macOS
source .venv/bin/activate
# Windows PowerShell:
# .\.venv\Scripts\Activate.ps1
```

## 13.2 requirements Dateien

**`requirements.txt`**

```txt
Flask==3.0.3
gunicorn==22.0.0
```

**`requirements-dev.txt`**

```txt
pytest==8.3.2
ruff==0.5.7
```

## 13.3 Install

```bash
pip install -r requirements.txt -r requirements-dev.txt
```
<img width="501" height="386" alt="2026-01-18 10_08_12-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_requirements txt (python-app)" src="https://github.com/user-attachments/assets/bcc8e2c1-e052-4cec-903e-b1fe522b744a" />

---

# 14) Python: einfache Flask-API – I

**`app/main.py`**

```python
import os
import time
from flask import Flask, jsonify

def create_app() -> Flask:
    app = Flask(__name__)
    started_at = int(time.time())

    @app.get("/health")
    def health():
        return jsonify(status="ok"), 200

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(host="0.0.0.0", port=int(os.getenv("PORT", "5000")))
```

Start:

```bash
python -m app.main
curl -s http://localhost:5000/health
```



---

# 15) Python: einfache Flask-API – II

Erweitere um `/data`.

**`app/main.py` (ersetzen/erweitern)**

```python
import os
import time
from flask import Flask, jsonify

def create_app() -> Flask:
    app = Flask(__name__)
    started_at = int(time.time())

    @app.get("/health")
    def health():
        return jsonify(status="ok"), 200

    @app.get("/data")
    def data():
        return jsonify(items=[{"id": 1, "name": "alpha"}, {"id": 2, "name": "beta"}]), 200

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(host="0.0.0.0", port=int(os.getenv("PORT", "5000")))
```

Test:

```bash
curl -s http://localhost:5000/data
```

---

# 16) Python: einfache Flask-API – III

Erweitere um `/info` + ENV + uptime.

**`app/main.py`**

```python
import os
import time
from flask import Flask, jsonify

def create_app() -> Flask:
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
            app="flask-devops-pipeline",
            version=os.getenv("APP_VERSION", "dev"),
            environment=os.getenv("APP_ENV", "local"),
            uptime_seconds=int(time.time()) - started_at,
        ), 200

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(host="0.0.0.0", port=int(os.getenv("PORT", "5000")))
```

**`app/__init__.py`**

```python
from .main import create_app
__all__ = ["create_app"]
```

Tests:

**`tests/test_api.py`**

```python
from app.main import create_app

def test_health():
    app = create_app()
    c = app.test_client()
    r = c.get("/health")
    assert r.status_code == 200
    assert r.get_json()["status"] == "ok"

def test_data():
    app = create_app()
    c = app.test_client()
    r = c.get("/data")
    assert r.status_code == 200
    assert "items" in r.get_json()

def test_info():
    app = create_app()
    c = app.test_client()
    r = c.get("/info")
    assert r.status_code == 200
    body = r.get_json()
    assert "app" in body and "version" in body and "environment" in body
```

Run:

```bash
pytest -q
```

---

# 17) Docker: Erstelle ein Dockerfile (Multi-Stage, Best Practices)

**`Dockerfile`**

```dockerfile
# ---- Builder Stage ----
FROM python:3.11-slim AS builder

WORKDIR /build

# System deps (minimal)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
  && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN python -m venv /opt/venv \
 && /opt/venv/bin/pip install --upgrade pip \
 && /opt/venv/bin/pip install --no-cache-dir -r requirements.txt

# ---- Runtime Stage ----
FROM python:3.11-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=5000

# Non-root user
RUN useradd -m appuser
WORKDIR /app

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY app ./app

EXPOSE 5000

# Simple container healthcheck (HTTP)
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/health').read()" || exit 1

USER appuser

# Gunicorn for prod-like
CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "app.main:create_app()"]
```

**`.dockerignore`**

```txt
.venv
__pycache__
*.pyc
.pytest_cache
.ruff_cache
.git
.github
tests
```

---

# 18) Docker: Baue mein Anwendungs-Image

```bash
docker build -t flask-api:local .
docker images | grep flask-api
```




<img width="935" height="529" alt="2026-01-18 21_03_41-" src="https://github.com/user-attachments/assets/10ce2df2-571e-4742-97e6-62a5e7020335" />


<img width="941" height="477" alt="2026-01-18 21_30_58-Greenshot" src="https://github.com/user-attachments/assets/2a97bf6b-3d79-4cde-8fdd-e2fa16b20b26" />

---

# 19) Docker: Deploye meine Anwendung als Container – I (Run)

```bash
docker run --rm -p 5000:5000 \
  -e APP_ENV=container \
  -e APP_VERSION=local \
  --name flask-api flask-api:local
```

Test:

```bash
curl -s http://localhost:5000/health
curl -s http://localhost:5000/info
```


<img width="971" height="527" alt="2026-01-18 11_45_31-Get data - abk18937&#39;s Workspace" src="https://github.com/user-attachments/assets/4b7fbdf1-d328-4c76-99d4-9a3524ab36b9" />


<img width="535" height="320" alt="2026-01-18 11_45_05-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_src_app py (python-app) - Sub" src="https://github.com/user-attachments/assets/d9b85fcb-cc1d-4892-b87e-fbfdf3632062" />


<img width="631" height="390" alt="2026-01-18 21_37_39-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_Dockerfile (python-app) - Sub" src="https://github.com/user-attachments/assets/fc4279b7-20f8-4bf8-ae62-60dd5a100b04" />



---

# 20) Docker: Deploye meine Anwendung als Container – II (Logs/Stop/Debug)

Logs:

```bash
docker logs -f flask-api
```

Stop:

```bash
docker stop flask-api
```

Shell (optional):

```bash
docker run --rm -it flask-api:local sh
```

---

# 21) Docker: Registry + persönliches Authentifizierungs-Token

Beispiel: **Docker Hub**

* Docker Hub → Account Settings → Security → New Access Token
* Token in Passwort-Manager speichern

Login:

```bash
docker login -u <DOCKERHUB_USERNAME>
# Passwort = Token
```
<img width="821" height="445" alt="2026-01-18 21_54_16-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_Dockerfile (python-app) - Sub" src="https://github.com/user-attachments/assets/b9a42916-919a-4449-993b-c54480764342" />

---

# 22) Docker: Übertrage mein Image in eine Registry

Tag + Push:

```bash
export DOCKERHUB_USER="<dein_user>"
export IMAGE="docker.io/${DOCKERHUB_USER}/flask-devops-pipeline"
export TAG="v0.1.0"

docker tag flask-api:local ${IMAGE}:${TAG}
docker push ${IMAGE}:${TAG}
```
<img width="867" height="360" alt="2026-01-18 21_57_12-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_Dockerfile (python-app) - Sub" src="https://github.com/user-attachments/assets/20095d4a-0f25-406a-aa29-0c9e7bb0e83c" />




<img width="775" height="515" alt="2026-01-18 22_04_05-__wsl localhost_Ubuntu-24 04_home_koffi_python-app_Dockerfile (python-app) - Sub" src="https://github.com/user-attachments/assets/9b9496c8-2a45-4a0e-bfef-8b8a954f5617" />

---

# 23) Kubernetes: Deploye lokalen Cluster (kind empfohlen)

**`scripts/10_kind_create.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail

kind create cluster --name devops --image kindest/node:v1.29.2
kubectl cluster-info --context kind-devops
```

Run:

```bash
chmod +x scripts/*.sh
./scripts/10_kind_create.sh
```

---

# 24) Kubernetes: Greife auf meinen lokalen Cluster zu

```bash
kubectl get nodes
kubectl get ns
```

---

# 25) Kubernetes: Konfiguriere Ingress-Controller (NGINX)

Für kind:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller
```

---

# 26) Kubernetes: Deploye meine Anwendung (Manifests, Best Practices)

## 26.1 Namespace

**`k8s/namespace.yaml`**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: flask
```

## 26.2 Deployment (Resources + Probes)

**`k8s/deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-api
  namespace: flask
  labels:
    app: flask-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-api
  template:
    metadata:
      labels:
        app: flask-api
    spec:
      containers:
        - name: flask-api
          image: docker.io/<DEIN_USER>/flask-devops-pipeline:v0.1.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5000
          env:
            - name: APP_ENV
              value: "kubernetes"
            - name: APP_VERSION
              value: "v0.1.0"
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
```

> Wichtig: Ersetze `<DEIN_USER>` und Tag.

---

# 27) Kubernetes: Erstellung von  Services

**`k8s/service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-api
  namespace: flask
spec:
  selector:
    app: flask-api
  ports:
    - name: http
      port: 80
      targetPort: 5000
  type: ClusterIP
```

---

# 28) Kubernetes: Über Ingress zugänglich – I

**`k8s/ingress.yaml`**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-api
  namespace: flask
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: flask.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: flask-api
                port:
                  number: 80
```

---

# 29) lokales Hostname-Routing

Damit `flask.local` auf deinen lokalen Ingress zeigt:

* Linux/macOS: `/etc/hosts`
* Windows: `C:\Windows\System32\drivers\etc\hosts`

Eintrag:

```
127.0.0.1 flask.local
```

Bei kind/Ingress kann es sein, dass du Port Forward brauchst (siehe Schritt 30).

---

# 30) Kubernetes: Über Ingress zugänglich – II (Apply + Test)

**`scripts/12_k8s_apply.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail

kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml

kubectl -n flask rollout status deploy/flask-api
kubectl -n flask get all
kubectl -n flask get ingress
```

Run:

```bash
./scripts/12_k8s_apply.sh
```

Test (manchmal braucht kind Port Forward auf ingress):

```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: flask.local" http://localhost:8080/health
curl -H "Host: flask.local" http://localhost:8080/info
```

---

# 31) Kubernetes: Lösche Deployments, bevor ich Helm installierst

**`scripts/13_k8s_cleanup.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail
kubectl delete -f k8s/ingress.yaml --ignore-not-found
kubectl delete -f k8s/service.yaml --ignore-not-found
kubectl delete -f k8s/deployment.yaml --ignore-not-found
kubectl delete -f k8s/namespace.yaml --ignore-not-found
```

Run:

```bash
./scripts/13_k8s_cleanup.sh
```

---

# 32) Helm: Installiere Helm

Test:

```bash
helm version
```

---

# 33) Helm: Erstelle Helm-Chart für meine Anwendung

## 33.1 `Chart.yaml`

**`helm/flask-api/Chart.yaml`**

```yaml
apiVersion: v2
name: flask-api
description: Flask API for DevOps pipeline demo
type: application
version: 0.1.0
appVersion: "0.1.0"
```

## 33.2 `values.yaml`

**`helm/flask-api/values.yaml`**

```yaml
replicaCount: 2

image:
  repository: docker.io/<DEIN_USER>/flask-devops-pipeline
  tag: "v0.1.0"
  pullPolicy: IfNotPresent

env:
  APP_ENV: "kubernetes"
  APP_VERSION: "v0.1.0"

service:
  type: ClusterIP
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
  host: flask.local
  path: /
```

## 33.3 Helpers

**`helm/flask-api/templates/_helpers.tpl`**

```tpl
{{- define "flask-api.name" -}}
flask-api
{{- end -}}

{{- define "flask-api.fullname" -}}
{{ include "flask-api.name" . }}
{{- end -}}
```

## 33.4 Deployment Template

**`helm/flask-api/templates/deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "flask-api.fullname" . }}
  labels:
    app: {{ include "flask-api.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "flask-api.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "flask-api.name" . }}
    spec:
      containers:
        - name: flask-api
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
          livenessProbe:
            httpGet:
              path: /health
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 5
            periodSeconds: 5
```

## 33.5 Service Template

**`helm/flask-api/templates/service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "flask-api.fullname" . }}
spec:
  selector:
    app: {{ include "flask-api.name" . }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
  type: {{ .Values.service.type }}
```

## 33.6 Ingress Template

**`helm/flask-api/templates/ingress.yaml`**

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "flask-api.fullname" . }}
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
                name: {{ include "flask-api.fullname" . }}
                port:
                  number: {{ .Values.service.port }}
{{- end }}
```

---

# 34) Helm: Hinweis zu Values-Dateien

Du kannst Umgebungen trennen:

* `values-dev.yaml`
* `values-prod.yaml`

Beispiel `values-dev.yaml` (optional):

```yaml
replicaCount: 1
env:
  APP_ENV: "dev"
```

---

# 35) Helm: Deploye mein Helm-Chart auf Kubernetes

Namespace:

```bash
kubectl create ns flask --dry-run=client -o yaml | kubectl apply -f -
```

Install:

```bash
helm lint helm/flask-api
helm upgrade --install flask-api helm/flask-api -n flask \
  --set image.repository=docker.io/<DEIN_USER>/flask-devops-pipeline \
  --set image.tag=v0.1.0
```

Status:

```bash
kubectl -n flask get all
kubectl -n flask get ingress
```

Test (Ingress Port Forward):

```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: flask.local" http://localhost:8080/health
```

---

# 36) Helm: Lösche mein Release, bevor du bereitstellst (Cleanup)

```bash
helm uninstall flask-api -n flask
```

---

# 36) Helm: Lösche mein Release, bevor du ArgoCD deployst (nochmal als Check)

Wenn du noch Releases hast:

```bash
helm list -n flask
helm uninstall flask-api -n flask || true
```

---

# 37) Helm: Deploye ArgoCD

ArgoCD Namespace + Install:

```bash
kubectl create ns argocd --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd rollout status deploy/argocd-server
```

Port Forward UI:

```bash
kubectl -n argocd port-forward svc/argocd-server 8081:443
# Browser: https://localhost:8081
```

---

# 38) GitOps: Melde mich bei ArgoCD an

Default Admin Passwort aus Secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

CLI installieren (siehe Schritt 50), aber du kannst auch erstmal UI nutzen.

Login per CLI (nachdem CLI installiert):

```bash
argocd login localhost:8081 --username admin --password <PASS> --insecure
```

---

# 39) GitOps: Übertragung meine Änderungen zu GitHub

`.gitignore`:

```txt
.venv
__pycache__
*.pyc
.pytest_cache
.ruff_cache
.env
.DS_Store
```

Commit & Push:

```bash
git add .
git commit -m "feat: app + docker + k8s + helm + argocd scaffolding"
git remote add origin <DEIN_REPO_URL>
git push -u origin main
```

---

# 40) GitOps: Integration ArgoCD und GitHub (Repo verbinden)

ArgoCD Application zeigt auf dein Helm-Chart im Repo.

**`argocd/application.yaml`**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: flask-api
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<DEIN_USER>/<DEIN_REPO>.git
    targetRevision: main
    path: helm/flask-api
    helm:
      valueFiles:
        - values.yaml
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

---

# 41) GitOps: Deploye meine App mit ArgoCD

In UI: Application → Sync
Oder CLI:

```bash
argocd app sync flask-api
argocd app wait flask-api --health
```

Test:

```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: flask.local" http://localhost:8080/info
```

---

# 42) GitOps: Lerne, Apps aus ArgoCD zu löschen

```bash
argocd app delete flask-api --yes
# oder:
kubectl -n argocd delete application flask-api
```

---

# 43) Continuous Integration: Trigger für GitHub Actions

Wir machen:

* CI Workflow bei `push` und `pull_request` auf `main`
* Steps: lint/test, docker build, docker push (nur bei push main), helm lint/template/package

---

# 44) Umgang mit sensiblen Daten in GitHub Actions

Du brauchst in GitHub → Repo → Settings → Secrets and variables → Actions:

**Secrets**

* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN` (oder PAT)
* `GH_PAT` (optional, wenn du in anderes Repo pushen willst)
* `ARGOCD_SERVER` (z.B. `localhost:8081` bei self-hosted runner / oder externe Adresse)
* `ARGOCD_AUTH_TOKEN` (später, Token statt Passwort)

> Wichtig: Bei GitHub-hosted Runnern hast du keinen Zugriff auf dein lokales Kind/Minikube. CD braucht dafür meist **self-hosted runner** (siehe 48/49).

---

# 45) Dynamische Docker-Tags – I (Commit SHA verwenden)

Wir generieren Tags:

* `sha-<kurzsha>`
* optional `latest` auf main

---

# 46) Dynamische Docker-Tags – II (Pipeline Code)

**`.github/workflows/ci.yml`**

```yaml
name: CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

env:
  IMAGE_NAME: flask-devops-pipeline

jobs:
  test-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install deps
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt -r requirements-dev.txt

      - name: Ruff
        run: |
          ruff check .
          ruff format --check .

      - name: Pytest
        run: pytest -q

  docker-build-push:
    needs: test-lint
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Compute tags
        id: vars
        run: |
          SHORT_SHA="${GITHUB_SHA::7}"
          echo "short_sha=$SHORT_SHA" >> $GITHUB_OUTPUT
          echo "tag=sha-$SHORT_SHA" >> $GITHUB_OUTPUT

      - name: Login DockerHub (only on push main)
        if: github.event_name == 'push'
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build (always)
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.tag }} .

      - name: Push (only on push main)
        if: github.event_name == 'push'
        run: |
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.tag }}

      - name: Output tag
        run: echo "Pushed tag: ${{ steps.vars.outputs.tag }}"

  helm-lint-template:
    needs: test-lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Helm lint
        run: |
          helm lint helm/flask-api
          helm template flask-api helm/flask-api > /tmp/rendered.yaml
```

---

# 47) Kürze Commit-HASH für bessere Lesbarkeit

Das ist im CI bereits drin: `SHORT_SHA="${GITHUB_SHA::7}"`.

---

# 48) Continuous Deployment: Verteilte Builds mit self-hosted Runnern

**Warum?**
CD, die `kubectl`/`argocd` gegen deinen Cluster ausführt, braucht Zugriff auf Cluster/Netz. Das geht sauber über:

* self-hosted runner auf deiner Maschine (oder VM), die Zugriff auf kind/minikube hat

## 48.1 Runner installieren (Kurz)

* GitHub Repo → Settings → Actions → Runners → New self-hosted runner
* Befehle aus GitHub ausführen (die sind individuell)

---

# 49) Continuous Deployment: Definiere Labels für deine Runner

Beim Runner Setup kannst du Labels setzen, z.B.:

* `k8s`
* `argocd`
* `docker`

Dann in Workflow:

```yaml
runs-on: [self-hosted, linux, x64, k8s, argocd]
```

---

# 50) Ladung der ArgoCD-CLI herunter

Linux:

```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
argocd version
```

macOS (brew):

```bash
brew install argocd
```

---

# 51) Nutzung der ArgoCD-API für die Anmeldung (Token statt Passwort)

Token erzeugen (einmalig):

```bash
argocd account generate-token --account admin
```

Dann als Secret in GitHub:

* `ARGOCD_AUTH_TOKEN` = token

Login im Workflow:

```bash
argocd login "$ARGOCD_SERVER" --auth-token "$ARGOCD_AUTH_TOKEN" --insecure
```

---

# 52) Synchronisiere ArgoCD Anwendungen programmgesteuert

```bash
argocd app sync flask-api
argocd app wait flask-api --health
```

---

# 53) GitHub Actions: Baue einen CD-Workflow – I (Ziel)

CD macht:

1. nimmt Image-Tag aus CI (`sha-xxxxxxx`)
2. schreibt dieses Tag in `helm/flask-api/values.yaml` in `image.tag`
3. commit & push nach main
4. triggert `argocd app sync`

---

# 54) CD-Workflow – II (YAML programmgesteuert ändern)

Wir nutzen `yq` (mikefarah).

---

# 55) CD-Workflow – III (Commit & Push)

Wir pushen mit `GITHUB_TOKEN` (gleicher Repo) oder `GH_PAT` (wenn nötig).

---

# 56) CD-Workflow – IV (ArgoCD Sync)

Nur wenn Runner Zugriff auf ArgoCD Server hat.

---

# 57) Werte aus verschiedenen Jobs nutzen

Wir übergeben Docker Tag via `outputs`.

---

# 58) GitOps: Ändere YAML-Dateien programmgesteuert

Das macht der CD Workflow Schritt mit `yq`.

---

# 59) GitOps: Aktualisiere entfernte GitHub-Repositories

Push via Git im Workflow.

---

# 60) GitOps: Synchronisiere ArgoCD aus deiner CD-Pipeline

Am Ende `argocd app sync`.

---

## ✅ Komplettes CD Workflow File

**`.github/workflows/cd.yml`**

```yaml
name: CD

on:
  workflow_run:
    workflows: ["CI"]
    types:
      - completed

env:
  IMAGE_NAME: flask-devops-pipeline

jobs:
  prepare:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    outputs:
      tag: ${{ steps.vars.outputs.tag }}
    steps:
      - name: Compute tag from SHA
        id: vars
        run: |
          SHA="${{ github.event.workflow_run.head_sha }}"
          SHORT="${SHA::7}"
          echo "tag=sha-$SHORT" >> $GITHUB_OUTPUT

  update-gitops-and-sync:
    needs: prepare
    runs-on: [self-hosted, linux, x64, k8s, argocd]
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main

      - name: Install yq
        run: |
          sudo curl -sSL -o /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
          sudo chmod +x /usr/local/bin/yq
          yq --version

      - name: Update Helm values image tag
        run: |
          yq -i '.image.tag = "${{ needs.prepare.outputs.tag }}"' helm/flask-api/values.yaml
          yq -i '.env.APP_VERSION = "${{ needs.prepare.outputs.tag }}"' helm/flask-api/values.yaml
          git diff

      - name: Commit & push
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@users.noreply.github.com"
          git add helm/flask-api/values.yaml
          git commit -m "chore: bump image tag to ${{ needs.prepare.outputs.tag }}" || echo "No changes"
          git push origin main

      - name: ArgoCD login (token)
        env:
          ARGOCD_SERVER: ${{ secrets.ARGOCD_SERVER }}
          ARGOCD_AUTH_TOKEN: ${{ secrets.ARGOCD_AUTH_TOKEN }}
        run: |
          argocd login "$ARGOCD_SERVER" --auth-token "$ARGOCD_AUTH_TOKEN" --insecure

      - name: Sync app
        run: |
          argocd app sync flask-api
          argocd app wait flask-api --health
```

> Wichtig:

* `ARGOCD_SERVER` muss von deinem self-hosted Runner erreichbar sein.
  Wenn du lokal per Port-Forward arbeitest, muss der Runner auf derselben Maschine laufen.

---

# Scripts (damit ich nicht alles tippen musste)

**`scripts/02_docker_build_run.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail
docker build -t flask-api:local .
docker run --rm -p 5000:5000 -e APP_ENV=container -e APP_VERSION=local flask-api:local
```

**`scripts/03_push_registry.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail
DOCKERHUB_USER="${1:?Usage: ./scripts/03_push_registry.sh <dockerhub_user> <tag>}"
TAG="${2:-v0.1.0}"
IMAGE="docker.io/${DOCKERHUB_USER}/flask-devops-pipeline"

docker tag flask-api:local ${IMAGE}:${TAG}
docker push ${IMAGE}:${TAG}
echo "Pushed: ${IMAGE}:${TAG}"
```

**`scripts/30_argocd_install.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail
kubectl create ns argocd --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd rollout status deploy/argocd-server
```

---

# README.md (voller Ablauf in einem Stück)

**`README.md`**

````md
# flask-devops-pipeline

## Local (Python)
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
python -m app.main
````

## Docker

```bash
docker build -t flask-api:local .
docker run --rm -p 5000:5000 -e APP_ENV=container -e APP_VERSION=local flask-api:local
```

## Kubernetes (kind)

```bash
kind create cluster --name devops
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller

kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

Ingress test:

```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -H "Host: flask.local" http://localhost:8080/health
```

## Helm

```bash
helm upgrade --install flask-api helm/flask-api -n flask \
  --create-namespace \
  --set image.repository=docker.io/<user>/flask-devops-pipeline \
  --set image.tag=v0.1.0
```

## ArgoCD

```bash
kubectl create ns argocd --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd port-forward svc/argocd-server 8081:443
```

Apply App:

```bash
kubectl apply -f argocd/application.yaml
```

## GitHub Actions

* CI: Tests, Docker Build/Push (push main), Helm lint/template
* CD: bump helm values image.tag + argocd sync via CLI (self-hosted runner)

```

---

## Schluss:

1) Ersetzung :
- `<DEIN_USER>` / `<DEIN_REPO>` im ArgoCD Application und Helm values
- `image:` im k8s deployment oder nutze Helm statt raw k8s

2) Dann:
- Push Repo
- Docker Hub token als Secret
- Runner (self-hosted) für CD einrichten
- ArgoCD Server + Token als Secrets setzen

3) Ergebnis:
- Push auf main → CI baut Image und pushed `sha-xxxxxxx`
- CD setzt automatisch `helm/flask-api/values.yaml:image.tag`
- ArgoCD synced automatisch oder per `argocd app sync`

---

** vielen dank **
