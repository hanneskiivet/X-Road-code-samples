# X-Road Code Samples

This repository provides X-Road code samples that show how to consume services
through X-Road. The examples show how X-Road compatible clients can be built
using different technologies and they help you to get started quickly. Currently
there are examples available in Java and JavaScript.

You can get started with development and testing without your own Security Server
installation using [Example Adapter](https://github.com/vrk-kpa/xrd4j/tree/master/example-adapter)
service. Another alternative for development/testing is [X-Road Test Service](https://github.com/petkivim/x-road-test-service). All the examples in
this repository use Example Adapter service. Once started the Example Adapter service
can be accessed at http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint where **x.x.x needs to be replaced with the current version number**.

## Prerequisites

Setup [Example Adapter](https://github.com/vrk-kpa/xrd4j/tree/master/example-adapter) service:

* Clone the repository, compile the XRd4J library, compile Example Adapter, start Example Adapter.
OR
* Use Example Adapter Docker [image](https://hub.docker.com/r/niis/example-adapter/).

```
docker run -p 8080:8080 niis/example-adapter
```

## Java

### XRd4J

[XRd4J](https://github.com/vrk-kpa/xrd4j) is a Java library for building X-Road v6 Adapter Servers and clients.

This example demonstrates how to invoke ```helloService``` service with one request parameter.

```
import com.pkrete.xrd4j.client.SOAPClient;
import com.pkrete.xrd4j.client.SOAPClientImpl;
import com.pkrete.xrd4j.client.deserializer.AbstractResponseDeserializer;
import com.pkrete.xrd4j.client.deserializer.ServiceResponseDeserializer;
import com.pkrete.xrd4j.client.serializer.AbstractServiceRequestSerializer;
import com.pkrete.xrd4j.client.serializer.ServiceRequestSerializer;
import com.pkrete.xrd4j.common.member.ConsumerMember;
import com.pkrete.xrd4j.common.member.ProducerMember;
import com.pkrete.xrd4j.common.message.ServiceRequest;
import com.pkrete.xrd4j.common.message.ServiceResponse;
import com.pkrete.xrd4j.common.util.MessageHelper;
import com.pkrete.xrd4j.common.util.SOAPHelper;

import javax.xml.soap.*;
...
...
String url = "http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint";

// Consumer that is calling a service
ConsumerMember consumer = new ConsumerMember("NIIS-TEST", "GOV", "1234567-8", "TestSystem");

// Producer providing the service
ProducerMember producer = new ProducerMember("NIIS-TEST", "GOV", "9876543-1", "DemoService", "helloService", "v1");
producer.setNamespacePrefix("ts");
producer.setNamespaceUrl("http://test.x-road.fi/producer");

// Create a new ServiceRequest object, unique message id is generated by MessageHelper.
// Type of the ServiceRequest is the type of the request data (String in this case)
ServiceRequest<String> request = new ServiceRequest<String>(consumer, producer, MessageHelper.generateId());

// Set username
request.setUserId("jdoe");

// Set message id
request.setId("12345");

// Set request data
request.setRequestData("Test message");

// Application specific class that serializes request data
ServiceRequestSerializer serializer = new HelloServiceRequestSerializer();

// Application specific class that deserializes response data
ServiceResponseDeserializer deserializer = new HelloServiceResponseDeserializer();

// Create a new SOAP client
SOAPClient client = new SOAPClientImpl();

// Send the ServiceRequest, result is returned as ServiceResponse object
ServiceResponse<String, String> serviceResponse = client.send(request, url, serializer, deserializer);

// Print out the SOAP message received as response
logger.info(SOAPHelper.toString(serviceResponse.getSoapMessage()));

// Print out only response data. In this case response data is a String.
logger.info(serviceResponse.getResponseData());

}

public class HelloServiceRequestSerializer extends AbstractServiceRequestSerializer {

  @Override
  protected void serializeRequest(ServiceRequest request, SOAPElement soapRequest, SOAPEnvelope envelope) throws SOAPException {
      SOAPElement data = soapRequest.addChildElement(envelope.createName("name"));
      data.addTextNode((String) request.getRequestData());
  }
}

public class HelloServiceResponseDeserializer extends AbstractResponseDeserializer<String, String> {
  @Override
  protected String deserializeRequestData(Node requestNode) throws SOAPException {
      for (int i = 0; i < requestNode.getChildNodes().getLength(); i++) {
          if (requestNode.getChildNodes().item(i).getLocalName().equals("name")) {
              return requestNode.getChildNodes().item(i).getTextContent();
          }
      }
      return null;
    }

  @Override
  protected String deserializeResponseData(Node responseNode, SOAPMessage message) throws SOAPException {
      for (int i = 0; i < responseNode.getChildNodes().getLength(); i++) {
          if (responseNode.getChildNodes().item(i).getLocalName().equals("message")) {
              return responseNode.getChildNodes().item(i).getTextContent();
          }
      }
      return null;
  }
}
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/xrd4j.

## JavaScript

### node-soap

[node-soap](https://github.com/vpulim/node-soap) is a Node module that lets you
connect to web services using SOAP.

This example demonstrates how to invoke ```getRandom``` service with no request parameters.
Sample client (```index.js```):

```
var soap = require('soap');
var url = 'http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint?wsdl';
var args = {};
soap.createClient(url, function(err, client) {

  client.addSoapHeader({
    "xrd:client": {
      "namespace": "xmlns:tns",
      xRoadInstance: 'NIIS-TEST',
      memberClass: 'GOV',
      memberCode: '1234567-8',
      subsystemCode: 'TestClient',
      attributes: {
        "id:objectType": 'SUBSYSTEM'
      }
    },
    "xrd:service": {
      xRoadInstance: 'NIIS-TEST',
      memberClass: 'GOV',
      memberCode: '9876543-1',
      subsystemCode: 'DemoService',
      serviceCode: 'getRandom',
      serviceVersion: 'v1',
      attributes: {
        "id:objectType": 'SERVICE'
      }
    },
    "xrd:id": 'ID11234',
    "xrd:userId": 'EE1234567890',
    "xrd:protocolVersion": '4.0'
    },
    null, null, null
  );
  client.getRandom(args, function(err, result) {
    console.log(result);
  });
});
```

How to create a simple client:

```
$ mkdir soap
$ cd soap
$ npm init -y
$ npm install soap
$ touch index.js
# Copy the code snippet below in index.js and update the adapter URL (x.x.x)
```

Make sure that the Example Adapter service is running and that the service
URL used in the code is correct. Then run the client:

```
$ node index.js
```

Example Adapter service returns a random number.

```
{ data: '8' }
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/node-soap.
