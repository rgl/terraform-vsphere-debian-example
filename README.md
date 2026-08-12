# Usage (Ubuntu 24.04 host)

Install the [Debian 13 VM template](https://github.com/rgl/debian-vagrant).

Install Terraform and govc (Debian):

```bash
wget https://releases.hashicorp.com/terraform/1.15.8/terraform_1.15.8_linux_amd64.zip
unzip terraform_1.15.8_linux_amd64.zip
sudo install terraform /usr/local/bin
rm terraform terraform_*_linux_amd64.zip
wget https://github.com/vmware/govmomi/releases/download/v0.55.1/govc_Linux_x86_64.tar.gz
tar xf govc_Linux_x86_64.tar.gz govc
sudo install govc /usr/local/bin/govc
rm govc govc_Linux_x86_64.tar.gz
```

Save your environment details as a script that sets the terraform variables from environment variables, e.g.:

```bash
cat >secrets.sh <<'EOF'
export TF_VAR_vm_hostname_prefix='example'
export TF_VAR_vm_count='1'
export TF_VAR_vm_cpu='2'
export TF_VAR_vm_memory='1' # [GiB]
export TF_VAR_vm_disk_os_size='10' # [GiB]
export TF_VAR_vsphere_user='administrator@vsphere.local'
export TF_VAR_vsphere_password='password'
export TF_VAR_vsphere_server='vsphere.local'
export TF_VAR_vsphere_datacenter='Datacenter'
export TF_VAR_vsphere_compute_cluster='Cluster'
export TF_VAR_vsphere_datastore='Datastore'
export TF_VAR_vsphere_network='VM Network'
export TF_VAR_vsphere_folder='example'
export TF_VAR_vsphere_debian_template='vagrant-templates/debian-13-uefi-amd64'
export GOVC_INSECURE='1'
export GOVC_URL="https://$TF_VAR_vsphere_server/sdk"
export GOVC_USERNAME="$TF_VAR_vsphere_user"
export GOVC_PASSWORD="$TF_VAR_vsphere_password"
EOF
```

**NB** You could also add these variables definitions into the `terraform.tfvars` file, but I find the environment variables more versatile as they can also be used from other tools, like govc.

Launch this example:

```bash
source secrets.sh
# see https://github.com/vmware/govmomi/blob/master/govc/USAGE.md
govc version
govc about
govc datacenter.info # list datacenters
govc find # find all managed objects
terraform init
terraform plan -out=tfplan
time terraform apply tfplan
ssh-keygen -f ~/.ssh/known_hosts -R "$(terraform output --json ips | jq -r '.[0]')"
ssh "vagrant@$(terraform output --json ips | jq -r '.[0]')"
sudo cloud-init schema --system --annotate
exit
time terraform destroy --auto-approve
```

List this repository dependencies (and which have newer versions):

```bash
GITHUB_COM_TOKEN='YOUR_GITHUB_PERSONAL_TOKEN' ./renovate.sh
```
