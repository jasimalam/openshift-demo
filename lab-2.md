# Modernize App Deployment with CI/CD — OpenShift Lab (2 hours)

**Goals (by end of lab)**

* Deploy a sample application using Source-to-Image (S2I).
* Configure automated builds (BuildConfig triggers) so commits kick off builds.
* Create and inspect a CI/CD pipeline (OpenShift Pipelines / Tekton or BuildConfig + DeploymentConfig steps).
* Observe pipeline runs and application rollout (GUI and CLI).

---

## Prerequisites

* OpenShift cluster and credentials (cluster URL, `oc` configured).
* `oc` CLI installed and logged in as a user with permissions to create projects, builds, pipelines, and deployments.
* A Git repository containing a simple app (e.g., Node.js, Python, or Java) with a `Dockerfile` or compatible for S2I. Replace `GIT_URL` in steps with your repo URL.
* Optional: OpenShift Pipelines (Tekton) installed in the cluster (if following the Tekton pipeline sub-section). If not installed, the lab uses BuildConfig + DeploymentConfig pipeline.

---

## Lab duration & schedule (2 hours)

* 0–10 min: Introduction and environment setup (create project / check access)
* 10–40 min: Part A — Deploy application using S2I (GUI + CLI)
* 40–80 min: Part B — Explore automated build pipelines (BuildConfig triggers; GUI + CLI)
* 80–110 min: Part C — Create a CI/CD pipeline (Tekton example + lighter BuildConfig flow)
* 110–120 min: Review pipeline run, check application rollout, cleanup, Q&A

---

# Detailed steps

> **NOTE:** Each major step includes both **GUI** instructions (OpenShift Console) and **CLI** instructions (`oc`). Follow either or both.

## 0. Environment setup (0–10 min)

### GUI

1. Log in to the OpenShift web console using your credentials.
2. Create a new project/namespace: **Administrator → Projects → Create Project**. Name it `ci-cd-lab`.
3. Open the project dashboard and note the navigation panels (Builds, Pipelines, Workloads, Networking).

### CLI

```bash
# create project
oc new-project ci-cd-lab --display-name="CI/CD Lab"
# confirm
oc project
oc whoami
```

Expected: project `ci-cd-lab` exists and you are a member.

---

## Part A — Deploy application using S2I (10–40 min)

We will use S2I to build the image from source and create an app deployment.

### GUI — S2I quick-start

1. In the project, click **+Add** → **From Git**.
2. Enter **Git Repo URL** (replace `GIT_URL`).
3. For **Builder Image**, choose a supported S2I builder (e.g., *Node.js*, *Python*, *OpenJDK*, or others available in the cluster). The Console labels these as *Builder image*.
4. Set the **Application Name** (e.g., `myapp`). Verify resources and route creation options. Click **Create**.
5. Monitor the Build under **Builds → Builds**. When complete, check **Workloads → Deployments** to see the running pod(s). Click the route URL to open the app.

Tips: If the builder image isn't visible, you may need to enable appropriate image streams or use `oc new-app` in CLI.

### CLI — S2I using `oc new-app`

```bash
# variables
PROJECT=ci-cd-lab
APPNAME=myapp
GIT_URL=https://github.com/your-org/your-app.git
BUILDER=nodejs:14  # adjust to a builder available in your cluster

oc project $PROJECT
# Create an S2I build & app from source
oc new-app ${BUILDER}~${GIT_URL} --name=${APPNAME}

# Watch the build
oc logs -f bc/${APPNAME}
# Check build status
oc get builds
# After build completes, check deployment
oc get pods -l app=${APPNAME}
# Expose service if not auto-created
oc expose svc/${APPNAME}
# Get route
oc get route ${APPNAME}
```

Expected: A BuildConfig and Build run, followed by an ImageStream and a Deployment (or DeploymentConfig). Application accessible via a route.

Troubleshooting:

* Build fails due to missing S2I builder: use an available builder image stream or add image stream definitions.
* If the repo requires a specific context directory or branch, pass `--context-dir` or `--strategy` options.

---

## Part A.1 — Manual Build (GUI & CLI)

### GUI — Manual Build

1. Go to **Builds → Build Configs**.
2. Click the BuildConfig for your app (example: `myapp`).
3. Click **Actions → Start Build**.
4. Observe the new Build appearing under **Builds → Builds**.
5. Open the build log to watch the S2I process.

### CLI — Manual Build

```bash
ochali_project=ci-cd-lab
APPNAME=myapp
oc project $APPNAME
# Start build manually
ochali start-build bc/$APPNAME
# Tail the logs
ochali logs -f build/$(oc get build -l buildconfig=$APPNAME -o name | tail -1)
```

---

## Part B — Explore automated build pipelines (40–80 min)

Two quick ways to achieve automation:

1. **BuildConfig triggers** (Git/webhook-based automatic builds) — default and simplest.
2. **OpenShift Pipelines (Tekton)** — full pipeline orchestration (recommended for modern CI/CD).

### B.1 BuildConfig triggers — GUI

