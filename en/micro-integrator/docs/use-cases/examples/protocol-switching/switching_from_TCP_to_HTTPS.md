# Switching from TCP to HTTP/S

This example demonstrates how WSO2 Micro Integrator receives SOAP messages over TCP and forwards them over HTTP.

TCP is not an application layer protocol. Hence there are no application level headers available in the requests. The Micro Integrator has to simply read the XML content coming through the socket and dispatch it to the right proxy service based on the information available in the message payload itself. The TCP transport is capable of dispatching requests based on addressing headers or the first element in the SOAP body. In this sample, we will get the sample client to send WS-Addressing headers in the request. Therefore the dispatching will take place based on the addressing header values.

## Synapse configuration

Following are the integration artifacts (proxy service) that we can used to implement this scenario.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<proxy xmlns="http://ws.apache.org/ns/synapse"
       name="StockQuoteProxy"
       transports="tcp"
       startOnLoad="true">
   <target>
      <endpoint>
         <address uri="http://localhost:9000/services/SimpleStockQuoteService"/>
      </endpoint>
      <inSequence>
         <log level="full"/>
         <property name="OUT_ONLY" value="true"/>
      </inSequence>
   </target>
</proxy>
```
## Build and Run

Create the artifacts:

1. [Set up WSO2 Integration Studio](../../../../develop/installing-WSO2-Integration-Studio).
2. [Create an ESB Solution project](../../../../develop/creating-projects/#esb-config-project).
3. Create the [proxy service](../../../../develop/creating-artifacts/creating-a-proxy-service) with the configurations given above.
4. [Deploy the artifacts](../../../../develop/deploy-and-run) in your Micro Integrator.

Set up the back-end service.

* Download the [stockquote_service.jar](
https://github.com/wso2-docs/WSO2_EI/blob/master/Back-End-Service/stockquote_service.jar)

* Run the mock service using the following command
```
$ java -jar stockquote_service.jar
```

[Enable the TCP transport](../../../setup/transport_configurations/configuring-transports/#configuring-the-tcp-transport) and start the Micro-Integrator.

Send the following message via TCP to the TCP listener port.
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header xmlns:wsa="http://www.w3.org/2005/08/addressing">
        <wsa:To>tcp://localhost:6060/services/StockQuoteProxy</wsa:To>
        <wsa:ReplyTo>
            <wsa:Address>http://www.w3.org/2005/08/addressing/none</wsa:Address>
        </wsa:ReplyTo>
        <wsa:MessageID>urn:uuid:464d2e2a-cd47-4c63-a7c6-550c282a1e3c</wsa:MessageID>
        <wsa:Action>urn:placeOrder</wsa:Action>
    </soapenv:Header>
    <soapenv:Body>
        <m0:placeOrder xmlns:m0="http://services.samples">
            <m0:order>
                <m0:price>172.23182849731984</m0:price>
                <m0:quantity>18398</m0:quantity>
                <m0:symbol>IBM</m0:symbol>
            </m0:order>
        </m0:placeOrder>
    </soapenv:Body>
</soapenv:Envelope>
``` 
In linux, we can save the above request in a <strong>request.xml</strong> file and use netcat to send the TCP request. 
```
netcat localhost 6060 < request.xml
```

