
# 2. Create a Project

![settings](image.png)

## Pipelines:
![pipelines](image-1.png)


### Agents:
![agents](image-9.png)

### It MUST BE after activation:

![alt text](image-10.png)

### Run Pipelines:

#### Frontend pipelines:  
    Source code must be on 'frontend' folder on git.

![alt text](image-11.png)

## Release Pipelines:
![release pl](image-2.png)

### NET Release Pipeline:
![alt text](image-4.png)
![alt text](image-8.png)

### Python Release Pipeline:
![alt text](image-5.png)
![alt text](image-6.png)

### Frontend Release Pipeline:
![alt text](image-3.png)
![alt text](image-7.png)
![alt text](image-20.png)

#### AzureBlob File Copy deploy

Task 'AzureBlob File Copy' - generate errors for Windows path:

    ##[error]The current operating system is not capable of running this task. That typically means the task was written for Windows only. For example, written for Windows Desktop PowerShell.

#### Azure CLI deploy

Inline Script:
```
az storage blob upload-batch -d '$web' -s ./ --account-name stitmarathonlexxaiprod
```
Working Directory:
```
$(System.DefaultWorkingDirectory)/_Frontend-App-CI/frontend/browser
```
![alt text](image-13.png)


## Result of deploy 

![alt text](image-12.png)
![alt text](image-21.png)


For the frontend application, you can find out the URL by running the following command:

```
$ az storage account show -n stitmarathonlexxaiprod --query "primaryEndpoints.web" --output tsv
https://stitmarathonlexxaiprod.z16.web.core.windows.net/
```


## Debug

### Python App Service
#### Configure alembic init
![alt text](image-22.png)
#### BASH
![alt text](image-16.png)

#### LOG stream
![alt text](image-17.png)

![alt text](image-18.png)

#### API Docs
![alt text](image-19.png)

### Web site:

Main page (https://stitmarathonlexxaiprod.z16.web.core.windows.net):
![alt text](image-14.png)
![alt text](image-28.png)

Login (Home work task):
![alt text](image-15.png)

Test auth Python API:
Login:
![alt text](image-23.png)
![alt text](image-24.png)

Register:
![alt text](image-27.png)
![alt text](image-25.png)
![alt text](image-26.png)