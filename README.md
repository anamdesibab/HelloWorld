# MQ Environment Setup

The following utilities will generate .mqsc files for each POET environment as seen on the [gapMQSeries Prod](https://github.gapinc.com/eis/gapMQSeriesMQSC_PROD) repository and [gapMQSeries Test](https://github.gapinc.com/eis/gapMQSeriesMQSC_TEST) repository. This is also where any changes should be committed. 

## Pre-requisites 

### mq-environment-config.json
This file contains all environment specific configurations. To add a new environment or change an existing one, simply modify this file.

### build_mq_environments.groovy
This file has all the required queues for the POET. To add a new queue, simply add a MQ object to build_mq_environments.groovy, see [MqsFileBuilder.groovy](MqsFileBuilder.groovy) for available methods. 

### MqsFileBuilder.groovy
The following are methods that can be used in build_mq_environments.groovy

   
  ##### header(String poetQueueManager)
  Adds a header to the file with the current date/time and the given queue manager name

  ##### localQueueManager(String name)
  Configures the local queue manager with the given name and preset settings.

  ##### remoteQueueManager(String name, String poetQueueManager, String connectionName)
  Sets up all required configuraton to connect to a remote queue manager with the given name. Also requires the local poetQueueManager name and a connectionName. This method adds both sender and receiver channels between the named remote queue manager and the local poetQueueManager. Additionally, the local transmit queue is also created.

  ##### queueModel(String name[, Map options])
  Adds a queue model to the configuration with preset settings, specific settings can be overridden by passing an options map.

  ##### queueLocal(String nodeId, String name[, Map options])
  Sets up a local queue and its corresponding error queue, nodeId and queue name are required. The name prefix (QC, QL) will automatically be determined and should not be included on the name. Additional settings can be included by passing an options map.

  ##### queueLocal(String name[, Map options])
  Creates a non dc-specific local queue. Different from above method in that:
    <ul>
      <li>nodeId is not required</li>
      <li>full queue name including prefix is expected</li>
      <li>a corresponding error queue is NOT created</li>
    </ul>


  ##### queueRemote(String nodeId, String name[, Map options])
  Sets up a remote queue, nodeId and queue name are required. The name prefix (QR) will automatically be added and should not be included on the name. Additional settings can be included by passing an options map.

  ##### queueRemote(String name[, Map options])
  Creates a non dc-specific remote queue. Different from above method in that:
    <ul>
      <li>nodeId is not required</li>
      <li>full queue name including prefix is expected</li>
    </ul>


  ##### queueAlias(String nodeId, String name)
  Creates an alias queue, nodeId and name are required. The name prefix (QA) will automatically be added and should not be included on the name.

  ##### channel(String name, String type[, Map options])
  Defines a channel with the given name and type. Preset settings can be overridden by passing an options map.

  ##### listener(String name, String port)
  Defines a listener with the given name on the given port.

  ##### systemBrokerService(String poetQueueManager)
  Defines the system broker service for the given local poetQueueManager

  ##### systemDefaults()
  This is somewhat of a catch-all method to create boilerplate configurations. Any changes to this should be made directly in MqsFileBuilder.groovy.

### compare_mqs_files.py
A simple utility to help with mqs file comparison. More than just a diff, will compare all contents of each file (independent of queue definition order) and output any differences.

    Example usage:

    `>python compare_mqs_files.py [options] <input-file-a> <input-file-b>`

    options:

    <dl>
      ##### --detailed
      includes detailed diff of mq definition options
    </dl>

## Build/Generate 
* Validate all the environment information in the mq-environment-config.json file.
* Review/update MQ objects in build_mq_environments.groovy file. 
* Run `groovy build_mq_environments.groovy`, .mqsc files will be generated in the mqsc directory of the above gapMQSeries repository fork.
* Review generated .mqsc files
* Optionally generated files can be compared using compare_mqs_files.py utility

## Deploy
* For local envirionments rebuild local Docker with new .mqsc file

  `>rebuild-container.sh` 
* For all other environments :
  * Remove all the already existing MQ objects from generated .mqsc file to make sure to have only delta MQ objects
  * Create infra story and attach update file with delta changes.
  * Assign it to MQ team for deployment.

