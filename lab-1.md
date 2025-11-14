# Lab 1: Deploy and Scale a Sample NGINX App on OpenShift

A hands-on guide to deploying, exposing, and scaling an NGINX application using both CLI and GUI methods.

---

## Table of Contents

- [Lab 1: Deploy and Scale a Sample NGINX App on OpenShift](#lab-1-deploy-and-scale-a-sample-nginx-app-on-openshift)
  - [Table of Contents](#table-of-contents)
  - [Step 1: Login to OpenShift](#step-1-login-to-openshift)
    - [GUI Method](#gui-method)
    - [CLI Method](#cli-method)
  - [Step 2: Create a Project](#step-2-create-a-project)
    - [GUI Method](#gui-method-1)
    - [CLI Method](#cli-method-1)
  - [Step 3: Deploy NGINX Image](#step-3-deploy-nginx-image)
    - [GUI Method](#gui-method-2)
    - [CLI Method](#cli-method-2)
  - [Step 4: Expose the Application](#step-4-expose-the-application)
    - [GUI Method](#gui-method-3)
    - [CLI Method](#cli-method-3)
  - [Step 5: Scale the Application](#step-5-scale-the-application)
    - [GUI Method](#gui-method-4)
    - [CLI Method](#cli-method-4)
    - [Bonus: Auto-scaling](#bonus-auto-scaling)
  - [Step 6: Verification](#step-6-verification)
    - [GUI Method](#gui-method-5)
    - [CLI Method](#cli-method-5)
  - [References](#references)

---

## Step 1: Login to OpenShift

Download OC Client: [https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/](https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/)

### GUI Method

1. Navigate to the OpenShift Web Console:
   ```
    OpenShift Console: https://console-openshift-console.apps.cluster-5d8xx.5d8xx.sandbox2826.opentlc.com
   ```

2. Login with credentials:
   - Username: `admin`
   - Password: `admin`

### CLI Method

Login to your cluster:

```bash
oc login https://api.cluster-5d8xx.5d8xx.sandbox2826.opentlc.com:6443 -u admin -p admin
```

Verify your login:

```bash
oc whoami
```

---

## Step 2: Create a Project

### GUI Method

1. Switch to Developer View → Projects → Create Project
2. Enter project name: `customer-demo`
3. Click Create

### CLI Method

```bash
oc new-project customer-demo
oc get projects
```

---

## Step 3: Deploy NGINX Image

### GUI Method

1. Navigate to Developer → +Add → Container Image
2. Configure the deployment:
   - Image Name: `nginxinc/nginx-unprivileged`
   - Application Name: `webapp`
3. Accept defaults and click Create

### CLI Method

```bash
oc new-app nginxinc/nginx-unprivileged --name=webapp
```

---

## Step 4: Expose the Application

### GUI Method

1. Go to Networking → Routes in your project
2. Click Create Route
3. Configure the route:
   - Name: `webapp`
   - Service: `webapp`
   - Target Port: `8080`
   - Secure Route (optional): Enable HTTPS and choose edge, reencrypt, or passthrough
4. Click Create
5. Open the Location URL to verify the NGINX welcome page

### CLI Method

Create an edge route:

```bash
oc create route edge webapp --service=webapp
```

Verify the route:

```bash
oc get route
```

Open the route URL in a browser to access the NGINX welcome page.

---

## Step 5: Scale the Application

### GUI Method

1. Navigate to Project → Workloads → Deployments → webapp
2. On the Overview tab, locate the Replicas section
3. Use the arrows (▲ / ▼) to adjust replica count
4. OpenShift automatically adjusts the number of pods

### CLI Method

Scale to 3 replicas:

```bash
oc scale deployment webapp --replicas=3
```

Verify the pods:

```bash
oc get pods
```

### Bonus: Auto-scaling

For automatic scaling based on CPU usage, create a Horizontal Pod Autoscaler:

```bash
oc autoscale deployment webapp --min=1 --max=5 --cpu-percent=50
```

---

## Step 6: Verification
### GUI Method

1. Check Pods: Navigate to Workloads → Pods to verify pod status
2. Check Service: Navigate to Networking → Services to confirm service exposure
3. Test Route: Open the route URL in a browser to confirm the NGINX welcome page displays

### CLI Method

Run verification commands:

```bash
oc get pods
oc get svc
oc get route
```

Verify:
- All pods are running
- Service is correctly exposed
- Route is accessible

---

## References

1. Getting Started with the OpenShift CLI  
   [https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)

2. Working with Projects  
   [https://docs.openshift.com/container-platform/latest/applications/projects/working-with-projects.html](https://docs.openshift.com/container-platform/latest/applications/projects/working-with-projects.html)

3. Deploying Applications  
   [https://docs.openshift.com/container-platform/latest/applications/creating_applications/creating-applications-using-cli.html](https://docs.openshift.com/container-platform/latest/applications/creating_applications/creating-applications-using-cli.html)

4. Route Configuration  
   [https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)

5. Horizontal Pod Autoscaling  
   [https://docs.openshift.com/container-platform/latest/nodes/pods/nodes-pods-autoscaling.html](https://docs.openshift.com/container-platform/latest/nodes/pods/nodes-pods-autoscaling.html)

6. Scaling Applications  
   [https://docs.openshift.com/container-platform/latest/applications/deployments/deployment-strategies.html](https://docs.openshift.com/container-platform/latest/applications/deployments/deployment-strategies.html)

---

Lab Complete! You've successfully deployed, exposed, and scaled an NGINX application on OpenShift.