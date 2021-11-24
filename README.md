# Tanzu On Demand Service Broker VM Extensions Addon

## What does this do?

This will insert custom VM extensions on any Tanzu Application Service tile that uses the On Demand Service Broker SDK.


## How do I install it?

1. Open a shell prompt on a BOSH CLI with access to your TKGI bosh director, such as Ops Manager.
2. Export your BOSH credentials to the enviornment.  These can be accessed via the Ops Manager GUI -> BOSH Director Tile -> Credentials Tab -> Bosh Commandline Credentials.    

e.g.
```
export BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=fakesecret BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate  BOSH_ENVIRONMENT=10.0.0.10
```
3. Copy or clone this repository onto this BOSH CLI workstation and create+upload the BOSH release to the director

```
git clone https://github.com/svrc/tanzu-osb-vm-extensions && cd tanzu-osb-vm-extensions
bosh create-release --force
bosh upload-release ./dev_releases/tanzu-osb-vm-extensions/tanzu-osb-vm-extensions-0+dev.1.yml 

```
4. Create a vm extension yml and upload it to the BOSH director.   See the example-vm-ext.yml for this, which places on-demand service instances into their own dedicated Azure resource group.   This is just an example, you can do anything you want with your target CPI. 
```
bosh -n update-config --name=external_rg --type=cloud ./example-vm-ext.yml
```
5. Configure the addon from this repo for each tile you want.   **Edit the addon.yml** to point to the BOSH deployment name for your tile.   Add array entries for more tile deployments, or create more addons.yml with different names if you need different vm extensions lists for different tiles. 
```
bosh -n update-config --name=insert-vm-ext --type=runtime ./addon.yml
```
6. "Apply Changes" in Ops Manager to apply the addon to your tiles.

7. Warning:  Existing on-demand instances will need to be manually updated at the BOSH level, i.e. you'll need to add the vm extensions to them manually.  This MUST be done prior to attempting ugprades/updates as the OSB won't like that its "current state" BOSH manifest doesn't match the "actual state" manifest.   But the bright side is, these changes will survive upgrades!
