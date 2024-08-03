# Useful CI/CD templates to deploy Next.js and .NET Core applications to Azure App Services from Azure DevOps pipelines and Github actions

## Overview

Searching for a suitable CI/CD template to automate application deployments to Azure App Services can be a challenging task. 

I collected a few useful compact templates that I use to establish CI/CD automations for deploying my Next.js and .NET Core applications to Azure App Services.

Folder structures:

- **nextjs**
  - .devops
    - azure-pipelines.yml
  - .github
    - workflows
      - main.yml

- **dotnetcore**
  - .devops
    - azure-pipelines.yml
  - .github
    - workflows
      - main.yml

## Getting started

### Azure DevOps pipelines

1. Copy a corresponding **.devops** folder with all subfolders and files into the root of your project.

2. Review and adjust variables found in the headers of **azure-pipelines.yml** files to match with your Azure environment.

   <pre>
   appServiceName: 'your-app-service-name' # Like in https://[your-app-service-name].azurewebsites.net
   azureSubscription: 'Azure RM RG Service Principal' # Project settings > Service connections > Azure Resource Manager + Subscription + Resource Group
   nodeVersion: '20.x' # Adjust according to your Node.js version
   runtimeStack: 'NODE|20-lts'
   </pre>
   
   If you need to hide any of those values, just use your DevOps Project > Pipelines > Library > Variable groups
     - Create a new group, for instance, **nextjs**.
     - Refer to the added variable using the following notation in **azure-pipelines.yml**: $(variable-name).
      
     <pre>
       variables:
       - group: nextjs # Then you can refer to the group's hidden variable in the place of its consumption as $(name-of-your-hidden-variable)
       - name: openVariable
         value: openVariableValue
     </pre>
     
3. Managing secure files.

   **.env** file is used to manage application settings in Next.js.
   - Typically, this file is excluded from the source control using the entry in .gitignore
   - When you build your project, Next.js uses entry names from a local (private) copy of .env file to generate static references in the target JS-code.
   - In many cases, if .env file is unavailable during the build time, it may create a tricky problem:
     Next.js just replaces empty entries with "undefined" values instead of expected empty strings "".

   Here, you have two simple alternatives to address the problem. 
   - Save your .env file into Azure DevOps project  > Pipelines > Library > Secure files > .env
     and use it in your pipeline to dynamically create entire content of .env file just before the build operation.
  
     OR
  
   - Use the dummy file .env with empty values like key1= during the build time.
     For example, create .env.dummy and just rename it in the pipeline to .env just before the build operation.
     Then you can use Environment variables added to your Azure App Service to load actual values dynamically by their key names at the runtime.

4. Create your Azure DevOps pipeline using the modified **azure-pipelines.yml**
   - Alternatively, create an ampty pipeline and modify its settings to refer to azure-pipelines.yml

5. Automatic execution on commit.
   - By default, pipeline execution is triggered manually.
   - You can enable it to trigger on commit as shown below.
  
   <pre>
   trigger:  
    branches:
      include:
        - none  # To be triggered manually
        #- main # To be triggered automatically on each commit to main branch      
   </pre>

   replace with
   
   <pre>
   trigger:  
     branches:
       include:
       - main
   </pre>
  
### Github actions

1. Copy corresponding **.github** folder with all subfolders and files into the root of your project.

2. Create repository secrets in Settings > Security > Secrets and variables > Actions
   - dotnetcore:
   - **AZURE_APP_SERVICE_NAME**
     - Save [your-app-service-name] value just like in https://[your-app-service-name].azurewebsites.net
   - **AZURE_APP_SERVICE_PUBLISH_PROFILE**
     - Download ProfileSettings.xml from your Azure App Service > Overview blade.
     - Save its entire XML content into **AZURE_APP_SERVICE_PUBLISH_PROFILE**

- nextjs: **AZURE_APP_SERVICE_NAME**,  **AZURE_APP_SERVICE_PUBLISH_PROFILE**, and optional **ENV_FILE_CONTENT**
  - Repeat two steps from dotnetcore (above)
  - Add the optional repository secret **ENV_FILE_CONTENT**
    - Save content of your .env file (if required)

3. Review and adjust .github/workflows/main.yml
   - By default, workflow execution is triggered manually.
   - You can enable it to trigger on commit as shown below.
   <pre>
   push:
     branches:
       # - main
       - none
   </pre>
   
   replace with
   
   <pre>
   push:
     branches:
       - main
       #- none
   </pre>
