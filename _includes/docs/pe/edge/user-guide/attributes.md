
* TOC
{:toc}

## Assigning custom attributes to entities and attributes managing

ThingsBoard **PE Edge** provides the ability to assign custom attributes to your [entities](/docs/user-guide/entities-and-relations/#entities-overview) and manage these attributes effectively. 

These attributes are stored in a database and can be used for data visualization and data processing. Attributes are treated as key-value pairs. 

For example:

```json
{
 "firmwareVersion":"v2.3.1", 
 "booleanParameter":true, 
 "doubleParameter":42.2, 
 "longParameter":73, 
 "configuration": {
    "someNumber": 42,
    "someArray": [1,2,3],
    "someNestedObject": {"key": "value"}
 }
}
```
{: .copy-code}

The flexibility and simplicity of this format facilitate easy and seamless integration with nearly any IoT device on the market. The key is always a string that represents the attribute name, while the attribute value can be a string, boolean, double, integer, or JSON object.


## Attributes naming convention

As a platform user, you can define attribute names using any format. However, we strongly recommended to follow [camelCase](https://en.wikipedia.org/wiki/Camel_case) style convention. This makes clarity when writing custom JavaScript or TBEL (ThingsBoard Expression Language) functions for data processing and visualization.

## Attribute types

ThingsBoard **PE Edge** platform supports three primary types of attributes that help in managing a device's configuration. These attributes are categorized into:

### 1. Server-side attributes

This type of attributes is supported by almost any platform entity such as **Device**, **Asset**, **Customer**, **Tenant**, **User**, etc.

Server-side attributes are the ones that you may configure via Administration UI or REST API.

{% capture delete_restrictions %}
**NOTE:** The device firmware can't access the server-side attribute.
{% endcapture %}
{% include templates/info-banner.md content=delete_restrictions %}

{:refdef: style="text-align: center;"}
![image](/images/user-guide/server-side-attributes.svg)
{: refdef}

Consider the scenario of developing a monitoring system we defined attributes as outlined in the table below:

| Attribute Name   | Data Type | Description                                                                 | Example Value                               | Comments                                                                         |
|------------------|-----------|-----------------------------------------------------------------------------|---------------------------------------------|----------------------------------------------------------------------------------|
| 'latitude'      | Double    | Latitude coordinate for geolocation.                                        | 40.7128                                     | Good to assign for assets that represent building or other real estate.          |
| 'longitude'      | Double     | Longitude coordinate for geolocation.                                       | -74.0060                                    | Good to assign for assets that represent building or other real estate.          |
| 'address'        | String    | Physical address of the asset.                                              | "123 Example St, New York, NY 10007"        | Map Widget (to visualize location)                                               |
| 'floorPlanImage' | URL       | URL link to an image of the building's floor plan.                          | "https://example.com/floorplan.jpg" | May contain a floor plan image URL link.                                         |
| 'maxTemperatureThreshold' | Integer   | Maximum temperature limit before an alert is triggered.                     | 75                                          | Used in monitoring/alert systems                                                 |
| 'temperatureAlarmEnabled' | Boolean   | Boolean flag to enable or disable temperature alarms.                       | true                                        | May be used to configure and enable/disable alarms for a certain device or asset |

#### Administration UI

To add attributes to the device via Administration UI you may follow these steps below:

{% include images-gallery.html imageCollection="server-side-attrs-pe-edge-ui" showListImageTitles="true" %}


{% capture bulk_provisioning %}
[Bulk provisioning](/docs/{{docsPrefix}}user-guide/bulk-provisioning/) feature allows you to quickly create multiple devices and assets and their attributes from a CSV file.
{% endcapture %}
{% include templates/info-banner.md content=bulk_provisioning %}

#### REST API

Use [REST API](/docs/{{docsPrefix}}reference/rest-api/) documentation to get the value of the JWT token. You will use it to populate the 'X-Authorization' header and authenticate your REST API call request.

Send a POST request with JSON representation of the attribute to the following URL: 

```text
https://$YOUR_THINGSBOARD_HOST/api/plugins/telemetry/$ENTITY_TYPE/$ENTITY_ID/SERVER_SCOPE
```

The example below creates an attribute with the name "**newAttributeName**" and value "**newAttributeValue**" for a device with **"ID"**: 'bbd53320-75a5-11ef-8424-bdf021575205' and ThingsBoard PE Edge:
```shell
curl -v 'https://thingsboard.cloud/api/plugins/telemetry/DEVICE/bbd53320-75a5-11ef-8424-bdf021575205/SERVER_SCOPE' \
-H 'x-authorization: Bearer $YOUR_JWT_TOKEN_HERE' \
-H 'content-type: application/json' \
--data-raw '{"newAttributeName":"newAttributeValue"}'
```
{: .copy-code}

Similarly, you can fetch all server-side attributes using the following command:

```shell
curl -v -X GET 'https://thingsboard.cloud/api/plugins/telemetry/DEVICE/bbd53320-75a5-11ef-8424-bdf021575205/values/attributes/SERVER_SCOPE' \
  -H 'x-authorization: Bearer $YOUR_JWT_TOKEN_HERE' \
  -H 'content-type: application/json' 
```
{: .copy-code}

The output will include 'key', 'value' and timestamp of the last update:

```json
[
    {
        "lastUpdateTs": 1617633139380,
        "key": "newAttributeName",
        "value": "newAttributeValue"
    }
]
```
{: .copy-code}

As an alternative to curl, you may use [Java](/docs/{{docsPrefix}}reference/rest-client/) or [Python](/docs/{{docsPrefix}}reference/python-rest-client/) REST clients.

### 2. Shared attributes

- Shared attributes is available only for [devices](/docs/user-guide/ui/devices/#adding-a-new-device). It is similar to the server-side attributes, the shared attributes have important differences:
- The device firmware/application may request the value of the shared attribute(s) or subscribe to the updates of the attribute(s);
- The devices which communicate over **MQTT** or other bidirectional communication protocols may subscribe to attribute updates and receive notifications in real-time;
- The devices which communicate over **HTTP** or other request-response communication protocols may periodically request the value of a shared attribute.

{:refdef: style="text-align: center;"}
![image](/images/user-guide/shared-attributes.svg)
{: refdef}

The primary application of shared attributes is to store device configurations. Given this, let's explore an example within a building monitoring system, featuring predefined attributes as outlined in the following table:

| Attribute Name        | Type of Data | Value | Description                                                       |
|-----------------------|--------------|-------|-------------------------------------------------------------------|
| 'targetFirmwareVersion' | Double       | 1.47  | May be used to store the firmware version for a particular Device.|
| 'maxTemperature'     | Integer      | 51    | May be used to automatically enable HVAC if it is too hot in the room.|

The user may change the attribute via Administration UI. The script or other server-side application may change the attribute value via REST API.

#### Administration UI

To add attributes to a device via the Administration UI, let's follow steps below":

{% include images-gallery.html imageCollection="shared-attrs-pe-edge-ui" %}

{% include templates/info-banner.md content=bulk_provisioning %}

#### REST API

Use [REST API](/docs/{{docsPrefix}}reference/rest-api/) documentation to get the value of the JWT token. You will use it to populate the 'X-Authorization' header and authenticate your REST API call request.

Send a POST request with JSON representation of the attribute to the following URL:

```text
https://$YOUR_THINGSBOARD_HOST/api/plugins/telemetry/$ENTITY_TYPE/$ENTITY_ID/SHARED_SCOPE
```

The example below creates an attribute with the name "**newAttributeName**" and value "**newAttributeValue**" for the device with "**ID**": 'bbd53320-75a5-11ef-8424-bdf021575205' and ThingsBoard Cloud server:
```shell
curl -v 'http://localhost:18080/api/plugins/telemetry/DEVICE/bbd53320-75a5-11ef-8424-bdf021575205/SHARED_SCOPE' \
-H 'x-authorization: Bearer $YOUR_JWT_TOKEN_HERE' \
-H 'content-type: application/json' \
--data-raw '{"newAttributeName":"newAttributeValue"}'
```
{: .copy-code}

Similarly, you can fetch all shared attributes using the following command:

```shell
curl -v -X GET 'http://localhost:18080/api/plugins/telemetry/DEVICE/bbd53320-75a5-11ef-8424-bdf021575205/values/attributes/SHARED_SCOPE' \
  -H 'x-authorization: Bearer $YOUR_JWT_TOKEN_HERE' \
  -H 'content-type: application/json' \
```
{: .copy-code}

The output will include 'key', 'value' and timestamp of the last update:

```json
[
    {
        "lastUpdateTs": 1617633139380,
        "key": "newAttributeName",
        "value": "newAttributeValue"
    }
]
```
{: .copy-code}

As an alternative to curl, you may use [Java](/docs/{{docsPrefix}}reference/rest-client/) or [Python](/docs/{{docsPrefix}}reference/python-rest-client/) REST clients.

#### 3. API for device firmware or applications:

- request *shared* attributes from the server: [MQTT API](/docs/{{docsPrefix}}reference/mqtt-api/#request-attribute-values-from-the-server), [CoAP API](/docs/{{docsPrefix}}reference/coap-api/#request-attribute-values-from-the-server), [HTTP API](/docs/{{docsPrefix}}reference/http-api/#request-attribute-values-from-the-server), [LwM2M API](/docs/{{docsPrefix}}reference/lwm2m-api/#attributes-api);
- subscribe to *shared* attribute updates from the server: [MQTT API](/docs/{{docsPrefix}}reference/mqtt-api/#subscribe-to-attribute-updates-from-the-server), [CoAP API](/docs/{{docsPrefix}}reference/coap-api/#subscribe-to-attribute-updates-from-the-server), [HTTP API](/docs/{{docsPrefix}}reference/http-api/#subscribe-to-attribute-updates-from-the-server), [LwM2M API](/docs/{{docsPrefix}}reference/lwm2m-api/#attributes-api);.

{% capture missed_updates %}

{% capture delete_restrictions %}
If the device went offline, it may miss the important attribute update notification.<br> We recommend subscribing to attribute updates on application startup and requesting the latest values of the attributes after each connect or reconnect.
{% endcapture %}
{% include templates/info-banner.md content=delete_restrictions %}

{% endcapture %}
{% include templates/info-banner.md content=missed_updates %}

### Client-side attributes

This type of attributes is available only for Devices. It is used to report various semi-static data from Device (Client) to ThingsBoard (Server). 
It is like [shared attributes](/docs/{{docsPrefix}}user-guide/attributes/#shared-attributes), but has one important difference:
- The device firmware/application may send the value of the attributes from the device to the platform.

{:refdef: style="text-align: center;"}
![image](/images/user-guide/client-side-attributes.svg)
{: refdef}

The most common use case of client attributes is to report device state.
Let's assume the same building monitoring solution and review a few examples:


| Attribute Name            | Type of Data | Description                                                                                           |
|---------------------------|--------------|-------------------------------------------------------------------------------------------------------|
| `currentFirmwareVersion`  | Double       | May be used to report the installed firmware/application version for the device to the platform.      |
| `currentConfiguration`    | String       | May be used to report the current firmware / application configuration to the platform.               |
| `currentState`            | Bool         | May be used to persist and restore the current firmware / application state via network if the device does not have persistent storage. |


The user and server-side applications may browser the client-side attributes via UI/REST API, but they are not able to change them. 
Basically, the value of the client-side attribute is read-only for the UI/REST API.

#### Fetch client-side attributes via REST API

Use [REST API](/docs/{{docsPrefix}}reference/rest-api/) documentation to get the value of the JWT token. You will use it to populate the 'X-Authorization' header and authenticate your REST API call request.

Send GET request to the following URL:

```text
https://$YOUR_THINGSBOARD_HOST/api/plugins/telemetry/$ENTITY_TYPE/$ENTITY_ID/CLIENT_SCOPE
```
{: .copy-code}

The example below gets all attributes for a device with **"ID"** 'ad17c410-914c-11eb-af0c-d5862211a5f6' and ThingsBoard **PE Edge** server:

```shell
curl -v -X GET 'https://thingsboard.cloud/api/plugins/telemetry/DEVICE/ad17c410-914c-11eb-af0c-d5862211a5f6/values/attributes/CLIENT_SCOPE' \
  -H 'x-authorization: Bearer $YOUR_JWT_TOKEN_HERE' \
  -H 'content-type: application/json' \
```
{: .copy-code}

The output will include 'key', 'value' and timestamp of the last update:

```json
[
    {
        "lastUpdateTs": 1617633139380,
        "key": "newAttributeName",
        "value": "newAttributeValue"
    }
]
```
{: .copy-code}

As an alternative to curl, you may use [Java](/docs/{{docsPrefix}}reference/rest-client/) or [Python](/docs/{{docsPrefix}}reference/python-rest-client/) REST clients.

#### API for device firmware or applications:

- publish *client-side* attributes to the server: [MQTT API](/docs/{{docsPrefix}}reference/mqtt-api/#publish-attribute-update-to-the-server), [CoAP API](/docs/{{docsPrefix}}reference/coap-api/#publish-attribute-update-to-the-server), [HTTP API](/docs/{{docsPrefix}}reference/http-api/#publish-attribute-update-to-the-server);
- request *client-side* attributes from the server: [MQTT API](/docs/{{docsPrefix}}reference/mqtt-api/#request-attribute-values-from-the-server), [CoAP API](/docs/{{docsPrefix}}reference/coap-api/#request-attribute-values-from-the-server), [HTTP API](/docs/{{docsPrefix}}reference/http-api/#request-attribute-values-from-the-server).

## Attributes persistence

ThingsBoard stores the latest value of the attribute and last modification time in the SQL database. This enables the use of [entity filters](/docs/{{docsPrefix}}user-guide/dashboards/#entity-filters) in the dashboards.
Changes to the attributes initiated by the user are recorded in the [audit logs](/docs/{{docsPrefix}}user-guide/audit-log/).
  
## Data Query API

Telemetry Controller provides the following REST API to fetch entity data:

![image](/images/user-guide/telemetry-service/rest-api.png)

{% capture api_note %}
**NOTE:** The API listed above is available via Swagger UI. Please review the general [REST API](/docs/{{docsPrefix}}reference/rest-api/) documentation for more details.
The API is backward compatible with TB v1.0+, and this is the main reason why API call URLs contain "plugin."
{% endcapture %}
{% include templates/info-banner.md content=api_note %}

## Data visualization

We assume you have already provisioned device attributes. Now you may use them in your dashboards. We recommend [dashboards overview](/docs/{{docsPrefix}}user-guide/dashboards/) to get started.
Once you are familiar with how to create dashboards and configure data sources, 
you may use [digital](/docs/{{docsPrefix}}user-guide/ui/widget-library/#digital-gauges) and [analog](/docs/{{docsPrefix}}user-guide/ui/widget-library/#analog-gauges) gauges to visualize 
temperature, speed, pressure, or other numeric values. You may also use [cards](/docs/{{docsPrefix}}user-guide/ui/widget-library/#cards) to visualize multiple attributes using card or [entities table](/docs/{{docsPrefix}}user-guide/ui/entity-table-widget/).

You may also use [input widgets](/docs/{{docsPrefix}}user-guide/ui/widget-library/#input-widgets) to allow dashboard users to change the values of the attributes on the dashboards.

## Rule engine

The [Rule Engine](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/re-getting-started/) is responsible for processing all sorts of incoming data and events.
You may find most popular scenarios of using attributes within rule engine below:

**Generate alarms based on the logical expressions against attribute values**

Use [alarm rules](/docs/{{docsPrefix}}user-guide/device-profiles/#alarm-rules) to configure the most common alarm conditions via UI 
or use [filter nodes](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/filter-nodes/) to configure more specific use cases via custom JS functions.

**Modify incoming client-side attributes before they are stored in the database**

Use [message type switch](/docs/{{docsPrefix}}/user-guide/rule-engine-2-0/filter-nodes/#message-type-switch-node) rule node to filter messages that contain "Post attributes" request. 
Then, use [transformation rule nodes](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/transformation-nodes/) to modify a particular message. 

**React on the change of server-side attribute**

Use [message type switch](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/filter-nodes/#message-type-switch-node) rule node to filter messages that contain "Attributes Updated" notification.
Then, use [action](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/action-nodes/) or [external](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/external-nodes/) to react on the incoming event.

**Fetch attribute values to analyze incoming telemetry from device**

Use [enrichment](/docs/{{docsPrefix}}user-guide/rule-engine-2-0/enrichment-nodes/) rule nodes to enrich the incoming telemetry message with attributes of the device, related asset, customer, or tenant.
This is an extremely powerful technique that allows to modify processing logic and parameters based on settings stored in the attributes. 

{% unless docsPrefix == "paas/" %}

## Performance enhancement

You can achieve higher performance with Attributes Cache enabled (see <b>cache.attributes.enabled</b> property of the [Configuration properties](/docs/user-guide/install/{{docsPrefix}}config/#thingsboard-core-settings)) 

Having attributes cache enabled, ThingsBoard will load the specific attribute from the database only once, each subsequent request to the attribute will be loaded from the faster cache connection.

{% endunless %}

{% capture delete_restrictions %}
**NOTE:** If you are using Redis cache, make sure that you change <b>maxmemory-policy</b> to <b>allkeys-random</b> to prevent Redis from filling up all available memory.
{% endcapture %}
{% include templates/info-banner.md content=delete_restrictions %}

## Old video Tutorial

<div id="video">
  <div id="video_wrapper">
    <iframe src="https://www.youtube.com/embed/JCW_hShAp7I" frameborder="0" allowfullscreen=""></iframe>
  </div>
</div>
