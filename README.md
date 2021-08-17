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

az account set --subscription $SUBSCRIPTION

# setup the code repo
mkdir -p git
git clone https://github.com/davew-msft/gpdb
cd gpdb

# rg creation
az group create --name $RG --location $LOCATION
az group create --name $SNAPSHOTRG --location $LOCATION

# fix the parameters as needed



# networking
az network vnet create \
  --name $VNET \
  --resource-group $RG \
  --address-prefix $ADDR_PREFIX \
  --subnet-name $SUBNET \
  --subnet-prefix $SUB_PREFIX

SUBNET_ID=$(az network vnet subnet show \
 --resource-group $RG \
 --vnet-name $VNET \
 --name $SUBNET \
 --query id -o tsv)


```