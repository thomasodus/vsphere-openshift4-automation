# vsphere openshift4 automation

VERY BASIC playbook using `openshift-install` and `govc` to create an openshift cluster in vsphere.

- Downloads Installer
- Generates Manifests
- Generates Ignition Configs
- Places the ignition configs in a local apache directory (for the VMs to fetch later)
- Creates VMs (prereq: template must exist in the cluster)
- Modifies VMs to add "append" ignition configs to them - to tell the VMs to get the config from the http server

Instructions followed as per: https://blog.openshift.com/openshift-4-2-vsphere-install-quickstart/

## How to run

I run this playbook from the same host that has had https://github.com/christianh814/ocp4-upi-helpernode run on it.

Ensure you have `govc` installed. 

```
curl -L https://github.com/vmware/govmomi/releases/download/v0.21.0/govc_linux_amd64.gz | gunzip > /usr/local/bin/govc
chmod +x /usr/local/bin/govc
```

Ensure `/usr/local/bin` is on your `$PATH`:

```
echo "PATH=\$PATH:/usr/local/bin" >> ~/.bashrc
source ~/.bashrc
```


And ensure you have the appropriate GOVC environment variables populated:

```
export GOVC_USERNAME=administrator@vsphere.local
export GOVC_PASSWORD=password
export GOVC_URL=my.vcenter.lan
export GOVC_INSECURE=true
```
Ensure you have the appropriate vars file populated as per the playbook.

Run the playbook with:

```
ansible-playbook create-vsphere-machines.yml -vv
```

Please note that everything happens in a temp directory:

```
TASK [create-ocp4-manifests : Create temporary directory] **********************************************************************************************************
task path: /some/path/vsphere-openshift4-automation/roles/create-ocp4-manifests/tasks/setup.yml:2
changed: [localhost] => {"changed": true, "gid": 1000, "group": "hello", "mode": "0700", "owner": "user", "path": "/tmp/ansible.s4i72i7g", "secontext": "unconfined_u:object_r:user_tmp_t:s0", "size": 40, "state": "directory", "uid": 1000}
```

In this case everything will be in `/tmp/ansible.s4i72i7g`

## Sample vars file:

```
---
openshift_installer_url: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux-4.2.4.tar.gz
openshift_client_url: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux-4.2.4.tar.gz
cluster_id: ocp1
cluster_domain: example.lab
ssh_key_file: /root/.ssh/id_rsa.pub
pull_secret_file: /root/pull-secret.json
ignition_base_url: http://some-accessible-http-site:8080/ignition
ignition_file_destination: /var/www/html/ignition/
folder:
network: VM Network
machines:
- name: bootstrapy
  host: esxhost1
  template: worker-bootstrap-template
  cores: 4
  ram: 8192
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AA
  disk: 120G
  type: bootstrap
- name: master0y
  host: esxhost1
  template: master-template
  cores: 4
  ram: 16384
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AB
  disk: 120G
  type: master
- name: master1y
  host: esxhost1
  template: master-template
  cores: 4
  ram: 16384
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AC
  disk: 120G
  type: master
- name: master2y
  host: esxhost1
  template: master-template
  cores: 4
  ram: 16384
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AD
  disk: 120G
  type: master
- name: worker0y
  host: esxhost1
  template: worker-bootstrap-template
  cores: 4
  ram: 8192
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AE
  disk: 120G
  type: worker
- name: worker1y
  host: esxhost1
  template: worker-bootstrap-template
  cores: 4
  ram: 8192
  datastore: esxhost1-2tb
  mac: 00:50:56:AA:AA:AF
  disk: 120G
  type: worker
```