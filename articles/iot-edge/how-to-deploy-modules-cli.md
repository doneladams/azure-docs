---
title: Deploy Azure IoT Edge modules (CLI) | Microsoft Docs 
description: Use the IoT extension for Azure CLI 2.0 to deploy modules to an IoT Edge device
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 06/08/2018
ms.topic: conceptual
ms.reviewer: menchi
ms.service: iot-edge
services: iot-edge
---

# Deploy Azure IoT Edge modules with Azure CLI 2.0

Once you create IoT Edge modules with your business logic, you want to deploy them to your devices to operate at the edge. If you have multiple modules that work together to collect and process data, you can deploy them all at once and declare the routing rules that connect them. 

[Azure CLI 2.0](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) is an open-source cross platform command-line tool for managing Azure resources such as IoT Edge. It enables you to manage Azure IoT Hub resources, device provisioning service instances, and linked-hubs out of the box. The new IoT extension enriches Azure CLI 2.0 with features such as device management and full IoT Edge capability.

This article shows how to create a JSON deployment manifest, then use that file to push the deployment to an IoT Edge device. For information about creating a deployment that targets multiple devices based on their shared tags, see [Deploy and monitor IoT Edge modules at scale](how-to-deploy-monitor-cli.md)

## Prerequisites

* An [IoT hub](../iot-hub/iot-hub-create-using-cli.md) in your Azure subscription. 
* An [IoT Edge device](how-to-register-device-cli.md) with the IoT Edge runtime installed.
* [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) in your environment. At a minimum, your Azure CLI 2.0 version must be 2.0.24 or above. Use `az –-version` to validate. This version supports az extension commands and introduces the Knack command framework. 
* The [IoT extension for Azure CLI 2.0](https://github.com/Azure/azure-iot-cli-extension).

## Configure a deployment manifest

A deployment manifest is a JSON document that describes which modules to deploy, how data flows between the modules, and desired properties of the module twins. For more information about how deployment manifests work and how to create them, see [Understand how IoT Edge modules can be used, configured, and reused](module-composition.md).

To deploy modules using Azure CLI 2.0, save the deployment manifest locally as a .txt file. You will use the file path in the next section when you run the command to apply the configuration to your device. 

Here's a basic deployment manifest with one module as an example:

   ```json
   {
     "moduleContent": {
       "$edgeAgent": {
         "properties.desired": {
           "schemaVersion": "1.0",
           "runtime": {
             "type": "docker",
             "settings": {
               "minDockerVersion": "v1.25",
               "loggingOptions": "",
               "registryCredentials": {
                 "registryName": {
                   "username": "",
                   "password": "",
                   "address": ""
                 }
               }
           },
           "systemModules": {
             "edgeAgent": {
               "type": "docker",
               "settings": {
                 "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                 "createOptions": "{}"
               }
             },
             "edgeHub": {
               "type": "docker",
               "status": "running",
               "restartPolicy": "always",
               "settings": {
                 "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                 "createOptions": "{}"
               }
             }
           },
           "modules": {
             "tempSensor": {
               "version": "1.0",
               "type": "docker",
               "status": "running",
               "restartPolicy": "always",
               "settings": {
                 "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                 "createOptions": "{}"
               }
             }
           }
         }
       },
       "$edgeHub": {
         "properties.desired": {
           "schemaVersion": "1.0",
           "routes": {
               "route": "FROM /* INTO $upstream"
           },
           "storeAndForwardConfiguration": {
             "timeToLiveSecs": 7200
           }
         }
       },
       "tempSensor": {
         "properties.desired": {}
       }
     }
   }
   ```

## Deploy to your device

You deploy modules to your device by applying the deployment manifest that you configured with the module information. 

Use the following command to apply the configuration to an IoT Edge device:

   ```cli
   az iot hub apply-configuration --device-id [device id] --hub-name [hub name] --content [file path]
   ```

The device id parameter is case-sensitive. The content parameter points to the deployment manifest file that you saved. 

## View modules on your device

Once you've deployed modules to your device, you can view all of them with the following command: 

View the modules on your IoT Edge device:
    
   ```cli
   az iot hub module-identity list --device-id [device id] --hub-name [hub name]
   ```

The device id parameter is case-sensitive.

   ![List modules](./media/how-to-deploy-cli/list-modules.png)

## Next steps

Learn how to [Deploy and monitor IoT Edge modules at scale](how-to-deploy-monitor.md)