# How to manage GitHub workflows on WSL
> See more: [Ubuntu blog | How we improved testing Ubuntu on WSL – and how you can too!](https://ubuntu.com/blog/improved-testing-ubuntu-wsl)


Most of the time, what works on Ubuntu desktop works on WSL as well. However, there are some exceptions. Furthermore, you may want to test software that lives both on Windows and inside WSL. In these cases, you may want to run your automated testing on a Windows machine with WSL rather than a regular ubuntu machine.

There exist Windows GitHub runners, but they do not support the latest version of WSL. The reason is that WSL is now a Microsoft Store application, which requires a logged-in user. GitHub runners, however, run as a service. This means that they are not on a user session, hence they cannot run WSL (or any other store application).
> See more: [Microsoft Dev blogs | What’s new in the Store version of WSL?](https://devblogs.microsoft.com/commandline/the-windows-subsystem-for-linux-in-the-microsoft-store-is-now-generally-available-on-windows-10-and-11/)

We propose you run your automated tests on a Windows virtual machine hosted on Azure. This machine will run the GitHub actions runner not as a service, but as a command-line application.

## Prerequisites
You'll need to have a machine that can host the actions runner. This guide shows how to accomplish this with a Microsoft Azure Virtual Machine.
To create a Windows 11 VM on Azure follow Azure's instructions, no special customization is necessary. Take note of the _resource group_ and _machine name_ that you use.
```{note}
You can self-host or use any other hosting service. This guide uses Azure as an example.
```

## 1. Set up the host
1.	Log in to your host. You can easily use a remote desktop client with Azure.
1.	Install WSL with `wsl --install`.
1.	Enable automatic logon
	> See more: [Microsoft Learn | Turn on automatic logon in Windows](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon).
1.	Install and configure the GitHub actions runner
	1. Head to GitHub, navigate to your repository settings and click _Actions_ > _Runners_ > _New self-hosted runner_ > _Windows_.
	2. Follow the instructions to install the runner in your VM. Make sure you do not enable running it as a service.
	> See more: [GitHub Docs | Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)
1.	Set up your runner as a startup application:
	1. Go to the directory you installed the GitHub runner in.
	2. Right-click on the `run.cmd` file, and click _Show more options_ > _Send to_ > _Desktop (create shortcut)_.
	3. Press Win+R, type `shell:startup` and press **OK**. A directory will open.
	4. Find the shortcut on the desktop and drag it to the startup directory.

## 2. Set up your repository
```{note}
This is for Azure VMs, the setup will be different if you use a different hosting service.
```
1. Obtain your Azure credentials.
   > See more: [Azure/login action | Login With OpenID Connect](https://github.com/Azure/login/blob/master/README.md#login-with-openid-connect-oidc-recommended).
2. Head to GitHub, navigate to your repository settings, and click _Secrets and variables_ > _Actions_ > _New repository secret_. Fill the form with the following information:
 	- Name: Any name you like. In our workflows we use `AZURE_VM_CREDS`.
 	- Secret: Copy-paste the Azure credentials.

## 3. Create a GitHub Actions workflow
1.	Add a `concurrency` directive to prevent different workflows from running at the same time.
	> See more: [GitHub Docs | Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency).


2. You can have your host running permanently, but that can get expensive and wasteful. To avoid this, create a workflow with the following jobs:
   	1. Start up the VM: 
		- Configure it so that it runs on a GitHub-hosted runner.
		- This step must power on your host machine. If you use Azure, use the [Azure/login action](https://github.com/Azure/login) plus the following command:
		  ```
		  az vm start --name ${AZ_MACHINE_NAME} --resource-group ${AZ_RESOURCE_GROUP}
		  ```
	1. Run your job:
      	- Configure it so that it runs on a `self-hosted` runner (your machine).
      	- Set a dependency on the previous step.
      	- We developed some actions to help you build your workflow.
      	  > See also: [WSL GitHub actions reference](reference::actions).
	2. Stop the VM:
      	- Configure it so that it runs on a GitHub-hosted runner.
		- Set a dependency on the previous step.
		- Set it up so that it always runs, even on failure or cancellation. See more: [GitHub Docs | Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions#always).
      	- This step must power off your host machine. If you use Azure, use the [Azure/login action](https://github.com/Azure/login) plus the following command:
		  ```
		  az vm deallocate --name ${AZ_MACHINE_NAME} --resource-group ${AZ_RESOURCE_GROUP}
		  ```
<!-- Next time you push to your repository, bla bla bla -->

## Example repositories
The following repositories use some variation of the workflow explained here.
- [Ubuntu/WSL example workflow](https://github.com/ubuntu/WSL/blob/main/.github/workflows/wsl-example.yaml)
- [Ubuntu/WSL-example hello world example](https://github.com/ubuntu/wsl-actions-example/blob/main/.github/workflows/test_wsl.yaml)
- [Ubuntu/WSL end-to-end tests](https://github.com/ubuntu/WSL/blob/main/.github/workflows/e2e.yaml)
