# Sample Bicep+Blazor+SWA

## How this was created

1. Initialize the [Bicep starter](https://github.com/azure-samples/azd-starter-bicep).

    ```shell
    azd init --template azd-starter-bicep
    ```
 
1. Run these scripts:

    ```shell
    dotnet new sln --name AZD.Example --output src
    dotnet new gitignore --output src
    dotnet new blazorwasm --name AZD.Example.Blazor --output src/web
    dotnet sln src/AZD.Example.sln add src/web/AZD.Example.Blazor.csproj
    ```

1. Optionally, test the web app:

    ```shell
    dotnet watch run --project src/web/AZD.Example.Blazor.csproj
    ```

1. Replace **./azure.yaml** with this content:

    ```yaml
    # yaml-language-server: $schema=https://raw.githubusercontent.com/Azure/azure-dev/main/schemas/v1.0/azure.yaml.json

    name: azd-starter
    workflows:
    up: 
        steps:
        - azd: provision
        - azd: deploy
    services:
    web:
        project: ./src/web
        language: csharp
        host: staticwebapp
        dist: wwwroot
        hooks:
        predeploy:
            posix:
            shell: sh
            run: dotnet publish -c Release --output .
            windows:
            shell: pwsh
            run: dotnet publish -c Release --output .
    ```

1. Add this resource to the **Add resources** section of **./infra/main.bicep**:

    ```bicep
    module web 'core/host/staticwebapp.bicep' = {
      name: 'static-web-app'
      scope: rg
      params: {
        name: '${abbrs.webStaticSites}-${resourceToken}'
        location: location
        tags: union(
          tags,
          {
            'azd-service-name': 'web'
          }
        )
      }
    }
    ```

1. Run `azd up --debug`.