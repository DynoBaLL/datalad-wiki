```{include} ../links.md
```

```{figure} ../_static/img/scenario_02.png
---
width: 100%
alt: Gitlab + S3 setup
name: gitlab_s3_setup
---
The Gitlab + S3 setup, where git is used for version control and metadata storage on GitLab, and git-annex is used for data management on S3 object storage. Each user's machine talks directly to both services: no intermediate machine is needed.
```

# Gitlab + S3 setup

For this setup, we will use the University of Geneva's S3 object storage service to store data and the University of Geneva's GitLab instance for version control, metadata storage, and collaboration. Unlike the [Gitlab + NAS setup](gitlab_nas.md), no {term}`VM` is needed: {term}`git-annex` speaks the S3 protocol natively, so every user's machine connects to the storage directly.

```{note}
Once the project is set up, a single `clone` command is enough for all users to access the project. The S3 sibling is enabled automatically, so users don't need to worry about the operating system, sibling configuration, or publish dependencies. They only need their own S3 credentials.
```

## Prerequisites

- [ ] [Install uv](install.md#uv)
- [ ] [Create SSH keys](install.md#ssh-keys)
- [ ] [Setup GitLab](install.md#gitlab)
- [ ] [Setup GitLab-python](install.md#gitlab-python)
- [ ] An S3 tenant (see [S3 storage](#s3-storage))


## Project setup

(s3-storage)=
### S3 storage

For this setup, we will use the University of Geneva's {term}`S3` object storage service, based on Hitachi Content Platform (HCP), to store data. Some of the advantages of using S3 storage include:
    - High storage capacity
    - On demand storage expansion
    - Direct access over HTTPS from any machine (laptop, {term}`HPC`, ...), without mounting a drive or using a {term}`VM`
    - Per-user credentials and permissions

You can request S3 storage by opening a ticket on the [UNIGE Digital Workspace](https://unige-dwp.onbmc.com/) page. More details can be found on [the University of Geneva's S3 storage page](https://catalogue-si.unige.ch/en/stockage-objet-S3-2).

The storage is organized as follows:
- a **tenant**, administered by the project manager, which has its own address (`[S3_HOST]`) and administration console.
- one or more **namespaces** inside the tenant. In S3 terminology, a namespace is a **bucket** (`[S3_BUCKET]`).
- **users**, created by the tenant administrator, who are given read / write permissions on namespaces.

The following parameters related to the S3 storage will be needed for the setup:

- `[S3_HOST]`: the address of the S3 tenant.
::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
[S3_HOST]
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
datalad.s3.unige.ch
```
:::
::::

- `[S3_BUCKET]`: the name of the namespace (bucket) where the data will be stored.
::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
[S3_BUCKET]
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
datalad
```
:::
::::

(user-creation)=
#### User creation

Each user needs an account on the tenant with read / write permissions on the namespace. The tenant administrator creates it from the HCP tenant administration console, available at `https://[S3_HOST]:8000`.

1. **Security** `>` **Users** `>` **Create User Account**: choose a username (`[S3_USERNAME]`) and a password (`[S3_PASSWORD]`).
2. **Security** `>` **Users** `>` **Assign Namespace Permissions**: select the namespace `[S3_BUCKET]` and grant at least the **Read** and **Write** permissions.

(api-credentials)=
#### API credentials

HCP derives the S3 API credentials from the user's username and password:
- `[AWS_ACCESS_KEY_ID]` is the base64 encoding of the username.
- `[AWS_SECRET_ACCESS_KEY]` is the MD5 hash of the password.

Each user can compute their own credentials:

::::{tab-set}

:::{tab-item} template (Linux / macOS)
:sync: 0
```{code-block} bash
echo -n "[S3_USERNAME]" | base64
echo -n "[S3_PASSWORD]" | md5sum | awk '{print $1}'
```
:::
:::{tab-item} example (Linux / macOS)
:sync: 1
```{code-block} bash
echo -n "jdoe" | base64
# amRvZQ==
echo -n "a very strong password" | md5sum | awk '{print $1}'
# 14850622ad686e969c25e0befc48fc6a
```
:::
::::

::::{tab-set}

:::{tab-item} template (Windows)
:sync: 0
```{code-block} powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("[S3_USERNAME]"))
$md5 = [Security.Cryptography.MD5]::Create()
-join ($md5.ComputeHash([Text.Encoding]::UTF8.GetBytes("[S3_PASSWORD]")) | ForEach-Object { $_.ToString("x2") })
```
:::
:::{tab-item} example (Windows)
:sync: 1
```{code-block} powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("jdoe"))
# amRvZQ==
$md5 = [Security.Cryptography.MD5]::Create()
-join ($md5.ComputeHash([Text.Encoding]::UTF8.GetBytes("a very strong password")) | ForEach-Object { $_.ToString("x2") })
# 14850622ad686e969c25e0befc48fc6a
```
:::
::::

git-annex reads the credentials from the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables:

::::{tab-set}

:::{tab-item} template (Linux / macOS)
:sync: 0
```{code-block} bash
export AWS_ACCESS_KEY_ID="[AWS_ACCESS_KEY_ID]"
export AWS_SECRET_ACCESS_KEY="[AWS_SECRET_ACCESS_KEY]"
```
:::
:::{tab-item} template (Windows)
:sync: 1
```{code-block} powershell
$env:AWS_ACCESS_KEY_ID="[AWS_ACCESS_KEY_ID]"
$env:AWS_SECRET_ACCESS_KEY="[AWS_SECRET_ACCESS_KEY]"
```
:::
::::

````{dropdown} [Optional] Keeping the credentials across sessions.

The commands above only set the variables for the current terminal session. To keep them, add the `export` lines to your `~/.bashrc` (Linux / macOS), or set them as user environment variables (Windows):

```powershell
[Environment]::SetEnvironmentVariable("AWS_ACCESS_KEY_ID", "[AWS_ACCESS_KEY_ID]", "User")
[Environment]::SetEnvironmentVariable("AWS_SECRET_ACCESS_KEY", "[AWS_SECRET_ACCESS_KEY]", "User")
```

Open a new terminal for the change to take effect.
````

```{danger}
The S3 credentials give access to all the data of the namespace. Do not share them, and never commit them to any repository or share them in any public space. Each user should have their own account.
```

### Datalad setup

::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
datalad create -c text2git [PROJECT_NAME]
cd [PROJECT_NAME]
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
datalad create -c text2git project_datalad
cd project_datalad
```
:::
::::


#### Add S3 sibling

Make sure the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables are set (see [API credentials](#api-credentials)), then create the S3 special remote:

::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
git annex initremote s3 type=S3 encryption=none bucket=[S3_BUCKET] host=[S3_HOST] port=443 protocol=https autoenable=true
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
git annex initremote s3 type=S3 encryption=none bucket=datalad host=datalad.s3.unige.ch port=443 protocol=https autoenable=true
```
:::
::::

```{tip}
- `autoenable=true` makes the S3 sibling available automatically to every user who clones the dataset.
- The credentials are read from the environment variables and are **not** stored in the dataset, so each user uses their own.
- Several datasets can share the same bucket. To keep the content of each dataset in its own folder of the bucket, add `fileprefix=[PROJECT_NAME]/` to the command.
```


#### Add Gitlab sibling


::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
datalad create-sibling-gitlab --access ssh -s origin --project [GITLAB_GROUP]/[PROJECT_NAME] --site [GITLAB_SITE] --layout flat
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
datalad create-sibling-gitlab --access ssh -s origin --project gitlab-group/my_datalad_project --site unige --layout flat
```
:::
::::


```{tip}
The `-s origin` flag specifies the name of the sibling.
When users clone the repository, datalad will always set up the remote named to `origin` (whatever the name used during project creation). We recommend to use the name `origin` during project configuration to ensure consistency.
```

```{tip}
If your `python-gitlab` configuration file is not in the default location (`~/.python-gitlab.cfg`), point to it with the `PYTHON_GITLAB_CFG` environment variable before running the command, e.g. `$env:PYTHON_GITLAB_CFG="C:\path\to\python-gitlab.cfg"`.
```


#### Add publish dependency

To ensure that the version control history is always synced to the data, we will set up a publish dependency between the GitLab sibling and the S3 sibling. This means that when you publish to GitLab, the data will also be published to the S3 storage, without the need to publish to the S3 sibling separately.

```powershell
datalad siblings configure -s origin --publish-depends s3
```

### Project use

#### Access control

##### Gitlab

The users must have access to the GitLab repository. You can add new members to the GitLab project by going to the project page on GitLab, clicking on "Settings" `>` "Members", and then adding the user's GitLab username or email address.
`https://[GITLAB_URL]/[GITLAB_GROUP]/[PROJECT_NAME]/project_members`

The user must also add their SSH key to their GitLab account to be able to authenticate when pushing and pulling from the repository. See the [SSH keys](install.md#ssh-keys) section for instructions on how to generate SSH keys and the [Gitlab](install.md#gitlab) section for instructions on how to add the SSH key to GitLab.

##### S3

The user must also have an account on the S3 tenant with read / write permissions on the namespace (see [User creation](#user-creation)), and set their own credentials as environment variables (see [API credentials](#api-credentials)).


#### Clone

::::{tab-set}

:::{tab-item} template
:sync: 0
```{code-block} powershell
datalad clone git@[GITLAB_URL]:[GITLAB_GROUP]/[PROJECT_NAME].git
```
:::
:::{tab-item} example
:sync: 1
```{code-block} powershell
datalad clone git@gitlab.unige.ch:gitlab-group/my_datalad_project.git
```
:::
::::

Data can then be retrieved with `datalad get` and published with `datalad push --to origin`, as long as the S3 credentials are set in the environment.



## Limitations

There are some limitations to this setup that users should be aware of:

- S3 accounts: each user needs an account on the S3 tenant, created by the tenant administrator, and permissions on the namespace. This might lead to some administrative overhead, especially for projects with many users.

- Credentials management: the credentials must be set as environment variables on every machine used to access the data (laptop, {term}`HPC`, ...). Since the secret key is derived from the password, changing the password changes the secret key.

- Object storage: data stored on S3 can only be accessed through the S3 protocol (via git-annex / datalad), not browsed like a regular network drive.
