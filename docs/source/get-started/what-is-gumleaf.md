# How gumleaf work
Purpose of this system is middleware between various systems.  Current only user is a large eCommerce seller, was developed in-house for them, eventual plans to commercialise it, so we try to keep things generic and configurable.

Currently we connect to Shopify (for orders), 2 different shipping/3PL providers (in Australia and Japan), an Inventory system (Unleashed), an SMS provider, bit.ly, Segment, Slack, Treasure Data, AWS, Google Maps
and we move data between them, much like Zapier or OneSaaS.  But the system is more targeting to developers right now, not ready to be used by end-users.

Every "process" is called a "middleware".  Which is one record in the "middleware" table.

Every external system is called an "endpoint", represented by the "endpoint" table.

Every connection to an external system is the "endpointconn" table.

The "middleware" (i.e. process) table links 2 "endpointconn" records together
So you are linking Shopify to Machship, for instance.
every endpointconn has some parameters (in endpointconnparam).  And there are also parameters in endpointparam which all endpointconns can see
every endpointconn record has a "conn_type" field and a "handler" field.  The "conn_type" is going to be a "Reader" or "Writer" (depending on whether this endpointconn is the SOURCE or DESTINATION), and the "handler" will be an "InputHandler" or "OutputHandler" (again, depending on whether this endpointconn is the SOURCE or DESTINATION)
InputReader, InputHandler, OutputHandler, and OutputWriter are all interfaces which are implemented by many classes
the basic concept of the system is

1: look at middleware record, get endpointconn SOURCE record

2: get "conn_type" (InputReader) from SOURCE endpointconn

3: get a dataset from that InputReader.  could be JSON Orders from Shopify, XML files via FTP, anything.  There is a different Reader for each kind of "retrieval" task

4: get "handler" (InputHandler) from SOURCE endpointconn

5: pass each "item" in that dataset to the InputHandler, which creates a standardised DataItem object in our internal format (DataItem is another core interface)

6: get "handler" (OutputHandler) from DESTINATION endpointconn

7: now take each DataItem, and pass it into OutputHandler for modification and processing, application of business rules, etc

8: get "conn_type" (OutputWriter) from DESTINATION endpointconn

9: pass each DataItem to the OutputWriter, so that some kind of output message can be sent to the DESTINATION system

and there can also be "intermediate" connections, which are endpointconn records which only have the "handler" field defined, and this is an OutputHandler.  So really, between points 5 and 6, there is

5.1 for each middlewareintermediateconn record linked to the middleware record, get the endpointconn.handler field (OutputHandler)
5.2 pass DataItem to that OutputHandler
so a good one to look at is the middleware named "Shopify -> NetSuite - create order".  There are several intermediate connections, and if you step through ConsoleController in a debugger, stepping INTO the key method calls of 

getInputData() << InputReader
processData()  << InputHandler and OutputHandler
sendOutput()  << OutputWriter