1. Open **Builds → Build Configs** and click your `myapp` BuildConfig.
2. Under **Configuration** you will find **Triggers**. Ensure GitHub (or GitLab) webhook triggers are enabled (Push events).
3. Copy the webhook URL and add it to your Git repository's webhook settings (e.g., GitHub → repo → Settings → Webhooks → Add webhook). Use `Push events`.
4. Make a small change in your repo (e.g., update README) and push — check the Build list to see a new build triggered automatically.

### B.1 BuildConfig triggers — CLI

```bash
# view buildconfig triggers
oc get bc ${APPNAME} -o yaml
# Find webhook URLs (github, generic, gitlab). To print only webhooks:
oc describe bc ${APPNAME}

# Example: trigger a build manually (simulate a webhook)
oc start-build bc/${APPNAME}
```

Expected: New builds start after each push/webhook.

### B.2 OpenShift Pipelines (Tekton) — GUI

> Use this path only if Tekton Pipelines (OpenShift Pipelines) is installed.

1. In the Console, go to **Pipelines → Pipelines** and click **Create Pipeline** (or import a Pipeline YAML).
2. Create a Pipeline that contains tasks: `fetch-source` (git-clone), `build` (s2i build or buildah/podman inside task), and `deploy` (apply a Deployment/DeploymentConfig or image rollout).
3. Create a PipelineRun linked to the pipeline; inspect logs as the pipeline executes.
4. Configure a TriggerBinding/TriggerTemplate and EventListener to accept webhooks from Git. Then add that webhook to your Git provider.

### B.2 OpenShift Pipelines (Tekton) — CLI (example minimal)

Create resources: `Pipeline`, `PipelineRun`, optionally `TriggerBinding` and `EventListener`. Minimal example using BuildConfig trigger integration (pseudo-steps):

```bash
# 1) Create a Task or use standard tasks (git-clone, buildah, kubernetes-actions)
# 2) Create a Pipeline referencing the Task(s)
oc apply -f pipeline.yaml
# 3) Start a PipelineRun
oc apply -f pipelinerun.yaml
# Watch logs
tkn pipelinerun logs <pipelinerun-name> -f
```

Notes:

* `tkn` CLI (Tekton CLI) is helpful but not required — you can use `oc` to view PipelineRun pods and logs.
* If Tekton is not installed, fall back to BuildConfig triggers + `oc start-build`.

Troubleshooting:

* PipelineTask pods may fail due to missing service account permissions — ensure `pipeline` service account has necessary SCC and image-pull rights.

---

## Part C — Review pipeline run and application rollout (80–110 min)

This section shows how to inspect pipeline output, track artifact/image promotion, and check rollout status.

### C.1 Review pipeline run — GUI

1. Open **Pipelines → PipelineRuns** (or **Builds → Builds** if using BuildConfig flow).
2. Click the running/finished PipelineRun to view logs for each Task.
3. Note the produced image tag (ImageStream or registry image) and the created Deployment/DeploymentConfig.

### C.1 Review — CLI

```bash
# If using BuildConfig
oc get builds -w
oc logs build/<build-name> -f

# If using Tekton
# list runs
oc get pipelineruns -n ci-cd-lab
# view logs with tkn (if installed)
tkn pipelinerun logs <pipelinerun-name> -f
# or inspect pods created for the pipelinerun
oc get pods -l tekton.dev/pipelineRun=<pipelinerun-name>
oc logs -f <task-pod-name>
```

### C.2 Inspect rollout and rollout history — GUI

1. Open **Workloads → Deployments** and click your deployment.
2. The Console shows current replica status, events, and rollout history.
3. Use the **Pods** link to view pod logs.

### C.2 Inspect rollout and rollout history — CLI

```bash
# Check rollout status
oc rollout status deployment/${APPNAME}
# Check rollout history
oc rollout history deployment/${APPNAME}
# Check pods & logs
oc get pods -l app=${APPNAME}
oc logs -f pod/<pod-name>
# If a deployment fails, describe it
oc describe deployment/${APPNAME}
```

Expected: Successful rollout with pods ready. Rollout history shows revisions when images changed.

---

## Optional: Promote image across environments (brief)

* Tag ImageStream to a new namespace, or push to internal registry then update Deployment image to new tag.

CLI example:

```bash
# tag the imagestream to 'stage'
oc tag ci-cd-lab/${APPNAME}:latest ci-cd-stage/${APPNAME}:promoted
# or update deployment image
oc set image deployment/${APPNAME} ${APPNAME}=image-registry.openshift-image-registry.svc:5000/ci-cd-lab/${APPNAME}:latest
```

---

## Final checks, cleanup and submission (110–120 min)

1. Confirm application reachable via route. Note the route URL.
2. Collect evidence: build logs, PipelineRun name, deployment revision.
3. Cleanup (optional):

```bash
oc delete project ci-cd-lab
```

---

## Lab assessment / checklist

* [ ] Project `ci-cd-lab` created
* [ ] Application built with S2I and running
* [ ] Webhook or `oc start-build` triggers a new build
* [ ] Pipeline (Tekton or simplified) created and executed
* [ ] Verified rollout and accessed application route

---

## Troubleshooting notes & tips

* If builds are timing out, verify network access to your Git provider and the builder images.
* If tasks fail in Tekton, check ServiceAccount permissions (rolebindings) and SCC.
* Use `oc describe` to get events and reasons for failures.