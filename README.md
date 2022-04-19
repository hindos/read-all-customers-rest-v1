# read-all-customers-rest-v1
ACE application which reads all customer records from MongoDB collection


## Overview

This message flow GETS a JSON MQ message containing a 'customer-details' payload, and writes the contents of the message into the *customer_details* MongoDB database.

The application is designed to be used in conjunction with the create-customer-mq-soap-to-mq-json-v1 application, which takes in the SOAP request message and converts it to JSON ready for consumption by this flow.

## Message flow nodes used in the flow

The message flow is very simple, comprising of only one message flow node.

![read-all-customers-rest-v1 sub flow implementation](https://github.ibm.com/cpat-agile-integration-sample/read-all-customers-rest-v1/blob/master/readme.images/overall-flow.png "read-all-customers-rest-v1 overall message flow")


### LoopBackRequest


* **Data source name:** customer_updates
* **LoopBack object:** customer_updates
* **Operation:** Retrieve
* **Output data location:** $OutputRoot

The LoopBackRequest node utilises the connection details provided by the datasources.json file in the [ACE Config](https://github.ibm.com/cpat-agile-integration-sample/ace-configurations "datasources.json location") repository. This json file is provided to teh ACE container via a 'generic' ACE Configuration k8s object and loaded into the filesystem of the ACE container.


## 'Retrieve customers' REST service definition

The REST service is very simple. It has one operation: **Retrieve customers**

Please see the swagger file, describing the REST service below:

```
{
  "swagger" : "2.0",
  "info" : {
    "title" : "readallcustomers",
    "version" : "1.0.0",
    "description" : "readallcustomers"
  },
  "paths" : {
    "/customers" : {
      "get" : {
        "operationId" : "getCustomers",
        "responses" : {
          "200" : {
            "description" : "The operation was successful.",
            "schema" : {
              "type" : "string"
            }
          }
        },
        "produces" : [ "application/json" ],
        "description" : "Retrieve customers"
      }
    }
  },
  "basePath" : "/readallcustomers/v1"
}
```

### Endpoint to call on ACE

To call the service you need to issue a **GET** request to the path shown below (you can actually find this from the ACE Dashboard as well, by selkecting the integration server and clicking on the application):

* http://${read_all_customers_http_route}/readallcustomers/v1/customers
* https://${read_all_customers_https_route}/readallcustomers/v1/customers

When deployed as part of the agile integration sample this service is secured using https and mutual authentication is also enabled. You can see an example curl command in the [putTestMessage.sh](https://github.ibm.com/cpat-agile-integration-sample/pipeline-infra/blob/master/admin/run/putTestMessage.sh "putTestMessage script")


## Error Handling

Currently this flow does not have any error handling, but we hope to add this in future.

## pipeline_properties.yaml

Each ACE application in the agile integration scenario has a yaml [file](https://github.ibm.com/cpat-agile-integration-sample/read-all-customers-rest-v1/blob/master/pipeline_properties.yaml "pipeline props yaml) which defines the deployment properties for the application. 

This is used to set things such as keystore and truststore ACE configuration objects, plus to specify if the ACE pod should register for tracing via the CP4I Operations Dashboard. In general, the types of things here are things that the application needs to use when deployed.

```
integrationServer:
  applicationNames:
    - readallcustomers
  releaseName: readallcustomers
  configurations:
    setdbparms:
      - dbparms.txt
    serverconf:
      - serverconf.yaml
    policyproject:
      - DefaultPolicies
    keystore:
      - tiger-bank-ace-server-keystore.jks
    truststore:
      - tiger-bank-ace-server-truststore.jks
    loopbackdatasource:
      - datasources.json
tracing: 
  tracingNS: tracing
  tracingEnabled: "true"
```

**Important note:** each of the properties in the this file must be specified. It is not, for example, a mechanism by which to turn off TLS. TLS must be used, but you can choose here the particulat keystore / truststore ACE configuration object you wish to use. The specification of TLS is controlled by the [serverconf.yaml](https://github.ibm.com/cpat-agile-integration-sample/ace-configurations/blob/master/serverconf.yaml "server conf") created by an Integration Admin and stored in the [ace-configurations](https://github.ibm.com/cpat-agile-integration-sample/read-all-customers-rest-v1 "ace config repo") repo.
