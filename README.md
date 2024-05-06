# aap-ocpvirt-provisioning

It is recommended that you fork this repo and experiment with your own changes.

## Prerequisites

WIP

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
ansible_user_private_key: |-
  -----BEGIN OPENSSH PRIVATE KEY-----
  -----END OPENSSH PRIVATE KEY-----
ansible_user_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFPs4Ix5OQSAQnC/TLjxVGyzX+1ClHpWE2+6sT6ufVGR"
controller_hostname: https://aap.apps.openshift.cluster
controller_username: cac-user
controller_password: "foobar"
```

## Configure AAP

### Ansible

```
ansible-galaxy install -r requirements.yml
ansible-playbook playbooks/configure-aap.yml -i inventory
```

### Ansible-Navigator

If you have an execution environment containing the dependencies, you can use it in Ansible Navigator by running:

`ansible-navigator run playbooks/aap/configure-aap.yml -e "@aap_vars.yml" -i inventory/`

