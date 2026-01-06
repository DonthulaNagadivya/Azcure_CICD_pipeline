
# Build and Deploy a Modern YouTube Clone Application in React JS with Material UI 5

Azcure_CICD_pipeline
Build and deploy the applications through pipelines

Steps to set the infrastructure
Login to VSCode
Run the below commands to download the application code
mkdir youtube_clone; cd youtube_clone
git init
git clone https://github.com/DonthulaNagadivya/Azcure_CICD_pipeline.git
Create a project in Azure DevOps and push the code by running the below commands on VSCode:

git remote add origin $YOURAZUREREPO
git push -u origin all
Note: Make sure to update your Azure repo in the above command
We have to create a self host agent if we want to use the selfhostedagent and follow the steps to configure the agent. makesure dependencies should be installed for this project need to install nodejs and npm
su - azureuser
cd /home/azureuser/myagent
./config.sh
sudo ./svc.sh install
sudo ./svc.sh start
sudo apt update
sudo apt install -y nodejs npm
check node -v
npm -v

Go to the Azure Portal and Create the Azure App Service

Implement the build pipeline

Understand the use of service connection and service principal
image
Note: You must set the app settings WEBSITE_DYNAMIC_CACHE=0 and WEBSITE_LOCAL_CACHE_OPTION=Never to disable all file caching
Structure of Azure DevOps build Pipeline
image

A trigger tells a Pipeline to run. It could be CI or Scheduled, manual(if not specified), or after another build finishes.
A pipeline is made up of one or more stages. A pipeline can deploy to one or more environments.
A stage organizes jobs in a pipeline, and each stage can have one or more jobs.
Each job runs on one agent, such as Ubuntu, Windows, macOS, etc. A job can also be agentless.
Each agent runs a job that contains one or more steps.
A step can be a task or script and is the smallest building block of a pipeline.
A task is a pre-packaged script that performs an action, such as invoking a REST API or publishing a build artifact.
An artifact is a collection of files or packages published by a run.
image

Pipeline code used in the PROJECT
trigger: 
- main

stages:
- stage: Build
  jobs:
  - job: Build
    pool:
     name: 'azureagent'
    steps:
    - task: Npm@1
      inputs:
        command: 'install'
    - task: Npm@1
      inputs:
        command: 'custom'
        customCommand: 'run build'

    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: 'build'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: Deploy 
  jobs:
  - job: Deploy
    pool:
     name: 'azureagent'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'drop'
        downloadPath: '$(System.ArtifactsDirectory)'
    - task: AzureRmWebAppDeployment@4
      inputs:
        ConnectionType: 'AzureRM'
        azureSubscription: 'Azure subscription 1($subscription id)'
        appType: 'webAppLinux'
        WebAppName: 'webappyoutube'
        packageForLinux: '$(System.ArtifactsDirectory)/drop'
        RuntimeStack: 'STATICSITE|1.0'
