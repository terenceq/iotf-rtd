===============================================================================
MQTT Connectivity for Applications
===============================================================================

An application must authenticate using a client ID in the following format:

	a:**org\_id**:**app_id**

- We do not impose any rules on the **app\_id** component of the client ID
- When connecting to the Quickstart service no authentication is required
- An application does not need to be registered before it can connect


----


MQTT client identifier
-------------------------------------------------------------------------------

-  Supply a client id of the form
   **a**:**org\_id**:**app\_id**
-  **a** indicates the client is an application
-  **org\_id** is your unique organization ID, assigned when you sign up
   with the service.  It will be a 6 character alphanumeric string.
-  **app\_id** is a user-defined unique string identifier for this client.

.. note:: Only one MQTT client can connect using any given client ID.  As soon 
    as a second client in your organization connects using an **app\_id** that you 
    have already connected the first client will be disconnected.


----


MQTT authentication
-------------------------------------------------------------------------------

Applications require an API Key to connect into an organization.  When an API Key 
is registered a token will be generated that must be used with that API key.  

The API key will look something like this: a-**org\_id**-a84ps90Ajs

The token will look something like this: MP$08VKz!8rXwnR-Q*

When making an MQTT connection using an API key the following applies:

- MQTT client ID: a:**org\id**:**app\_id**
- MQTT username must be the API key: a-**org\_id**-a84ps90Ajs
- MQTT password must be the authentication token: MP$08VKz!8rXwnR-Q*


----


Publishing device events
-------------------------------------------------------------------------------
An application can publish events as if they came from any registered device.

-  Publish to topic iot-2/type/**device\_type**/id/**device\_id**/evt/**event\_id**/fmt/**format\_string**

.. tip:: You may have a number of devices that are already generating bespoke data
    that you wish to send to IoTF.  One way to get that data into the service would
    be to write an application that processes the data and publishes it to IoTF.

----


Publishing device commands
-------------------------------------------------------------------------------
An application can publish a command to any registered device.

-  Publish to topic iot-2/type/**device\_type**/id/**device\_id**/cmd/**command\_id**/fmt/**format\_string**

----


Subscribing to device events
-------------------------------------------------------------------------------
An application can subscribe to events from one or more devices.

-  Subscribe to topic iot-2/type/**device\_type**/id/**device\_id**/evt/**event\_id**/fmt/**format\_string**

.. note:: The MQTT "any" wildcard character (+) may be used for any of the following 
    components if you want to subscribe to more than one type of event, or events 
    from more than a single device.

    - device\_type
    - device\_id
    - event\_id
    - format\_string


----


Subscribing to device commands
-------------------------------------------------------------------------------
An application can subscribe to commands being sent to one or more devices.

-  Subscribe to topic iot-2/type/**device\_type**/id/**device\_id**/cmd/**command\_id**/fmt/**format\_string**

.. note:: The MQTT "any" wildcard character (+) may be used for any of the following 
    components if you want to subscribe to more than one type of event, or events 
    from more than a single device.

    - device\_type
    - device\_id
    - cmd\_id
    - format\_string


----

	
Subscribing to device status messages
-------------------------------------------------------------------------------
An application can subscribe to monitor status of one or more devices.

-  Subscribe to topic iot-2/type/**device\_type**/id/**device\_id**/mon

.. note:: The MQTT "any" wildcard character (+) may be used for any of the following 
    components if you want to subscribe to updates from more than one device.

    - device\_type
    - device\_id


----


Subscribing to application status messages
-------------------------------------------------------------------------------
An application can subscribe to monitor status of one or more applications.

-  Subscribe to topic iot-2/app/**app\_id**/mon

.. note:: The MQTT "any" wildcard character (+) may be used for **app\_id** if you 
    want to subscribe for updates for all applications.


----


Quickstart restrictions
-------------------------------------------------------------------------------

If you are writing application code that wants to support use with Quickstart
you must take into account the following features present in the
registered service that are not supported in Quickstart: 

- Publishing commands
- Subscribing to commands
- Use of the MQTT "any" wildcard character (+) for the following topic components:

  - device\_type
  - app\_id
- MQTT connection over SSL


----


Scalable Applications
-------------------------------------------------------------------------------

You can build scalable applications which will load balance messages across 
multiple instances of your application by making a few changes to how your 
application connects to the IoT Foundation. Applications taking advantage
of this feature must only attempt to make non-durable subscriptions. A bit
of experimentation may be needed to understand how many clients are needed
for the optimum balance in load.

-  Supply a client id of the form
   **A**:**org\_id**:**app\_id**
-  **A** indicates the client is a scalable application
-  **org\_id** is your unique organization ID, assigned when you sign up
   with the service.  It will be a 6 character alphanumeric string.
-  **app\_id** is a user-defined unique string identifier for this client.
-  Create a non-durable subscription 

.. note:: Only non-durable subscriptions are supported for scalable applications. 
    Please note that the client id must begin with a capital 'A' in order to designated
    as a scalable application by IoTF. Multiple clients that are part of the scalable
    application should use the exact same client id.

----


How It Works
~~~~~~~~~~~~
The IoTF service extends the MQTT 3.1.1 specification to provide support for shared subscriptions. 
Shared subscription can provide simple load balancing functionality for applications. A shared 
subscription might be needed if a back-end enterprise application can not keep up with the number 
of messages being published to a specific topic space. For example if many devices were publishing 
messages that are being processed by a single application. It might be helpful to leverage the load 
balancing capability of a shared subscription. IoTF shared subscription support is limited to 
non-durable subscriptions only.  

A simple example of an auto-scaling application:

-  client 1 connects as A:abc123:myApplication and subscribes to all device events
   client 1 will receive 100% of the device events published
-  client 2 connects as A:abc123:myApplication and subscribes to all device events
   now, client 1 and client 2 will share all of the events published between them. that is
   the load is now shared between client 1 and client 2.
-  client 3 connects as A:abc123:myApplication and subscribes to all device events
   now, instance 1, 2 and 3 will process the events shared amongst all three instances
-  clients 2 and 3 unsubscribe from all device events now, although instance 2 and 3 are 
   still connected to the service, instance 1 will be receiving  all device events published
