```{include} ./links.md
```
# Gitlab + NAS setup

For this setup, we will use the University of Geneva's NAS storage service to store data and the University of Geneva's GitLab instance for version control, metadata storage, and collaboration. This setup allows you to manage your data and projects effectively while leveraging the capabilities of both NAS and GitLab.

## Prerequisites

- [ ] [Install uv](install.md#uv)
- [ ] [Create SSH keys](install.md#ssh-keys)
- [ ] [Setup GitLab](install.md#gitlab)
- [ ] [Setup NAS](install.md#nas)


## NAS storage

For this setup, we will use the University of Geneva's {term}`NAS` storage service to store data. Sone of the advantages of using NAS storage include:
    - High storage capacity
    - On demand storage expansion
    - Fast data access
    - Automatic data backup and redundancy

To use the NAS storage, you can use an existing storage location or create a new one. You can ask for a new storage location by opening a ticket on the {ref}`unige-digitalworkspace` page: **NASac - Storage space for research data**. At the time of writing, the cost of NAS storage is 75 CHF per TB per year, with the first 1TB being free. Some faculties may contribute to the cost of storage.

The following parameters related to the NAS storage will be needed for the setup:

- `[NAS_URL]`: the URL of the NAS storage.
- `[ISIS_ID]`: your ISIS ID, which is used for authentication and access control of UNIGE services. 
- `[ISIS_PASSWORD]`: your ISIS password, which is used for authentication and access control of UNIGE services. 

## Setting up the remote machine

For this setup, we will use a remote machine to manage interaction with {term}`NAS`. This can be a {term}`VM` or a physical machine that has access to the NAS storage. The remote machine will act as an intermediary between your local machine and the NAS, allowing you to manage your data and projects effectively.
For maximum compatibility, we recommend using a Linux-based remote machine, as it provides better support for the tools and workflows we will be using.

### VM setup

If you choose to use a {term}`VM`, you can request one from the University of Geneva's IT services by opening a ticket on the {ref}`unige-digitalworkspace` page. **Infrastructure - Hosting as a Service : Submit a request** `>` **IaaS - virtual server hosting (Linux, Windows)** `>` **Create** . We recommend to select a Linux-based operating system for the VM, such as Ubuntu or CentOS. The cost of a VM depends on the resources allocated to it, such as CPU, RAM, and storage. The smallest VM configuration (1 vCPU, 1 GB RAM, 50 GB storage) should be sufficient for most use cases and costs around 37 CHF / year.

The following parameters related to the VM will be needed for the setup:
- `[VM_IP]`: the IP address of the VM, which is used to connect to the VM remotely.
- `[VM_USERNAME]`: the username for the VM, which is used for authentication when connecting to the VM remotely.
- `[VM_PASSWORD]`: the password for the VM, which is used for authentication when connecting to the VM remotely.

### Mounting NAS on the remote machine.

Instal required packages for mounting NAS on the remote machine:
``` bash
sudo apt install cifs-utils psmisc
```

Create a `~/credentials` file on the remote machine with the following content:

```json
username=[ISIS_ID]
password=[ISIS_PASSWORD]
domain=ISIS
```

To make sure that the credentials file is secure, you can set the permissions of the file to be readable only by the root user:
```bash
chmod 600 ~/credentials
sudo chown root:root credentials
```

Then mount the NAS storage on the remote machine using the following command where you replace `[NAS_URL]` with the URL of the NAS storage and `[DESTINATION_PATH]` with the path where you want to mount the NAS storage on the remote machine:

```bash
sudo mount -t cifs -o credentials=~/credentials,uid=$(id -u),gid=$(id -g),file_mode=0664,dir_mode=0775 //[NAS_URL] [DESTINATION_PATH]
```

## Gitlab setup