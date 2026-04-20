# Install

## uv

While datalad can be installed using various methods, we recommend using [uv](https://docs.astral.sh/uv/) for managing the project's dependencies. uv is a modern Python tool that provides a simple and efficient way to manage project dependencies and virtual environments.
To install uv, you can run the following command in your terminal:

::::{tab-set}

:::{tab-item} macOS & Linux
:sync: 0 
```{code-block} bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
:::
:::{tab-item} Windows
:sync: 1
```{code-block} powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
:::
::::


You might need to restart your terminal after installing uv for the changes to take effect.

## Datalad

To install datalad and its dependencies using uv, you can run the following commands in your terminal:

::::{tab-set}

:::{tab-item} macOS & Linux
:sync: 0 
```{code-block} bash
uv tool install datalad
uv tool install datalad-next
uv tool install git-annex
```
:::
:::{tab-item} Windows
:sync: 1
```{code-block} powershell
uv tool install datalad
uv tool install datalad-next
uv tool install git-annex
```
:::
::::


This will install {term}`datalad`, {term}`datalad-next`, and {term}`git-annex` in your environment, allowing you to use them for managing your data and projects effectively.

{term}`git-annex` is a tool that allows you to manage large files with Git without storing the file contents in the Git repository. Datalad relies on git-annex for handling large datasets, so it's important to have it installed as well.

{term}`datalad-next` is a newer version of datalad that includes additional features and improvements. We recommend installing this version for the best experience, especially on Windows, where it provides better support and performance. {term}`datalad-next` must be enabled once installed:

::::{tab-set}

:::{tab-item} macOS & Linux
:sync: 0 
```{code-block} bash
git config --global --add datalad.extensions.load next
```
:::
:::{tab-item} Windows
:sync: 1
```{code-block} powershell
git config --global --add datalad.extensions.load next
```
:::
::::

## SSH keys

To use {term}`datalad` effectively, especially when working with remote repositories and servers such as {term}`gitlab` and remote machines({term}`VM`) it is recommended to set up {term}`ssh` keys for secure and convenient authentication.

::::{tab-set}

:::{tab-item} macOS & Linux
:sync: 0 
```{code-block} bash
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```
:::
:::{tab-item} Windows
:sync: 1
```{code-block} powershell
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```
:::
::::

The new ssh key will be generated and saved in the default location (usually `~/.ssh/id_rsa` for the private key and `~/.ssh/id_rsa.pub` for the public key). 

```{danger}
SSH keys are sensitive and should be kept secure. Do not share your private key with anyone, and consider using a passphrase for added security. Make sure to never commit your private key to any repository or share it in any public space.
```

# Gitlab