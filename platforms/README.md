# Install Tanzu Community Edition
https://tanzucommunityedition.io/docs/latest/cli-installation/

```
brew install vmware-tanzu/tanzu/tanzu-community-edition
```
```
HOMEBREW-INSTALL-LOCATION}/configure-tce.sh
cd /opt/homebrew/Cellar/tanzu-community-edition/v0.9.1
libexec/configure-tce.sh
```
## Prepare Azure
https://tanzucommunityedition.io/docs/latest/azure-mgmt/

## Deploy a Standalone Cluster to Azure 
```
tanzu standalone-cluster create --ui
```

