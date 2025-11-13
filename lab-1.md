# Lab 1: Deploy and Scale a Sample NGINX App on OpenShift

## Table of Contents

- [Lab 1: Deploy and Scale a Sample NGINX App on OpenShift](#lab-1-deploy-and-scale-a-sample-nginx-app-on-openshift)
  - [Table of Contents](#table-of-contents)
  - [Step 1: Login to OpenShift](#step-1-login-to-openshift)
    - [CLI](#cli)
    - [GUI](#gui)
  - [Step 2: Create a Project](#step-2-create-a-project)
    - [CLI](#cli-1)
    - [GUI](#gui-1)
  - [Step 3: Deploy NGINX Image](#step-3-deploy-nginx-image)
    - [CLI](#cli-2)
    - [GUI](#gui-2)
  - [Step 4: Expose the Application](#step-4-expose-the-application)
    - [CLI](#cli-3)
    - [GUI](#gui-3)
  - [Step 5: Scale the Application](#step-5-scale-the-application)
    - [CLI](#cli-4)
    - [GUI](#gui-4)
  - [Step 6: Quality Assurance (QA)](#step-6-quality-assurance-qa)
    - [CLI](#cli-5)
    - [GUI](#gui-5)
  - [References](#references)

---

## Step 1: Login to OpenShift

### CLI

1. Login to your cluster using the `oc` command:

```bash
oc login https://api.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com:6443 -u admin -p admin
```

2. Verify your login:

```bash
oc whoami
```

### GUI

1. Open your browser and navigate to the OpenShift Web Console:

   ```
   https://console-openshift-console.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com
   ```
2. Login using the credentials:

   * **admin:** `admin`

---

## Step 2: Create a Project

### CLI

```bash
oc new-project customer-demo
oc get projects
```

### GUI

1. Switch to **Developer View → Projects → Create Project**.
2. Enter the project name (`customer-demo`) and click **Create**.

---

## Step 3: Deploy NGINX Image

### CLI

```bash
oc new-app nginxinc/nginx-unprivileged --name=webapp
```

### GUI

1. Navigate to **Developer → +Add → Container Image**.
2. Enter the following details:

   * **Image Name:** `nginxinc/nginx-unprivileged`
   * **Application Name:** `webapp`
3. Accept defaults and click **Create**.

---

## Step 4: Expose the Application

### CLI

1. Create an edge route:

```bash
oc create route edge webapp --service=webapp
```

2. Verify the route:

```bash
oc get route
```

3. Open the route URL in a browser to access the NGINX welcome page.

### GUI

1. Go to **Networking → Routes** in your project.
2. Click **Create Route**.
3. Fill in the form:

   * **Name:** `webapp`
   * **Service:** `webapp`
   * **Target Port:** `8080`
   * **Secure Route (optional):** Check to enable HTTPS and choose **edge**, **reencrypt**, or **passthrough**.
4. Click **Create**.
5. Open the **Location URL** to verify the NGINX welcome page.

---

## Step 5: Scale the Application

### CLI

1. Scale your deployment to 3 replicas:

```bash
oc scale deployment webapp --replicas=3
```

2. Verify the pods:

```bash
oc get pods
```

### GUI

1. Go to **Project → Workloads → Deployments → webapp**.
2. On the **Overview tab**, locate the **Replicas** section.
3. Click the small arrows (▲ / ▼) to increase or decrease replicas.
4. OpenShift automatically adjusts the number of pods.

> **Tip:** For automatic scaling based on CPU usage, you can use a Horizontal Pod Autoscaler:

```bash
oc autoscale deployment webapp --min=1 --max=5 --cpu-percent=50
```

---

## Step 6: Quality Assurance (QA)

### CLI

```bash
oc get pods
oc get svc
oc get route
```

* Verify that all pods are running.
* Confirm the service and route are correctly exposed.

### GUI

1. Navigate to **Workloads → Pods** to check pod status.
2. Navigate to **Networking → Services** to verify service exposure.
3. Open the route URL in a browser to confirm the NGINX welcome page is displayed.

---

## References

1. OpenShift Web Console: [https://console-openshift-console.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com](https://console-openshift-console.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com)
2. OpenShift CLI Download: [https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/](https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/)
3. OpenShift NGINX Example: [https://github.com/sclorg/nginx-container](https://github.com/sclorg/nginx-container)
4. OpenShift Routes & Networking Guide: [https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)
5. OpenShift Horizontal Pod Autoscaler: [https://docs.openshift.com/container-platform/latest/scaling/using-autoscaling.html](https://docs.openshift.com/container-platform/latest/scaling/using-autoscaling.html)

