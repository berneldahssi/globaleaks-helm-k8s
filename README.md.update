# Globaleaks Kubernetes Deployment with Traefik Ingress Controller

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30.4-blue.svg)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-v3.16.1-blue)](https://helm.sh/)
[![Traefik](https://img.shields.io/badge/Traefik-2.10.7-yellow.svg)](https://doc.traefik.io/traefik/)
[![Globaleaks](https://img.shields.io/badge/Globaleaks-5.0.10-green.svg)](https://hub.docker.com/r/globaleaks/globaleaks)
![Rancher v1.16.0](https://img.shields.io/badge/Rancher-v1.16.0-blue.svg)
![k3s](https://img.shields.io/badge/k3s-v1.16.0-orange.svg)


Welcome to the Globaleaks Kubernetes deployment using **Helm** and **Traefik** as the ingress controller! This repository contains the necessary files and instructions to deploy Globaleaks on Kubernetes using a Helm chart.

### Objective:
The goal of this project is to deploy **Globaleaks** on Kubernetes using **Traefik** as the ingress controller, packaged as a Helm chart for easy deployment and management.

---

## 🛠 Task Overview:
1. **Deploy Globaleaks** using Kubernetes and Traefik as the ingress controller.
2. Package the deployment into a **Helm chart** for easy distribution.
3. Implement a **daily backup strategy** using Kubernetes CronJobs and Velero for volume snapshots.

### 💡 Prerequisites:
- **Rancher Desktop** with **k3s** as the Kubernetes runtime.
- **Traefik** and **Helm** already installed and configured within the Kubernetes cluster.

### 🚀 Deployment Instructions:

#### 1. Clone the Repository:
```bash
git clone https://github.com/mirabelle4/globaleaks-helm-k8s.git
cd globaleaks-helm-k8s
```

#### 2. Decompress the Helm Package:
```bash
tar -xzvf globaleaks-helm-chart.tgz
```

#### 3. Install the Helm Chart:
```bash
helm install globaleaks ./globaleaks-helm-chart
```

#### 4. Port-Forwarding for Traefik Dashboard:
To access the Traefik dashboard, run the following command:

```bash
kubectl port-forward -n kube-system $(kubectl -n kube-system get pods --selector "app.kubernetes.io/name=traefik" -o jsonpath="{.items[0].metadata.name}") 9000:9000
```

Now, you can access the Traefik dashboard at:

```bash
http://localhost:9000/dashboard/
```

#### 5. Accessing the Globaleaks Application:
To access Globaleaks through the browser, use:

```bash
http://localhost:443
```

If you're deploying this in a production environment or using a custom domain, you'll need to replace `localhost` in the IngressRoute definition file with your actual domain name.

---

### 📦 Helm Chart Details:
The provided Helm chart includes the following:
- Kubernetes Deployment for Globaleaks.
- Traefik IngressRoute configuration.
- Persistent Volume setup to handle Globaleaks data.
- Services and Pods to ensure scalability and stability.

---

### 🔄 Daily Backup Strategy

#### 1. Backup Strategy Using Velero for Persistent Volume Snapshots:
Velero is a tool that helps manage backups, restores, and disaster recovery in Kubernetes. You can use it to create volume snapshots of Globaleaks’ persistent storage.

##### Step-by-Step Setup of Velero:

Install Velero using Helm:
```bash
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm install velero vmware-tanzu/velero --namespace velero --create-namespace \
  --set-file credentials.secretContents.cloud=./credentials-velero \
  --set configuration.provider=aws \
  --set configuration.backupStorageLocation.name=default \
  --set configuration.backupStorageLocation.bucket=<YOUR-BUCKET> \
  --set configuration.volumeSnapshotLocation.name=default \
  --set configuration.volumeSnapshotLocation.config.region=<YOUR-REGION>
```
*Note: You will need to replace the AWS S3 bucket and region with your actual cloud storage configuration (e.g., AWS, Azure, GCP).*

Configure Velero for daily backups:
```bash
velero create schedule daily-backup --schedule="0 2 * * *" --include-namespaces=default
```
This creates a daily backup at 2:00 AM UTC for all resources within the default namespace, including Globaleaks data.

Take manual snapshots:
```bash
velero backup create globaleaks-backup --include-namespaces=default
```

Check backup status:
```bash
velero backup describe globaleaks-backup
```

Restore from a backup:
```bash
velero restore create --from-backup globaleaks-backup
```

---

#### 2. Backup Strategy Using Kubernetes CronJobs for File Backup:
In addition to volume snapshots, you can also back up specific Globaleaks data using Kubernetes CronJobs to schedule regular file backups.

##### Step-by-Step Setup of Kubernetes CronJobs:

Create a CronJob YAML file:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: globaleaks-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2:00 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: alpine
            volumeMounts:
            - name: globaleaks-data
              mountPath: /data
            command:
            - /bin/sh
            - -c
            - "tar -czf /backup/globaleaks-backup-$(date +\\%F).tar.gz -C /data ."
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: globaleaks-data
            persistentVolumeClaim:
              claimName: globaleaks-pvc
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
```

Apply the CronJob:
```bash
kubectl apply -f globaleaks-backup-cronjob.yaml
```

Verify the CronJob:
```bash
kubectl logs $(kubectl get pods --selector=job-name=globaleaks-backup -o jsonpath='{.items[0].metadata.name}')
```

Retrieve backups: You can retrieve the generated `.tar.gz` files from the backup storage (e.g., S3, NFS, or any storage solution mounted in the `backup-pvc`).

---

### 🌟 Final Notes:
- Ensure to replace `localhost` with your domain name in the IngressRoute file when deploying to production or non-local environments.
- The daily backup strategy using Velero for volume snapshots and Kubernetes CronJobs for file backups provides a robust and automated backup solution.

## 🔗 References

- [Globaleaks Docker Hub](https://hub.docker.com/r/globaleaks/globaleaks)
- [Traefik Routing in Kubernetes](https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/)
- [Rancher Desktop Installation Guide](https://docs.rancherdesktop.io/getting-started/installation/)
