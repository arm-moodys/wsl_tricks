# WSL Tricks

# Dev Setup Environment

## Table of Contents

- [Dev Setup Environment](#dev-setup-environment)
  - [Table of Contents](#table-of-contents)
    - [Using Company Certificates When Working on a Corporate Network](#using-company-certificates-when-working-on-a-corporate-network)
      - [Importing Company Certificates (Ubuntu)](#importing-company-certificates-ubuntu)
      - [Importing Company Certificates (RHEL)](#importing-company-certificates-rhel)
    - [General Tips](#general-tips)
    - [WSL - Use Corporate Certificates (RHEL)](#wsl---use-corporate-certificates-rhel)
    - [WSL - Use Corporate Certificates (Ubuntu)](#wsl---use-corporate-certificates-ubuntu)

## Required Steps

1. `wsl --update`
1. `wsl --install -d Ubuntu-24.04`
1. Create initial user.
1. Setup your init.sh (One Command Setup) script.
1. Clone this repo to your home directory.
1. `sudo su -`
1. Move the cloned repo under /opt/
1. Copy the `/opt/dev_env_setup/wsl_u24.04/wsl.conf` file under `/etc/`
1. (Optional): Copy the required certificates to the WSL intance:
   1. Copy the required certs from Windows to the WSL instance:

      ```bash
      cp /mnt/c/cert_for_wsl/* /usr/local/share/ca-certificates/ ; chmod 644 /usr/local/share/ca-certificates/* ; ll /usr/local/share/ca-certificates/
      ```

   1. Update and Upgrade:

      ```bash
      apt update ; apt upgrade
      ```
   
   1. Refresh all the certs:

      ```bash
      update-ca-certificates --fresh
      ```

1. Now run the setup script:

  ```bash
  1. Login again and udpate the dns server names:

   ```bash
  bash /opt/dev_env_setup/wsl_u24.04/01_run_as_root.sh SKIP_ROOT_PASS ; unlink /etc/resolv.conf ; echo -e "nameserver 1.1.1.1\nnameserver 1.0.0.1" > /etc/resolv.conf
  ```

1. Exit and shutdown the WSL instance:

   ```powershell
   wsl --shutdown
   ```

1. Wait 10 seconds.
1. Now you can start using your fully setup WSL instance.

### Using Company Certificates When Working on a Corporate Network

When using WSL (or any VM that is provisioned by the user and not the company) on a corporate machine/network we usually need to import the company's certificates on the WSL instance, this is because companies usually implement SSL MITM to monitor all traffic to prevent get malware into the company's network.

1. On the WSL machine, clone this repo under `/opt/`
1. On the Windows machine:
   1. Create a folder directly under the `c` drive using powershell:

      ```powershell
      New-Item -ItemType "directory" -Path "c:\cert_for_wsl"
      ```

   1. Copy the company certificates to `c:\cert_for_wsl`

#### Importing Company Certificates (Ubuntu)

> [!IMPORTANT]
> The steps below are provided so the user know what's the process to do so but this should be part of the automation setup process.

1. On the Ubuntu WSL instance
   1. Log in as root.
   1. Update certs:

      ```bash
      sudo update-ca-trust
      ````

   1. Copy the required certs from Windows to the WSL instance:

        ```bash
        cp /mnt/c/cert_for_wsl/* /usr/local/share/ca-certificates/ ; chmod 644 /usr/local/share/ca-certificates/*
        ```

   1. Refresh all the certs:

       ```bash
       sudo update-ca-certificates --fresh
       ```

#### Importing Company Certificates (RHEL)

> [!IMPORTANT]
> The steps below are provided so the user know what's the process to do so but this should be part of the automation setup process.

1. On the RHEL WSL instance:
   1. Log in as root.
   1. Update certs:

      ```bash
      update-ca-trust
      ````

   1. Copy the required certs from Windows to the WSL instance:

        ```bash
        cp /mnt/c/cert_for_wsl/* /etc/pki/ca-trust/source/anchors/ ; chmod 644 /etc/pki/ca-trust/source/anchors/*
        ```

   1. Refresh all the certs:

       ```bash
       update-ca-trust extract`
       ```

### General Tips

- Completely Uninstall Docker from Ubuntu:

    ```bash
    dpkg -l | grep -i docker ;  sudo apt-get purge -y docker-engine docker docker.io docker-ce docker-ce-cli ; sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce ; sudo rm -rf /var/lib/docker /etc/docker ; sudo rm /etc/apparmor.d/docker ; sudo groupdel docker ; sudo rm -rf /var/run/docker.sock
    ```

- Load scripts from `/etc/profile.d/` by Adding the following to the `~/.bashrc` of the 1st created user:
  >Note: This is not needed as automation takes for this but leaving here just in case

    ```bash
    if [ -d /etc/profile.d ]; then
    for i in /etc/profile.d/*.sh; do
        if [ -r $i ]; then
        . $i
        fi
    done
    unset i
    fi
    ```

### WSL - Use Corporate Certificates (RHEL)

1. Go to Manage User Certificates
1. Trusted Root Certification Authorities
1. Certificates
1. Open the root CA certificate you are interested in
1. Details
1. Copy To File
1. Next
1. Select "Base64 encoded X.509"
1. **Important**: Save the certs using the .crt extension to a share location
1. Go to the RHEL machine
1. `sudo update-ca-trust`
1. `sudo cp <path_to_crt_file>/* /etc/pki/ca-trust/source/anchors/`
1. `update-ca-trust extract`
1. Now let's verify we can reach https websites `curl https:///example.com`

### WSL - Use Corporate Certificates (Ubuntu)

1. Go to Manage User Certificates
1. Trusted Root Certification Authorities
1. Certificates
1. Open the root CA certificate you are interested in
1. Details
1. Copy To File
1. Next
1. Select "Base64 encoded X.509"
1. **Important**: save the certs using the .crt extension to a share location
1. Go to the Ubuntu machine
1. `sudo update-ca-trust`
1. `sudo cp <path_to_crt_file>/* /usr/local/share/ca-certificates/`
1. `sudo update-ca-certificates --fresh`
1. Now let's verify we can reach https websites `curl https:///example.com`
