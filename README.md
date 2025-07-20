## GDC Software Only installation notes

#### Prepare autoinstall iso 


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


May need to configure passwordless sudo

echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ubuntu

##  Configure base OS

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

# reboot
```
