# OpenShift CI/CD Lab

## Prerequisites

- OpenShift cluster access with credentials
- `oc` CLI installed
- Git repository: `https://github.com/jasimalam/fruit-basket-app`

---

## Part 1: Environment Setup

### Step 1.1: Log in to OpenShift

Open browser, go to cluster URL, enter credentials.

### Step 1.2: Create Project

**Web Console:**
- **Administrator** → **Home** → **Projects** → **Create Project**
- Name: `ci-cd-lab`
- Click **Create**

**CLI:**

```bash
oc login <cluster-url>
oc new-project ci-cd-lab
oc project
```

---

## Part 2: Deploy Application

### Step 2.1: Deploy from Source

**Web Console:**
- Switch to **Developer** perspective
- **+Add** → **Import from Git**
- Git Repo URL: `https://github.com/jasimalam/fruit-basket-app`
- Builder Image: **Python 3.11 (UBI 8)**
- Application: `myapp-group`
- Name: `myapp`
- Resources: **Deployment**
- Check **Create a route**
- Click **Create**

**CLI:**

```bash
oc new-app python:3.11-ubi8~https://github.com/jasimalam/fruit-basket-app --name=myapp
oc logs -f bc/myapp
oc expose svc/myapp
oc get route myapp
```

### Step 2.2: Access Application

**Web Console:**  
Topology view → click route icon on `myapp` node

**CLI:**
```bash
oc get route myapp -o jsonpath='{.spec.host}'
```

Open the URL in browser.

---

## Part 3: Manual Builds

### Trigger a Build

**Web Console:**  
**Builds** → **myapp** → **Actions** → **Start build**

**CLI:**

```bash
oc start-build myapp
oc get builds
oc logs -f bc/myapp
```

---

## Part 4: Automated Builds

### Get Webhook URL

**Web Console:**  
**Builds** → **myapp** → Webhooks section → Copy GitHub webhook URL

**CLI:**
```bash
oc describe bc/myapp | grep -A 1 "Webhook GitHub"
```

Copy the URL that appears (it's very long and starts with `https://`).

### Step 4.2: Understand Webhook Triggers

The webhook URL allows GitHub (or another Git service) to notify OpenShift when you push code changes. When OpenShift receives this notification, it automatically starts a new build.

**Note**: Setting up the actual webhook requires write access to the Git repository. If you're using the sample repository, your instructor may demonstrate this, or you can skip to testing with manual builds.

To set up a webhook (if you have your own repository):
1. Go to your GitHub repository
2. Click **Settings** → **Webhooks** → **Add webhook**
3. Paste the webhook URL from OpenShift
4. Content type: **application/json**
5. Select **Just the push event**
6. Click **Add webhook**

### Step 4.3: View Build Triggers Configuration

```bash
oc get bc/myapp -o yaml | grep -A 10 triggers
```

You should see configurations for different types of triggers (GitHub, Generic, ImageChange, ConfigChange).

---

## Part 5: Monitor Application Rollout (20 minutes)

### Check Status

**Web Console:**  
**Administrator** → **Workloads** → **Deployments** → **myapp**

**CLI:**
```bash
oc get deployment myapp
oc rollout status deployment/myapp
oc get pods -l app=myapp
```

### View Logs

**Web Console:**  
**Workloads** → **Pods** → click pod → **Logs** tab

**CLI:**
```bash
oc get pods -l app=myapp
oc logs <pod-name>
oc logs -f <pod-name>
```

### Deployment History

```bash
oc rollout history deployment/myapp
```

---

## Part 6: CI/CD Pipeline

### Check Pipeline Installation
**Web Console:**  
Check left menu for **Pipelines**

**CLI:**
```bash
oc get pods -n openshift-pipelines
```

### Create Pipeline

Create `pipeline.yaml`:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: myapp-pipeline
  namespace: ci-cd-lab
spec:
  params:
    - name: git-url
      type: string
      description: Git repository URL
    - name: app-name
      type: string
      description: Application name
  tasks:
    - name: fetch-repository
      taskRef:
        name: git-clone
        kind: ClusterTask
      params:
        - name: url
          value: $(params.git-url)
        - name: deleteExisting
          value: "true"
      workspaces:
        - name: output
          workspace: shared-workspace
    
    - name: build-image
      taskRef:
        name: buildah
        kind: ClusterTask
      params:
        - name: IMAGE
          value: image-registry.openshift-image-registry.svc:5000/ci-cd-lab/$(params.app-name):latest
      runAfter:
        - fetch-repository
      workspaces:
        - name: source
          workspace: shared-workspace
    
    - name: deploy
      taskRef:
        name: openshift-client
        kind: ClusterTask
      params:
        - name: SCRIPT
          value: |
            oc rollout status deployment/$(params.app-name)
      runAfter:
        - build-image
  
  workspaces:
    - name: shared-workspace
```

Apply:
```bash
oc apply -f pipeline.yaml
oc get pipeline
```

### Run Pipeline

Create `pipelinerun.yaml`:

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: myapp-pipeline-run-1
  namespace: ci-cd-lab
spec:
  pipelineRef:
    name: myapp-pipeline
  params:
    - name: git-url
      value: https://github.com/jasimalam/fruit-basket-app
    - name: app-name
      value: myapp
  workspaces:
    - name: shared-workspace
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
```

Execute:
```bash
oc apply -f pipelinerun.yaml
```

### Monitor Execution

**Web Console:**  
**Pipelines** → **PipelineRuns** → **myapp-pipeline-run-1**

**CLI:**
```bash
oc get pipelinerun myapp-pipeline-run-1
oc logs -f $(oc get pods -l tekton.dev/pipelineRun=myapp-pipeline-run-1 -o name | head -1)
```

---

## Verification

Check all resources:

```bash
oc get all -l app=myapp
curl -I $(oc get route myapp -o jsonpath='{.spec.host}')
```

Checklist:
- [ ] Project created
- [ ] App built and running
- [ ] Manual builds work
- [ ] Webhook configured
- [ ] Logs accessible
- [ ] Pipeline executed
- [ ] Route accessible

---

## Troubleshooting

**Build fails:**
```bash
oc get imagestreams -n openshift
```

**Pod crashes:**
```bash
oc logs <pod-name>
```

**Route not working:**
```bash
oc get route
oc get svc
```

**Pipeline permission issues:**
```bash
oc get sa pipeline
oc describe sa pipeline
```

---

## Cleanup

```bash
oc delete project ci-cd-lab
```
