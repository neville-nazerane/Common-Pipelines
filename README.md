# Introduction

Reusable YAML for azure devops. To use these you can fork this repository and use the following on your YAML code to reference the template file:

```yaml
resources:
  repositories:
    - repository: templates
      type: github
      name: <username>/Common-Pipelines
      endpoint: <your endpoint setup from devops>
```

# Currently available templates:

## netcore-ubuntu

This is meant to deploy a .NET Core web app (excluding blazor WASM) to any Ubuntu VM that is setup with Nginx.

1. Make sure you have followed this official documentaiton to setup Nginx http://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-3.1. You can skip the parts to setup the Linux service (covered under the **Monitor the app**
 section) using the .service file and to upload the binaries, since this template would take care of those. 

2. As of now, this template assumes you have a virtual machine called "websites" under an environment called "Development". However, this is soon to change. For now. this can be setup by going under Pipelines > Environments in azure devops. Select "New Environment". Enter the name as "Development" and choose the *Resource* as a linux virtual machine. Follow the instructions and copy-paste the command line arguement provided in your linux VM. Make sure the VM is named "websites". 

3. Once the above is setup, you can use a YAML similar to the below. Make sure you replace everything within the <> including the those symbols

```yaml
pool:
  vmImage: 'ubuntu-latest'

resources:
  repositories:
    - repository: templates
      type: github
      name: <username>/Common-Pipelines
      endpoint: <your endpoint setup from devops>


extends:
  template: Shared\netcore-ubuntu.yml@templates
  parameters:
    usePreRelease: <true if you want to use .net 5 prereleases>
    projectPath: <path to the web app to deploy>
    projectName: <a unique project name>
    destDirectory: <destination directory where code is deployed>
    port: <port specified in NGINX for the webapp. default is 5000>

    service:
      description: <description for your .service file>
      identifier: <unique identifier for your .service file>
      name: <name for you .service file>
    configs:
     # add all configurations for your web app. Below are two examples
     # the second is how secrets such as connection strings can be added. 
     # read this for more information on secret variables: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#secret-variables
      ASPNETCORE_ENVIRONMENT: Production
      connectionStrings__db: $(Secrets.Db)
    preBuildSteps:

    # following are optional pre-conditions before the deployment process. 
    # this can be used to run tests. below is an example
      - script: dotnet test myproject.Tests
        displayName: Unit Tests
```





