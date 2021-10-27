There are 2 branches to this repo:

* mktplace has the original code using the AMP offering from tanzu
* master:  updated, experimental ARM template that uses premium storage, etc

## Migrate GPDB to Azure

Here is the process I used:

Open [cloudshell](shell.azure.com) from the [Azure portal](portal.azure.com) and choose the `bash` shell option

```bash
# change vars as needed
# this block needs to be rerun if cloudshell times out
# run `watch ls` in cloudshell to avoid the timeout and ctl+c to abort it
export SUBSCRIPTION=airs
export RG=rexall
export SNAPSHOTRG=rexall_snapshot
export LOCATION=eastus
masterInstanceType="Standard_H16"
segmentInstanceType="Standard_H16"
databaseVersion=GP6
masterDiskSize=500
segmentDiskSize=8000
segmentInstanceCount=2
adminPublicKey="ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAt7NuDNYDwBNObe5n6b5PRxy3/eQzqQVvBUZ0hDcYPbdaxxw+e6Yc5pmEXsl6lkCGX2GBCEMC2FX7jB4mfjq9sLrT9t83gTBZ71zZY6xLalI6G2jEAprCB9wDFRHSoO9LNZU9VYhkYPY+0mv1CTPmY5HdenWdJ6wvCBU5R3iGju0Fz7FGtqD4JKfoY/Z9OsObLq2xG/5+Tgw72e+evJSIAG6j3ix+AgO2aoqMI0npUiPDg16lbMglwTbv79wg/cRnf/D5SLJMVjaB6jGH4s2iXnI1lWOgcoaYB6D95zLS5VNYDnqtYVhysXa+AX3YM/ITTSgyEvAteWkJM7088rTM5Q== DWentzel@RAD-1DWENTZE-LT\n"

az account set --subscription $SUBSCRIPTION

# setup the code repo
mkdir -p git
cd git
rm -rf gpdb
git clone https://github.com/davew-msft/gpdb gpdb
cd gpdb

# rg creation
az group create --name $RG --location $LOCATION
az group create --name $SNAPSHOTRG --location $LOCATION

# fix the parameters as needed
jq --arg LOCATION "$LOCATION" '.parameters.location.value = $LOCATION' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg SNAPSHOTRG "$SNAPSHOTRG" '.parameters.backupResourceGroup.value = $SNAPSHOTRG' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg masterInstanceType "$masterInstanceType" '.parameters.masterInstanceType.value = $masterInstanceType' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg segmentInstanceType "$segmentInstanceType" '.parameters.segmentInstanceType.value = $segmentInstanceType' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg databaseVersion "$databaseVersion" '.parameters.databaseVersion.value = $databaseVersion' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg masterDiskSize "$masterDiskSize" '.parameters.masterDiskSize.value = $masterDiskSize' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg segmentDiskSize "$segmentDiskSize" '.parameters.segmentDiskSize.value = $segmentDiskSize' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg segmentInstanceCount "$segmentInstanceCount" '.parameters.segmentInstanceCount.value = $segmentInstanceCount' parameters.json > parameters.tmp && mv parameters.tmp parameters.json
jq --arg adminPublicKey "$adminPublicKey" '.parameters.adminPublicKey.value = $adminPublicKey' parameters.json > parameters.tmp && mv parameters.tmp parameters.json

az deployment group create \
  --resource-group $RG \
  --template-file template.json \
  --parameters @parameters.json


# to remove everything ...
az group delete --name $RG
az group delete --name $SNAPSHOTRG
```