# Enabling REST to SOAP
    
This example demonstrates how you can expose a SOAP service over REST using an API in WSO2 Micro Integrator.
    
## Synapse configuration
    
Following is a sample REST Api configuration that we can used to implement this scenario.

In this API configuration we have defined two resources. One is for the HTTP method GET and the other one is for POST. In the first resource, we have defined the uri-template as /view/{symbol} so that request will be dispatched to this resource when you invoke the API using the following URI: `http://127.0.0.1:8290/stockquote/view/IBM`
    
The context of this REST Api is `stockquote`. The SOAP payload required for the SOAP back-end service is constructed using the payload factory mediator defined in the inSequence. The value for the \<m0:symbol\> element is extracted using the following expression:
    
`get-property('uri.var.symbol')`
    
Here ‘symbol’ refers to the variable we defined in the uri-template (/view/{symbol}). Therefore, for the above invocation, the 'uri.var.symbol' property will resolve to the value ‘IBM’.
    
After constructing the SOAP payload, the request will be sent to the SOAP back-end service from the \<send\> mediator, which has an address endpoint defined inline with the format="soap11" attribute in the address element. The response received from the back-end soap service will be sent to the client in plain old XML (POX) format.
    
In the second resource, we have defined the URL mapping as "/order/\*". Since this has POST as the HTTP method, the client has to send a payload to invoke this. 
        
```xml
<api xmlns="http://ws.apache.org/ns/synapse" name="StockQuoteAPI" context="/stockquote">
   <resource uri-template="/view/{symbol}" methods="GET">
      <inSequence>
        <payloadFactory>
          <format>
             <m0:getQuote xmlns:m0="http://services.samples">
                <m0:request>
                   <m0:symbol>$1</m0:symbol>
                </m0:request>
             </m0:getQuote>
           </format>
           <args>
            <arg expression="get-property('uri.var.symbol')"/>
           </args>
        </payloadFactory>
        <header name="Action" value="urn:getQuote"/>
        <send>
          <endpoint>
            <address uri="http://localhost:9000/services/SimpleStockQuoteService" format="soap11"/>
          </endpoint>
        </send>
      </inSequence>
      <outSequence>
        <send/>
      </outSequence>
   </resource>
   <resource url-mapping="/order/*" methods="POST">
      <inSequence>
        <property name="FORCE_SC_ACCEPTED" value="true" scope="axis2"/>
        <property name="OUT_ONLY" value="true"/>
        <header name="Action" value="urn:placeOrder"/>
        <send>
            <endpoint>
                <address uri="http://localhost:9000/services/SimpleStockQuoteService" format="soap11"/>
            </endpoint>
         </send>
      </inSequence>      
   </resource>
</api>
```

## Build and run

Create the artifacts:

1. [Set up WSO2 Integration Studio](../../../../develop/installing-WSO2-Integration-Studio).
2. [Create an ESB Solution project](../../../../develop/creating-projects/#esb-config-project).
3. [Create the rest api](../../../../develop/creating-artifacts/creating-an-api) with the configurations given above.
4. [Deploy the artifacts](../../../../develop/deploy-and-run) in your Micro Integrator.

Set up the back-end service.

Invoke the sample Api by executing the following command:

```bash
curl -v -X GET "http://localhost:8290/stockquote/view/IBM?param1=value1&param2=value2"
```
To execute the sample:

1.  Save the following sample place order request as `placeorder.xml` in your local file system and execute the command. This payload is used to invoke a SOAP service.
    
      ```xml
      <placeOrder xmlns="http://services.samples">
        <order>
           <price>50</price>
           <quantity>10</quantity>
           <symbol>IBM</symbol>
        </order>
      </placeOrder>
      ```
    
2.  Following is the cURL command to execute the first resource:
    
    `curl -v http://127.0.0.1:8290/stockquote/view/IBM`
    
3.  Following is a sample cURL command to invoke the second resource:
    
    `curl -v -d @placeorder.xml -H "Content-type: application/xml" http://127.0.0.1:8290/stockquote/order/`
 
You will see something similar to following line printed in the sample axis2 server (back-end server) console.
    
`Tue Mar 19 09:30:33 IST 2013 samples.services.SimpleStockQuoteService  :: Accepted order #1 for : 10 stocks of IBM at $ 50.0`
    
This SOAP service invocation is an OUT_ONLY invocation, so the ESB profile is not expecting any response back from the SOAP service. Since we have set the FORCE_SC_ACCEPTED property value to true, the Micro Integrator returns a 202 response back to the client.