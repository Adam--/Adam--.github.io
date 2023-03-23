# Integrating Pipenv and Azure Private Package Indexes

Pipenv is a popular tool for automatically creating and managing Python virtual environments and installing and managing pip packages.

For more information on pipenv, the problems it solves and its benefits checkout the first page of the pipenv docs: [Pipenv: Python Dev Workflow for Humans](https://pipenv-fork.readthedocs.io/en/latest/index.html).

Unfortunately, using Azure private package indexes present some technical challenges regarding authentication. Specifically, Azure uses a web based authentication workflow that needs some addition configuration to work with pipenv.

It is assumed that you have a python and pip installed and added to your shell’s path. It's generally recommended to pip install to the user site using --user; however, I found that these user directories are typically not added to the path.

## Install the tools

pipenv, artifacts-keyring, and azure-devops-artifacts-helpers will need to be installed in the same site for the base interpreter. If you run the command as shown below this will satisfy that requirement.

```
pip install pipenv artifacts-keyring azure-devops-artifacts-helpers
```

- [Pipenv](https://pypi.org/project/pipenv/) is the main tool.
- [artifacts-keyring](https://pypi.org/project/artifacts-keyring/) provides authentication for consuming Python packages to or from Azure Artifacts feeds within Azure DevOps.
- [azure-devops-artifacts-helpers](azure-devops-artifacts-helpers) is a helper tool that will bootstrap virtual environments with artifacts-keyring allowing them to authenticate to Azure Artifacts feeds.

## Configure the virtual environment seeder

An environmental variable is used with azure-devops-artifacts-helpers to seed the virtual environment with the requirements for the azure private package index. Add this environmental variable with the following command.

```
setx VIRTUALENV_SEEDER azdo-pip
```

Shells need to be restarted for environmental variables to be picked up; however, this is not necessary if you do not plan on using pipenv directly from this shell.

## Authenticate to the private index

From your shell, run the following command. This will attempt to install hil-test-rig from our private package index which will trigger authentication and save it to your local keyring. Follow the prompts to sign into https://microsoft.com/devicelogin with the code. This should be the only time you have to login regardless of the project or virtual environment you are in.

```
pip install hil-test-rig --index="https://pkgs.dev.azure.com/lakeshorecryotronics/_packaging/LakeShoreInternalPythonPackages/pypi/simple/"
```

Remove the temporarily installed hil-test-rig package. You don't need it in your base environment.

```
pip uinstall hil-test-rig
```

## Configure PyCharm

When opening a project for the first time with a Pipfile, PyCharm will automatically configure a virtual environment using pipenv. If this doesn't happen, it's possible that the project has been previously configured for a different interpreter. The easiest way to fix this is to close the project, delete the .idea folder, and reopen the project. Full instructions for configuring a pipenv environment can be found at [Configure a Pipenv environment - PyCharm Documentation](https://www.jetbrains.com/help/pycharm/pipenv.html)

## Configure Visual Studio Code

VS Code should automatically detect and activate virtual environments, see [Using Python Environments in Visual Studio Code - How the extension looks for environments](https://code.visualstudio.com/docs/python/environments#_how-the-extension-looks-for-environments). Unlike PyCharm, it doesn't manage them, and it is expected that you understand pipenv and use the terminal to use pipenv.

## Using Pipenv directly

Pipenv can be used directly from the command line. The following commands are useful:

- `pipenv install` will install the packages listed in the Pipfile
- `pipenv install <package>` will install the specified package and update the Pipfile
- `pipenv shell` will activate the virtual environment

See full documentation at [Pipenv: Python Dev Workflow for Humans — Pipenv Documentation](https://pipenv.pypa.io/en/latest/)

## Example pipfile

Note that disable_pip_input needs to be set to false to use keychain authentication. See [Advanced Usage of Pipenv — Injecting credentials through keychain support)](https://pipenv.pypa.io/en/latest/advanced/#injecting-credentials-through-keychain-support)


```
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[[source]]
url = "https://pkgs.dev.azure.com/ORGANIZATION/_packaging/FEED/pypi/simple/"
verify_ssl = true
name = "pkgsdevazure"

[packages]
PACKAGE-NAME = {version = "*", index = "pkgsdevazure"}

[dev-packages]

[requires]
python_version = "3.11"

[pipenv]
disable_pip_input = false
```
