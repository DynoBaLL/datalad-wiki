```{include} ../links.md
```

# Usage

Once a project is set up (see the setup guides), the day-to-day commands are the same whatever the storage used.

## Clone a project

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

## Get data

Cloning only retrieves the file names and history. Download the content of the files you need with:

```powershell
datalad get [PATH]
```

To free local space, drop the content of files that are safely stored on the remote storage:

```powershell
datalad drop [PATH]
```

## Save changes

```powershell
datalad save -m "describe your changes"
```

## Publish changes

Publish the history to GitLab and the data to the storage in a single command:

```powershell
datalad push --to origin
```

## Get the latest changes

```powershell
datalad update --how merge
```
