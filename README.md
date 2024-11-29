# Zabbix on K3S
* Setup Zabbix on K3S with helm
  * Values: https://github.com/zabbix-community/helm-zabbix/blob/master/charts/zabbix/README.md
* Add Backup with retention


## Setup Zabbix
```bash
genpasswd() {
   local l=$1
   [ "$l" == "" ] && l=16
   tr -dc A-Za-z0-9_=., < /dev/urandom | head -c ${l} | xargs 
}

HOST=mon.domain.tld

helm repo add zabbix-community https://zabbix-community.github.io/helm-zabbix
helm search repo zabbix-community/zabbix -l
export ZABBIX_CHART_VERSION='5.0.2'

helm upgrade --install zabbix zabbix-community/zabbix  \
  --dependency-update \
  --create-namespace \
  --version $ZABBIX_CHART_VERSION \
  -n monitoring \
  --set postgresql.persistence.enabled=true \
  --set postgresql.persistence.storageSize=10Gi \
  --set postgresAccess.password=$(genpasswd) \
  --set ingressRoute.enabled=true \
  --set ingressRoute.hostName=$HOST \
  --debug

kubectl get pv

## Update Zabbix

To update Zabbix to the latest version using Helm, follow these steps:

1. Update the Helm repository to ensure you have the latest chart versions:

   ```bash
   helm repo update
   ```

2. Set the desired Zabbix chart version. You can find the latest version by searching the Helm repository:

   ```bash
   helm search repo zabbix-community/zabbix -l
   export ZABBIX_CHART_VERSION='5.0.2'  # Replace with the latest version
   ```

3. Upgrade your existing Zabbix installation:

   ```bash
   helm upgrade zabbix zabbix-community/zabbix \
     --version $ZABBIX_CHART_VERSION \
     -n monitoring \
     --reuse-values \
     --debug
   ```

   The `--reuse-values` flag ensures that your current configuration is retained during the upgrade.
```

## Uninstal Zabbix
```bash
helm uninstall zabbix -n monitoring
kubectl delete namespace monitoring
```

## Install Backup
```bash
kubectl config set-context --current --namespace=monitoring
for y in zabbix-backup-pvc.yml zabbix-backup-cron.yml; do echo "install $y"; kubectl apply -f $y; done
```

