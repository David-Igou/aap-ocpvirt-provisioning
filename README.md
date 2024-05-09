# aap-ocpvirt-provisioning

It is recommended that you fork this repo and experiment with your own changes.

This repository contains automation for OCP, AAP, and OCP Virtualization provisioning. All playbooks are not needed to start provisioning, they are only included for readers that are potentially unfamilar with parts of the platform. The Openshift Virtualization inventories and playbooks can easily be integrated into a pre-existing AAP Project and Inventory in your instance without running the configure-aap playbook.

## Prerequisites

An Openshift cluster running:

* Openshift Virtualization Operator
* Ansible Automation Platform Operator

It is assumed in this exercise that AAP and the Openshift Virtualization VMs all use the internal cluster network. That said, the steps should be similar if you are utilizing an external AAP and use Multus on the OCP cluster, provided you update the inventory plugin to query for the external network interfaces on the VMs.

## Install Ansible dependencies

This repository makes use of the following Ansible collections:

- `redhat.openshift_virtualization`

- `infra.controller_configuration`

- `kubevirt.core`

and the following plugins:

- `redhat.openshift_virtualization.kubevirt`

These collections and plugins are available for download via the ansible-galaxy automation hub. In order to download these collections you first need to ensure that `/etc/ansible/ansible.cfg` file is configured to interact with RedHat automation hub using your `offline token`.

You can find your offline token by navigating to https://console.redhat.com/ansible/automation-hub/token

![](./docs/pics/token_url.png)

**_Click "Load Token" to obtain your token._**

You will also need to grab your `SSO URL` and published `Server URL`. These url's can be found on the same page as your offline token.


Once you have your `offline token`, `sso url`, and `server url` you need to update your `/etc/ansible/ansible.cfg` file to the following configuration

    [galaxy]
    server_list = automation_hub, galaxy

    ## config for galaxy server (listed in server_list)
    [galaxy_server.galaxy]
    url=https://galaxy.ansible.com

    ## config for automation_hub (listed in server_list)
    [galaxy_server.automation_hub]
    url=<Server URL>
    auth_url=<SSO URL>

    token=<offline token>


If your `ansible.cfg` is configured correctly. You shoule be able to run the following command successfully

    $ ansible-galaxy install -r requirements.yml

![](./docs/pics/successful_install.png)

If you recieve this error message then your `ansible.cfg` is failing to authenticate properly. Please review your `/etc/ansible/ansible.cfg` 

![](./docs/pics/failed_install.png)

## Create openshift serviceaccounts and obtain tokens

```bash
cd openshift-serviceaccount-manifests
vim kustomization.yaml # Set namespace if required
oc apply -k .
oc create token virtualmachine-deployer -n <namespace> --duration=$((365*24))h
oc create token virtualmachine-reader -n <namespace> --duration=$((365*24))h
```

## Create aap_vars.yml

In production, it is recommended to use a service account or token for configuring AAP as code

```bash
cp aap_vars.yml.example aap_vars.yml
vim aap_vars.yml
```

If you forked this repo, update the git_repo field to your fork

```yaml
---
ocp_api: https://api.openshift.cluster:6443
inventory_sa_token: "changeme"
provisioner_sa_token: "changeme"
git_repo: https://github.com/David-Igou/aap-ocpvirt-provisioning.git
aap_organization: igou
ansible_user: cloud-user
ansible_user_private_key: |-
  -----BEGIN OPENSSH PRIVATE KEY-----
  -----END OPENSSH PRIVATE KEY-----
ansible_user_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFPs4Ix5OQSAQnC/TLjxVGyzX+1ClHpWE2+6sT6ufVGR"
controller_hostname: https://aap.apps.openshift.cluster
controller_username: cac-user
controller_password: "foobar"
```

### Optional: Set the scope of Openshift Virtualization Inventory

If you are working in a multi-tenant environment, you might want to update the inventory plugin to only discover VirtualMachines from certain namespaces, for example:

```yaml
---
plugin: redhat.openshift_virtualization.kubevirt
connections:
  - namespaces:
      - igou
```

## Configure AAP

To configure your AAP instance, create an SCM Project using your forked repository, and then create supporting objects

<Table of objects and values will go here>



### Ansible

```
ansible-galaxy install -r requirements.yml
ansible-playbook playbooks/configure-aap.yml -i inventory
```

### Ansible-Navigator

If you have an execution environment containing the dependencies, you can use it in Ansible Navigator by running:

`ansible-navigator run playbooks/aap/configure-aap.yml -e "@aap_vars.yml" -i inventory/`

##

When your Inventory and SCM source are configured,