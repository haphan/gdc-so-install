## GDC Software Only installation notes

### Preload nodes with Ubuntu - headless installation

This steps are to consistently bootstrap base OS for all baremetal nodes intended to use in Admin Clusters and User Clusters using Canonical autoinstall method.


```bash
wget https://raw.githubusercontent.com/cloudymax/pxeless/refs/heads/develop/image-create.sh
bash image-create.sh -a -u config.yaml -s ./ubuntu-22.04.5-live-server-amd64.iso -d autoinstall.iso
```

the example of the `config.yaml`, update to your own settings

```yaml
cat config.yaml
#cloud-config
autoinstall:
  version: 1
  keyboard: {layout: us, toggle: null, variant: ''}
  locale: en_US.UTF-8
  identity:
    hostname: ubuntu-server
    username: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    # You must generate a hashed password for the user.
    # You can generate one using: openssl passwd -6
    # Example password is "password"
    password: "$6$2.E0LxOi5uSti8RZ$hNbUbs9sexkQ2c6HsyHV2F1hxzdHH85U0eT1/91BNhAhOSjWq4yxRVilMEJSdFQ8M.rjLQe7avwQIAsCGR/lT/"
  ssh:
    install-server: true
    allow-pw: false
    authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDJVuqn/EX52+wh02rKZbYjEmeUl/Iold2uoDub6CRNvhvnMPWtLssrT+5i9wNesg+NC/fPxIQquGc9SH09CsoZYzxmpVagHzLqzwgt3ZFMfn2VF+tqez7fkMejKCCWzCn+9SStVfYh1ad7feNrkrbZx5IAUOeY+dIakFRVGEEDhiwMdKSkudt+rus3UknPxrDw+PYCWXrXR01PNS0FikSthXkPJ0kxDSwSTYfJzFBAAKd0g9L5/JYWj0OpaFcoAQ66GYdjkotQGGj48d7jK9dBp8s6rxW3Lq773q/04vSwrhcPbRn+xqcrozl3SDFaIYTVpcmZetNuOTYrDcswvBwH thanhha.work@gmail.com
  packages:
    - openssh-server
  user-data:
    write_files:
      - path: /etc/ssh/sshd_config.d/60-disable-password-auth.conf
        content: |
          PasswordAuthentication no
```


Configure passwordless sudo manually, this step apparently can be added to autoinstall script above

echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ubuntu

```bash
# increase system limit 
# fs.inotify.max_user_instances: 8192 
# fs.inotify.max_user_watches: 524288

echo 'fs.inotify.max_user_instances=8192' | sudo tee --append /etc/sysctl.conf
echo 'fs.inotify.max_user_watches=524288' | sudo tee --append /etc/sysctl.conf

# reboot or reload sysctl
sysctl --system
```


###  Configure base OS for Admin workstation (can be VM)

```bash
# disale ufw firewall
ufw disable && ufw status 

# install docker runtime
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common \
  docker.io
docker version
```

```bash
# increase system limit 
# fs.inotify.max_user_instances: 8192 
# fs.inotify.max_user_watches: 524288

echo 'fs.inotify.max_user_instances=8192' | sudo tee --append /etc/sysctl.conf
echo 'fs.inotify.max_user_watches=524288' | sudo tee --append /etc/sysctl.conf

# reboot or reload sysctl
sysctl --system
```


### Installing Utils in admin workstation

make sure to install [gcloud-cli first](https://cloud.google.com/sdk/docs/install#deb)

```bash
# this goes in your 
gloud login
gcloud storage cp gs://anthos-baremetal-release/bmctl/1.32.200-gke.104/linux-amd64/bmctl .
chmod +x ./bmctl
```

configure google cloud resources
```bash
gcloud services enable --project=haph-sandbox-sandbox-971300 \
    anthos.googleapis.com \
    anthosaudit.googleapis.com \
    anthosgke.googleapis.com \
    cloudresourcemanager.googleapis.com \
    compute.googleapis.com \
    connectgateway.googleapis.com \
    container.googleapis.com \
    gkeconnect.googleapis.com \
    gkehub.googleapis.com \
    gkeonprem.googleapis.com \
    iam.googleapis.com \
    kubernetesmetadata.googleapis.com \
    logging.googleapis.com \
    monitoring.googleapis.com \
    opsconfigmonitoring.googleapis.com \
    serviceusage.googleapis.com \
    stackdriver.googleapis.com \
    storage.googleapis.com

```

### Install ADMIN Cluster

To run these command from Admin workstation. Making sure it can reach all the nodes intended for Admin cluster via ssh and can perform passwordless sudo.

```bash
bmctl create config -c ADMIN_CLUSTER_NAME
```

The file is created at `bmctl-workspace/ADMIN_CLUSTER_NAME/ADMIN_CLUSTER_NAME.yaml`.  Update this yaml file according to the design. See (sample)[https://cloud.google.com/kubernetes-engine/distributed-cloud/bare-metal/docs/reference/config-samples#user_clusters] here.

One yaml file is done, create the cluster 

```bash
bmctl create cluster -c ADMIN_CLUSTER_NAME
```

After this step is done, an `kubeconfig` file for admin cluster is generate in `bmctl-workspace/ADMIN_CLUSTER_NAME/`, use this kubeconfig to validate all nodes are in `Ready` state.

### Install USER Cluster


```bash
bmctl create config -c USER_CLUSTER_NAME
```

The file is created at `bmctl-workspace/USER_CLUSTER_NAME/USER_CLUSTER_NAME.yaml`. Update this yaml file according to the design. 


```bash
bmctl create cluster -c USER_CLUSTER_NAME –-kubeconfig $ADMIN_KUBECONFIG
```


