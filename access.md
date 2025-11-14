# OpenShift Cluster Credentials

## OpenShift Access

### Console & API
- **Console URL**: https://console-openshift-console.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com
- **API URL**: https://api.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com:6443
- **OC Client Download**: http://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable-4.15/openshift-client-linux-amd64-rhel9.tar.gz

### OpenShift Users
- **Cluster Admin**
  - Username: `admin`
  - Password: `MTE4MDY5`
- **Regular Users**
  - Usernames: `user1` to `user2`
  - Password: `openshift`

## Gitea

### Access
- **URL**: https://gitea.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com

### Credentials
- **Admin Username**: `opentlc-mgr`
- **Admin Password**: `MTE4MDY5`
- **User Accounts**: `user1` to `user2`
- **User Password**: `openshift`

### Migrated Repository
- https://github.com/rh-mad-workshop/modern-app-dev.git

## OpenShift GitOps (ArgoCD)

- **URL**: https://openshift-gitops-server-openshift-gitops.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com
- **Note**: Authentication via RHSSO is enabled

## RHSSO (Red Hat Single Sign-On / Keycloak)

### Access
- **Admin Console**: https://keycloak-rhsso.apps.cluster-hmp6f.hmp6f.sandbox2076.opentlc.com

### Credentials
- **Admin Username**: `admin`
- **Admin Password**: `YHdt6vSvUwRlrw==`

## Bastion Host SSH Access

### Connection
```bash
ssh lab-user@bastion.hmp6f.sandbox2076.opentlc.com
```

### Credentials
- **Username**: `lab-user`
- **Password**: `ONAG24qSLfuO`