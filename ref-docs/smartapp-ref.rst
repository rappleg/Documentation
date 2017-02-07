.. _smartapp_ref:

SmartApp
========

A SmartApp is a Groovy-based program that allows developers to create automations for users to tap into the capabilities of their devices.

They are created through the "New SmartApp" action in the IDE. There is no "class" for a SmartApp per se, but there are various methods and properties available to SmartApps that are documented below.

When a SmartApp executes, it executes in the context of a certain installation instance. That is, a user installs a SmartApp on their mobile application, and configures it with devices or rules unique to them. A SmartApp is not continuously running; it is executed in response to various schedules or subscribed-to events.

----

The following methods should be defined by all SmartApps. They are called by the SmartThings platform at various points in the SmartApp lifecycle.

.. _smartapp_installed:

installed()
-----------

.. note::

    This method is expected to be defined by SmartApps.

Called when an instance of the app is installed. Typically subscribes to events from the configured devices and creates any scheduled jobs.

**Signature:**
    ``void installed()``

**Returns:**
    void

**Example:**

.. code-block:: groovy

    def installed() {
        log.debug "installed with settings: $settings"

        // subscribe to events, create scheduled jobs.
    }

----

.. _smartapp_updated:

updated()
---------

.. note::

    This method is expected to be defined by SmartApps.


Called when the preferences of an installed app are updated. Typically unsubscribes and re-subscribes to events from the configured devices and unschedules/reschedules jobs.

**Signature:**
    ``void uninstalled()``

**Returns:**
    void

**Example:**

.. code-block:: groovy

    def updated() {
        unsubscribe()
        // resubscribe to device events, create scheduled jobs
    }

----

uninstalled()
-------------

.. note::

    This method may be defined by SmartApps.


Called, if declared, when an app is uninstalled. Does not need to be declared unless you have some external cleanup to do. subscriptions and scheduled jobs are automatically removed when an app is uninstalled, so you don't need to do that here.

**Signature:**
    ``void uninstalled()``

**Returns:**
    void

**Example:**

.. code-block:: groovy

    def uninstalled() {
        // external cleanup. No need to unsubscribe or remove scheduled jobs
    }

----

The following methods and attributes are available to call in a SmartApp:

<device or capability preference name>
--------------------------------------

A reference to the device or devices selected during app installation or update.

**Returns:**
    :ref:`device_ref` or a list of Devices - the Device with the given preference name, or a list of Devices if ``multiple:true`` is specified in the preferences.

**Example:**

.. code-block:: groovy

    preferences {
        ...
        input "theswitch", "capability.switch"
        input "theswitches", "capability.switch", multiple:true
        ...
    }

    ...
    // the name of the preference becomes the reference for the Device object
    theswitch.on()
    theswitch.off()

    // multiple:true means we get a list of devices
    theswitches.each {log.debug "Current switch value: ${it.currentSwitch"}

    // we can still call methods directly on the list; it will apply the method to each device:

    theswitches.on() // turn all switches on

----

<number or decimal preference name>
-----------------------------------

A reference to the value entered for a number or decimal input preference.

**Returns:**
    `BigDecimal`_ - the value entered for a number or decimal input preference.

**Example:**

.. code-block:: groovy

    preferences {
        ...
        input "num1", "number"
        input "dec1", "decimal"
        ...
    }

    ...
    // preference name is a reference to a BigDecimal that is the value the user entered.
    log.debug "num1: $num1" //=> value user entered for num1 preference
    log.debug "dec1: $dec1" //=> value user entered for dec1 preference
    ...

----

<text, mode, or time preference name>
-------------------------------------

A reference to the value entered for a ``text``, ``mode``, or ``time`` input type.

The following table explains the value and format returned for the various input types:

==========  ============
Input Type  Return Value
==========  ============
text        `String`_ - the value entered as text
mode        `String`_ - the name of the mode selected
time        `String`_ - the full date string in the format of “yyyy-MM-dd’T’HH:mm:ss.SSSZ"
==========  ============

**Example:**

.. code-block:: groovy

    preferences {
        ...
        input "mytext", "text"
        input "mymode", "mode"
        input "mytime", "time"
        ...
    }

    log.debug "mytext: $mytext"
    log.debug "mymode: $mymode"
    log.debug "mytime: $mytime"

    // time is in format compatible with most scheduling APIs.
    // we can pass the value directly to the APIs that accept a date string:
    runOnce(mytime, someHandlerMethod)
    schedule(myTime, someHandlerMethod)


----

.. _add_child_app:

addChildApp()
-------------

Adds a child app to a SmartApp.

**Signature:**
    ``InstalledSmartApp addChildApp(String namespace, String smartAppVersionName, String label, Map properties)``

**Throws:**
    ``IllegalArgumentException`` - If a label was not supplied
    ``NotFoundException`` - If the given SmartApp name was not found in the given Namespace.

**Parameters:**
    `String`_ ``namespace`` - the namespace of the child SmartApp

    `String`_ ``smartAppVersionName`` - the name of the SmartApp

    `String`_ ``label`` - a label to give the child app

    `Map`_ ``properties`` *(optional)* - A map with SmartApp properties for the child app.

**Returns:**
    :ref:`installed_smart_app_wrapper` - The InstalledSmartAppWrapper instance that represents the child SmartApp that was created.

addChildDevice()
----------------

Adds a child device to a SmartApp. An example use is in service manager SmartApps.

**Signature:**
    ``DeviceWrapper addChildDevice(String namespace, String typeName, String deviceNetworkId, hubId, Map properties)``

**Throws:**
    ``UnknownDeviceTypeException``

**Parameters:**
    `String`_ ``namespace`` - the namespace for the device. Defaults to ``installedSmartApp.smartAppVersionDTO.smartAppDTO.namespace``

    `String`_ ``typeName`` - the device type name

    `String`_ ``deviceNetworkId`` - the device network id of the device

    ``hubId`` - *(optional)* The hub id. Defaults to ``null``

    `Map`_ ``properties`` *(optional)* - A map with device properties.

**Returns:**
    ``DeviceWrapper`` - The device that was created.

----

apiServerUrl()
--------------

Returns the URL of the server where this SmartApp can be reached for API calls, along with the specified path appended to it. Use this instead of hard-coding a URL to ensure that the correct server URL for this installed instance is returned.

**Signature:**
    ``String apiServerUrl(String path)``

**Parameters:**
    `String`_ ``path`` - the path to append to the URL

**Returns:**
    The URL of the server for this installed instance of the SmartApp.

**Example:**

.. code-block:: groovy

    // logs <server url>/my/path
    log.debug "apiServerUrl: ${apiServerUrl("/my/path")}"

    // The leading "/" will be added if you don't specify it
    // logs <server url>/my/path
    log.debug "apiServerUrl: ${apiServerUrl("my/path")}"

----

atomicState
-----------

A map of name/value pairs that SmartApp can use to save and retrieve data across SmartApp executions. This is similar to :ref:`smartapp-state`, but will immediately write and read from the backing data store. Prefer using ``state`` over ``atomicState`` when possible.

**Signature:**
    ``Map atomicState``

**Returns:**
    `Map`_ - a map of name/value pairs.

.. code-block:: groovy

    atomicState.count = 0
    atomicState.count = atomicState.count + 1

    log.debug "atomicState.count: ${atomicState.count}"

    // use array notation if you wish
    log.debug "atomicState['count']: ${atomicState['count']}"

    // you can store lists and maps to make more intersting structures
    atomicState.listOfMaps = [[key1: "val1", bool1: true],
                        [otherKey: ["string1", "string2"]]]

----

.. _smartapp_can_schedule:

canSchedule()
-------------

Returns true if the SmartApp is able to schedule jobs. SmartApps are limited to 6 pending scheduled executions.

**Signature:**
    ``Boolean canSchedule()``

**Returns:**
    `Boolean`_ - ``true`` if additional jobs can be scheduled, ``false`` otherwise.

**Example:**

.. code-block:: groovy

    log.debug "Can schedule? ${canSchedule()}"

----

.. _smartapp_find_all_child_apps_by_name:

findAllChildAppsByName()
------------------------

Finds all child SmartApps matching the specified name.
This includes child SmartApps that have both "complete" and "incomplete" :ref:`installation states <isa_get_installation_state>`.

**Signature:**
    ``List<InstalledSmartApp> findAllChildAppsByName(String namespace, String name)``

**Parameters:**
    `String`_ ``name`` - the name of the SmartApp to find.

**Returns:**
    A list of :ref:`installed_smart_app_wrapper`, or an empty list if none are found.

**Example:**

.. code-block:: groovy

    def children = findAllChildAppsByName("My Child App")
    log.debug "found ${children.size()} child apps"

    children.each { child ->
        log.debug "child app ${child.id} has installation state ${child.installationState}"
    }

----

.. _smartapp_find_all_child_apps_by_namespace_and_name:

findAllChildAppsByNamespaceAndName()
------------------------------------

Finds all child SmartApps matching the specified namespace and name.
This includes child SmartApps that have both "complete" and "incomplete" :ref:`installation states <isa_get_installation_state>`.

**Signature:**
    ``List<InstalledSmartApp> findAllChildAppsByNamespaceAndName(String namespace, String name)``

**Parameters:**
    `String`_ ``namespace`` - the namespace of the SmartApp to find.

    `String`_ ``name`` - the name of the SmartApp to find.

**Returns:**
    A list of :ref:`installed_smart_app_wrapper`, or an empty list if none are found.

**Example:**

.. code-block:: groovy

    def children = findAllChildAppsByNamespaceAndName("somenamespace", "My Child App")
    log.debug "found ${children.size()} child apps"

    children.each { child ->
        log.debug "child app ${child.id} has installation state ${child.installationState}"
    }

----

.. _smartapp_find_child_app_by_name:

findChildAppByName()
--------------------

Finds a child SmartApp matching the specified name.
This includes child SmartApps that have both "complete" and "incomplete" :ref:`installation states <isa_get_installation_state>`.

**Signature:**
    ``def findChildAppByName(String appName)``

**Parameters:**
    `String`_ ``appName`` - the name of the SmartApp to find.

**Returns:**
    A :ref:`installed_smart_app_wrapper` if a child app is found that matches the specified name; ``null`` if no child app that matches the name is found.
    If there are multiple child apps that match the specified name, only the first one found will be returned.

**Example:**

.. code-block:: groovy

    def child = findChildAppByName("My Child App")
    log.debug "child app id ${child?.id} has installation state ${child.installationState}"

----

.. _smartapp_find_child_app_by_namespace_and_name:

findChildAppByNamespaceAndName()
--------------------------------

Finds a child SmartApp matching the specified namespace and name.
This includes child SmartApps that have both "complete" and "incomplete" :ref:`installation states <isa_get_installation_state>`.

**Signature:**
    ``def findChildAppsByNamespaceAndName(String namespace, String name)``

**Parameters:**
    `String`_ ``namespace`` - the namespace of the SmartApp to find.

    `String`_ ``name`` - the name of the SmartApp to find.

**Returns:**
    A :ref:`installed_smart_app_wrapper`, or null if no child app is found.
    If multiple child apps are found that match the namespace and name, the first one will be returned.

**Example:**

.. code-block:: groovy

    def child = findChildAppByNamespaceAndName("somenamespace", "My Child App")
    log.debug "child app id ${child?.id} has installation state ${child.installationState}"

----

.. _smartapp_get_all_child_apps:

getAllChildApps()
-----------------

Gets a list of child apps associated with this SmartApp.
This includes child SmartApps that have both "complete" and "incomplete" :ref:`installation states <isa_get_installation_state>`.

**Signature:**
    ``List<InstalledSmartApp> getAllChildApps()``

**Returns:**
    `List`_ < :ref:`installed_smart_app_wrapper` > - A list of child SmartApps

**Example:**

.. code-block:: groovy

    def childApps = app.getAllChildApps()
    log.debug "This app has ${childApps.size()} child apps"

    childApps.each { child ->
        log.debug "child app with id ${child.id} has installation state ${child.installationState}"
    }

----

.. _smartapp_get_child_apps:

getChildApps()
--------------

Gets a list of child apps associated with this SmartApp.
This only includes child SmartApps that have an :ref:`installation state <isa_get_installation_state>` of "complete".

**Signature:**
    ``List<InstalledSmartApp> getChildApps()``

**Returns:**
    `List`_ < :ref:`installed_smart_app_wrapper` > - A list of child SmartApps

**Example:**

.. code-block:: groovy

    def childApps = getChildApps()

    // Update the label for all child apps
    childApps.each {
        if (!it.label?.startsWith(app.name)) {
            it.updateLabel("$app.name/$it.label")
        }
    }

----

deleteChildDevice()
-------------------

Deletes the child device with the specified device network id.

**Signature:**
    ``void deleteChildDevice(String deviceNetworkId)``

**Throws:**
    ``NotFoundException``

**Parameters:**
    `String`_ ``deviceNetworkId`` - the device network id of the device

**Returns:**
    void

----

getAllChildDevices()
--------------------

Returns a list of all child devices, including virtual devices. This is a wrapper for ``getChildDevices(true)``.

**Signature:**
    ``List getAllChildDevices()``

**Returns:**
    `List`_ - a list of all child devices.

----

getApiServerUrl()
-----------------

Returns the URL of the server where this SmartApp can be reached for API calls. Use this instead of hard-coding a URL to ensure that the correct server URL for this installed instance is returned.

**Signature:**
    ``String getApiServerUrl()``

**Returns:**
    `String`_ - the URL of the server where this SmartApp can be reached.

----

getChildDevice()
----------------

Returns a device based upon the specified device network id. This is mostly used in service manager SmartApps.

**Signature:**
    ``DeviceWrapper getChildDevice(String deviceNetworkId)``

**Parameters:**
    `String`_ ``deviceNetworkId`` - the device network id of the device

**Returns:**
    ``DeviceWrapper`` - The device found with the given device network ID.

----

getChildDevices()
-----------------

Returns a list of all child devices. An example use would be in service manager SmartApps.

**Signature:**
    ``List getChildDevices(Boolean includeVirtualDevices)``

**Parameters:**
    `Boolean`_ ``true`` if the returned list should contain virtual devices. Defaults to ``false``. *(optional)*

**Returns:**
    `List`_ - A list of all devices found.

----

.. _smartapp_getlocation:

getLocation()
-------------

The :ref:`location_ref` into which this SmartApp has been installed.

**Signature:**
    ``Location getLocation()``

**Returns:**
    :ref:`location_ref` - The Location into which this SmartApp has been installed.

----

.. _smartapp_get_sunrise_and_sunset:

getSunriseAndSunset()
---------------------

Gets a map containing the local sunrise and sunset times.

**Signature:**
    ``Map getSunriseAndSunset([Map options])``

**Parameters:**

    `Map`_ ``options`` *(optional)*

    The supported options are:

    ==============  ===========
        Option      Description
    ==============  ===========
    zipCode         | `String`_ - the zip code to use for determining the times.
                    | If not specified then the coordinates of the hub location are used.
    locationString  | `String`_ - any location string supported by the Weather Underground APIs.
                    | If not specified then the coordinates of the hub location are used
    sunriseOffset   | `String`_ - adjust the sunrise time by this amount.
                    | See `timeOffset()`_ for supported formats
    sunsetOffset    | `String`_ - adjust the sunset time by this amount.
                    | See `timeOffset()`_ for supported formats
    ==============  ===========

**Returns:**
    `Map`_ - A Map containing the local sunrise and sunset times as `Date`_ objects: ``[sunrise: Date, sunset: Date]``

**Example:**

.. code-block:: groovy

    def noParams = getSunriseAndSunset()
    def beverlyHills = getSunriseAndSunset(zipCode: "90210")
    def thirtyMinsBeforeSunset = getSunriseAndSunset(sunsetOffset: "-00:30")

    log.debug "sunrise with no parameters: ${noParams.sunrise}"
    log.debug "sunset with no parameters: ${noParams.sunset}"
    log.debug "sunrise and sunset in 90210: $beverlyHills"
    log.debug "thirty minutes before sunset at current location: ${thirtyMinsBeforeSunset.sunset}"

----

getWeatherFeature()
-------------------

Calls the Weather Underground API to to return weather forecasts and related data.

**Signature:**
    ``Map getWeatherFeature(String featureName [, String location])``

.. note::

    ``getWeatherFeature`` simply delegates to the Weather Underground API, using the specfied ``featureName`` and ``location`` (if specified). For full descriptions on the available features and return information, please consult the `Weather Underground API docs <http://www.wunderground.com/weather/api/d/docs?>`__.


**Parameters:**
    `String`_ ``featureName``
    The weather feature to get. This corresponds to the available "Data Features" in the Weather Underground API.

    `String`_ ``location`` *(optional)*
    The location to get the weather information for (ZIP code). If not specified, the location of the user's hub will be used.

**Returns:**
    `Map`_ - a Map containing the weather information requested. The data returned will vary depending on the feature requested. See the Weather Underground API documentation for more information.

----

.. _smartapp_http_delete:

httpDelete()
------------

Executes an HTTP DELETE request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

**Signature:**
    ``void httpDelete(String uri, Closure closure)``

    ``void httpDelete(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP DELETE call to.

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

**Returns:**
    void

----

.. _smartapp_http_error:

httpError()
-----------

Throws a ``SmartAppException`` with the specified status code and message.

This should be used to send an HTTP error to any calling client.

**Signature:**
    ``def httpError(Integer status, message)``

**Parameters:**
    `Integer`_ status - The HTTP error code to send.
    message - the error message.

**Example:**

.. code-block:: groovy

    def someMethod() {
        httpError(400, "something went wrong")
    }

----

.. _smartapp_http_get:

httpGet()
---------

Executes an HTTP GET request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpGet(String uri, Closure closure)``

    ``void httpGet(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP GET call to

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ - ``closure`` - The closure that will be called with the response of the request.


**Example:**

.. code-block:: groovy

    def params = [
        uri: "http://httpbin.org",
        path: "/get"
    ]

    try {
        httpGet(params) { resp ->
            resp.headers.each {
            log.debug "${it.name} : ${it.value}"
        }
        log.debug "response contentType: ${resp.contentType}"
        log.debug "response data: ${resp.data}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }

----

.. _smartapp_http_head:

httpHead()
----------

Executes an HTTP HEAD request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

**Signature:**
    ``void httpHead(String uri, Closure closure)``

    ``void httpHead(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP HEAD call to

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

----

.. _smartapp_http_post:

httpPost()
----------

Executes an HTTP POST request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPost(String uri, String body, Closure closure)``

    ``void httpPost(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP POST call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.


**Example:**

.. code-block:: groovy

    try {
        httpPost("http://mysite.com/api/call", "id=XXX&value=YYY") { resp ->
            log.debug "response data: ${resp.data}"
            log.debug "response contentType: ${resp.contentType}"
        }
    } catch (e) {
        log.debug "something went wrong: $e"
    }

----

.. _smartapp_http_post_json:

httpPostJson()
--------------

Executes an HTTP POST request with a JSON-encoded body and content type, and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPostJson(String uri, String body, Closure closure)``

    ``void httpPostJson(String uri, Map body, Closure closure)``

    ``void httpPostJson(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP POST call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

**Example:**

.. code-block:: groovy

    def params = [
        uri: "http://postcatcher.in/catchers/<yourUniquePath>",
        body: [
            param1: [subparam1: "subparam 1 value",
                     subparam2: "subparam2 value"],
            param2: "param2 value"
        ]
    ]

    try {
        httpPostJson(params) { resp ->
            resp.headers.each {
                log.debug "${it.name} : ${it.value}"
            }
            log.debug "response contentType: ${resp.contentType}"
        }
    } catch (e) {
        log.debug "something went wrong: $e"
    }

----

.. _smartapp_http_put:

httpPut()
---------

Executes an HTTP PUT request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPut(String uri, String body, Closure closure)``

    ``void httpPut(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP PUT call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

**Example:**

.. code-block:: groovy

    try {
        httpPut("http://mysite.com/api/call", "id=XXX&value=YYY") { resp ->
            log.debug "response data: ${resp.data}"
            log.debug "response contentType: ${resp.contentType}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }

----

.. _smartapp_http_put_json:

httpPutJson()
-------------

Executes an HTTP PUT request with a JSON-encoded body and content type, and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPutJson(String uri, String body, Closure closure)``

    ``void httpPutJson(String uri, Map body, Closure closure)``

    ``void httpPutJson(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP PUT call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Forced response content type and request Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ `closure` - The closure that will be called with the response of the request.

----

.. _smartapp_nextoccurrence:

nextOccurrence()
----------------

Returns a Date when the time specified in the input occurs next.

**Signature:**
    ``Date nextOccurrence(String timeString)``

**Parameters:**
    `String`_ ``timeString`` - An ISO-8601 date string as returned from ``time`` input preferences of the SmartApp.

.. note::

    Note that if the input ``timeString`` does not contain time zone, this method will throw an ``IllegalArgumentException``.

**Returns:**
    `Date`_ - The Date when the time specified in the ``timeString`` occurs next. If the specified time has already occurred, then returns the next day Date object when the specified time occurs next. If the specified time has not yet occurred, then returns today's Date object when the specified time will occur.

**Example:**

.. code-block:: groovy

    preferences {

      section() {
            input "Time1", "time", title: "Time1"
            input "Time2", "time", title: "Time2"
      }
    }

    ...

    // Current time is 16:25 October 24, 2016, Time1 input is 16:23 and Time2 input is 16:34
    log.debug "nextOccurrence(Time1) value is: ${nextOccurrence(Time1)}"
    log.debug "nextOccurrence(Time2) value is: ${nextOccurrence(Time2)}"
    // The above log statements will print the following:
    nextOccurrence(Time1) value is: Tue Oct 25 23:23:00 UTC 2016
    nextOccurrence(Time2) value is: Mon Oct 24 23:34:00 UTC 2016

----

.. _smartapp_now:

now()
-----

Gets the current Unix time in milliseconds.

**Signature:**
    ``Long now()``

**Returns:**
    `Long`_ - the current Unix time.

----

parseJson()
-----------

Parses the specified string into a JSON data structure.

**Signature:**
    ``Map parseJson(stringToParse)``

**Parameters:**
    `String`_ ``stringToParse`` - The string to parse into JSON

**Returns:**
    `Map`_ - a map that represents the passed-in string in JSON format.

----

parseXml()
----------

Parses the specified string into an XML data structure.

**Signature:**
    ``GPathResult parseXml(stringToParse)``

**Parameters:**
    `String`_ ``stringToParse`` - The string to parse into XML

**Returns:**
    `GPathResult`_ - A GPathResult instance that represents the passed-in string in XML format.

----

parseLanMessage()
-----------------

Parses a Base64-encoded LAN message received from the hub into a map with header and body elements, as well as parsing the body into an XML document.

**Signature:**
    ``Map parseLanMessage(stringToParse)``

**Parameters:**
    `String`_ ``stringToParse`` - The string to parse

**Returns:**
    `Map`_  - a map with the following structure:

    ======== ============== ===================
    key      type           description
    ======== ============== ===================
    header   `String`_      the headers of the request as a single string
    headers  `Map`_         a Map of string/name value pairs for each header
    body     `String`_      the request body as a string
    ======== ============== ===================

----

parseSoapMessage()
------------------

Parses a Base64-encoded LAN message received from the hub into a map with header and body elements, as well as parsing the body into an XML document. This method is commonly used to parse `UPNP SOAP <http://www.w3.org/TR/soap12-part1/>`__ messages.

**Signature:**
    ``Map parseLanMessage(stringToParse)``

**Parameters:**
    `String`_ ``stringToParse`` - The string to parse

**Returns:**
    `Map`_ - A map with the following structure:

    ======== ============== ===================
    key      type           description
    ======== ============== ===================
    header   `String`_      the headers of the request as a single string
    headers  `Map`_         a Map of string/name value pairs for each header
    body     `String`_      the request body as a string
    xml      `GPathResult`_ the request body as a `GPathResult`_ object
    xmlError `String`_      error message from parsing the body, if any
    ======== ============== ===================

----

.. _smartapp_render:

render()
--------

Returns a HTTP response to the calling client with the options specified.

**Signature:**
    ``def render(Map options)``

**Parameters:**
    `Map`_ options - the options for what is returned to the client:

    =========== ===========
    option      description
    =========== ===========
    contentType The value of the "Content-Type" request header. "application/json" if not specified.
    status      The HTTP status of the response. 200 if not specified.
    data        Required. The data for this response.
    =========== ===========

**Example:**

.. code-block:: groovy

    def someMethod() {
        def html = """
            <!DOCTYPE HTML>
            <html>
                <head><title>Some Title</title></head>
                <body><p>Some Text</p></body>
            </html>
        """

        render contentType: "text/html", data: html
    }

----

.. _smartapp_run_in:

runIn()
-------

Executes a specified ``handlerMethod`` after ``delaySeconds`` have elapsed.

**Signature:**
    ``void runIn(delayInSeconds, handlerMethod [, options])``

.. tip::

    It's important to note that we will attempt to run this method at this time, but cannot guarantee exact precision. We typically expect per-minute level granularity, so if using with values less than sixty seconds, your mileage will vary.

**Parameters:**
    ``delayInSeconds`` - The number of seconds to execute the ``handlerMethod`` after.

    ``handlerMethod`` - The method to call after ``delayInSeconds`` has passed. Can be a string or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    overwrite ``true`` or ``false``  Specify ``[overwrite: false]`` to not overwrite any existing pending schedule handler for the given method (the default behavior is to overwrite the pending schedule). Specifying ``[overwrite: false]`` can lead to multiple different schedules for the same handler method, so be sure your handler method can handle this.
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runIn(300, myHandlerMethod)
    runIn(400, "myOtherHandlerMethod", [data: [flag: true]])

    def myHandlerMethod() {
        log.debug "handler method called"
    }

    def myOtherHandlerMethod(data) {
        log.debug "other handler method called with flag: $data.flag"
    }

----

.. _smartapp_run_every_1_minute:

runEvery1Minute()
------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every minute.
Using this method will pick a random start time in the next minute, and run every minute after that.

**Signature:**
    ``void runEvery1Minute(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the 1 minute period.

**Parameters:**
    ``handlerMethod`` - The method to call every minute. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery1Minute(handlerMethod1)
    runEvery1Minute(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_5_minutes:

runEvery5Minutes()
------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every five minutes.
Using this method will pick a random start time in the next five minutes, and run every five minutes after that.

**Signature:**
    ``void runEvery5Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the 5 minute period.

**Parameters:**
    ``handlerMethod`` - The method to call every five minutes. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery5Minutes(handlerMethod1)
    runEvery5Minutes(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_10_minutes:

runEvery10Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every ten minutes. Using this method will pick a random start time in the next ten minutes, and run every ten minutes after that.

**Signature:**
    ``void runEvery10Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the ten minute period.

**Parameters:**
    ``handlerMethod`` - The method to call every ten minutes. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery10Minutes(handlerMethod1)
    runEvery10Minutes(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_15_minutes:

runEvery15Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every fifteen minutes. Using this method will pick a random start time in the next five minutes, and run every five minutes after that.

**Signature:**
    ``void runEvery15Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the fifteen minute period.

**Parameters:**
    ``handlerMethod`` - The method to call every fifteen minutes. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery15Minutes(handlerMethod1)
    runEvery15Minutes(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_30_minutes:

runEvery30Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every thirty minutes. Using this method will pick a random start time in the next thirty minutes, and run every thirty minutes after that.

**Signature:**
    ``void runEvery30Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the thirty minute period.

**Parameters:**
    ``handlerMethod`` - The method to call every thirty minutes. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery30Minutes(handlerMethod1)
    runEvery30Minutes(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_1_hours:

runEvery1Hour()
---------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every hour. Using this method will pick a random start time in the next hour, and run every hour after that.

**Signature:**
    ``void runEvery1Hour(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the one hour period.

**Parameters:**
    ``handlerMethod``- The method to call every hour. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery1Hour(handlerMethod1)
    runEvery1Hour(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_every_3_hours:

runEvery3Hours()
----------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every three hours. Using this method will pick a random start time in the next hour, and run every three hours after that.

**Signature:**
    ``void runEvery3Hours(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time. With this method, the executions will be spread out over the three hour period.

**Parameters:**
    ``handlerMethod`` - The method to call every three hours. Can be the name of the method as a string, or a reference to the method.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    runEvery3Hours(handlerMethod1)
    runEvery3Hours(handlerMethod2, [data: [key1: 'val1']])

    def handlerMethod1() {
        log.debug "handlerMethod1"
    }

    def handlerMethod2(data) {
        log.debug "handlerMethod2, data: $data"
    }

----

.. _smartapp_run_once:

runOnce()
---------

Executes the ``handlerMethod`` once at the specified date and time.

**Signature:**
    ``void runOnce(dateTime, handlerMethod [, options])``

**Parameters:**
    ``dateTime`` - When to execute the ``handlerMethod``. Can be either a `Date`_ object or an ISO-8601 date string. For example, ``new Date() + 1`` would run at the current time tomorrow, and ``"2017-07-04T12:00:00.000Z"`` would run at noon GMT on July 4th, 2017.

    ``handlerMethod`` - The method to execute at the specified ``dateTime``. This can be a reference to the method, or the method name as a string.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    overwrite ``true`` or ``false``  Specify ``[overwrite: false]`` to not overwrite any existing pending schedule handler for the given method (the default behavior is to overwrite the pending schedule). Specifying ``[overwrite: false]`` can lead to multiple different schedules for the same handler method, so be sure your handler method can handle this.
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    // execute handler at 4 PM CST on October 21, 2015 (e.g., Back to the Future 2 Day!)
    runOnce("2015-10-21T16:00:00.000-0600", handler)

    def handler() {
        ...
    }

----

.. _smartapp_schedule:

schedule()
----------

Creates a scheduled job that calls the ``handlerMethod`` once per day at the time specified, or according to a cron schedule.

**Signature:**
    ``void schedule(dateTime, handlerMethod [, options])``

    ``void schedule(cronExpression, handlerMethod [, options])``

**Parameters:**

    ``dateTime`` - A `Date`_ object, an ISO-8601 formatted date time string.

    `String`_ ``cronExpression`` - A cron expression that specifies the schedule to execute on.

    ``handlerMethod`` - The method to call. This can be a reference to the method itself, or the method name as a string.

    ``options`` *(optional)* - A map of parameters, with the following keys supported:

    ========= ====================== ===========
    Key       Possible values        Description
    ========= ====================== ===========
    data      A map of data          A map of data that will be passed to the handler method.
    ========= ====================== ===========

**Returns:**
    void

.. tip::

    Since calling ``schedule()`` with a dateTime argument creates a recurring scheduled job to execute *every day* at the specified time, the *date information is ignored. Only the time portion of the argument is used.*

.. tip::

    Full documentation for the cron expression format can be found in the `Quartz Cron Trigger Tutorial <http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html>`__

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "timeToRun", "time"
        }
    }

    ...
    // call handlerMethod1 at time specified by user input
    schedule(timeToRun, handlerMethod1)

    // call handlerMethod2 every day at 3:36 PM CST
    schedule("2015-01-09T15:36:00.000-0600", handlerMethod2)

    // execute handlerMethod3 every hour on the half hour (using a randomly chosen seconds field)
    schedule("12 30 * * * ?", handlerMethod3)
    ...

    def handlerMethod1() {...}
    def handlerMethod2() {...}
    def handlerMethod3() {...}

----

.. _smartapp_send_event:

sendEvent()
-----------

Creates and sends an event constructed from the specified properties. If a device is specified, then a DEVICE event will be created, otherwise an APP event will be created.

.. note::

    SmartApps typically *respond to events*, not create them. In more rare cases, certain SmartApps or Service Manager SmartApps may have reason to send events themselves. ``sendEvent`` can be used for those cases.

**Signature:**
    ``void sendEvent(Map properties)``

    ``void sendEvent(Device device, Map properties)``

**Parameters:**
    `Map`_ ``properties`` - The properties of the event to create and send.

    Here are the available properties:

    =================    ===========
    Property             Description
    =================    ===========
    name (required)      `String`_ - The name of the event. Typically corresponds to an attribute name of a capability.
    value (required)     The value of the event. The value is stored as a string, but you can pass numbers or other objects.
    descriptionText      `String`_ - The description of this event. This appears in the mobile application activity for the device. If not specified, this will be created using the event name and value.
    displayed            Pass ``true`` to display this event in the mobile application activity feed, ``false`` to not display. Defaults to ``true``.
    linkText             `String`_ - Name of the event to show in the mobile application activity feed.
    isStateChange        ``true`` if this event caused a device attribute to change state. Typically not used, since it will be set automatically.
    unit                 `String`_ - a unit string, if desired. This will be used to create the ``descriptionText`` if it (the ``descriptionText`` option) is not specified.
    :ref:`device_ref`    ``device`` - The device for which this event is created for.
    data                 A map of additional information to store with the event
    =================    ===========


.. tip::

    Not all event properties need to be specified. ID properties like ``deviceId`` and ``locationId`` are automatically set, as are properties like ``isStateChange``, ``displayed``, and ``linkText``.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    // create and send an event with name "temperature" and value 72
    sendEvent(name: "temperature", value: 72, unit: "F")

    // create and send event with additional data
    sendEvent(name: "myevent", value: "myvalue", data: [moreInfo: "more information", evenMoreInfo: 42])

----

.. _smartapp_sendhubcommand:

sendHubCommand()
----------------

Sends a command to the Hub, with the details of the command encapsulated within a HubAction object.

**Signature:**
    ``void sendHubCommand(HubAction action)``

    ``void sendHubCommand(List<HubAction> actions, delay)``

**Parameters:**
    ``HubAction action`` - A HubAction object

    ``List<HubAction> actions`` - A list of HubAction objects


    ``delay`` - An integer number representing milliseconds. This is the delay between commands when a list of HubAction objects are sent using ``List<HubAction> actions`` parameter. The default value of delay is 1000.

**Returns:**
    void

**Example:**
    During the discovery phase of a LAN-connected device the following discovery command can be sent to the Hub.

.. code-block:: groovy

    // Send a single HubAction command to the Hub
    void ssdpDiscover() {
        sendHubCommand(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:ZonePlayer:1", physicalgraph.device.Protocol.LAN))
    }
    // Send a List of HubAction commands to the Hub with a delay of 3 seconds between each HubAction command
    void sendMultiDevice() {
        List actions = []
        actions.add(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:ZonePlayer:1", physicalgraph.device.Protocol.LAN))
        actions.add(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:MediaRenderer:1", physicalgraph.device.Protocol.LAN))
        actions.add(new physicalgraph.device.HubAction("lan discovery urn:samsung.com:device:RemoteControlReceiver:1", physicalgraph.device.Protocol.LAN))
        sendHubCommand(actions, 3000)
    }

----

sendLocationEvent()
-------------------

Sends a LOCATION event constructed from the specified properties. See the :ref:`event_ref` reference for a list of available properties. Other SmartApps can receive location events by subscribing to the location. Examples of exisisting location events include sunrise and sunset.

**Signature:**
    ``void sendLocationEvent(Map properties)``

**Parameters:**
    `Map`_ ``properties`` - The properties from which to create and send the event.

    Here are the available properties:

    ================    ===========
    Property            Description
    ================    ===========
    name (required)     `String`_ - The name of the event. Typically corresponds to an attribute name of a capability.
    value (required)    The value of the event. The value is stored as a string, but you can pass numbers or other objects.
    descriptionText     `String`_ - The description of this event. This appears in the mobile application activity for the device. If not specified, this will be created using the event name and value.
    displayed           Pass ``true`` to display this event in the mobile application activity feed, ``false`` to not display. Defaults to ``true``.
    linkText            `String`_ - Name of the event to show in the mobile application activity feed.
    isStateChange       ``true`` if this event caused a device attribute to change state. Typically not used, since it will be set automatically.
    unit                `String`_ - a unit string, if desired. This will be used to create the ``descriptionText`` if it (the ``descriptionText`` option) is not specified.
    data                A map of additional information to store with the event
    ================    ===========

**Returns:**
    void

----

.. _smartapp_send_notification:

sendNotification()
------------------

Sends the specified message and displays it in the *Hello, Home* portion of the mobile application.

**Signature:**
    ``void sendNotification(String message [, Map options])``

**Parameters:**
    `String`_ ``message`` - The message to send to *Hello, Home*

    `Map`_ ``options`` *(optional)* - Options for the message. The following options are available:

    ======== ===========
    option   description
    ======== ===========
    method   `String`_ - One of ``"phone"``, ``"push"``, or ``"both"``. Defaults to "``both``".
    event    ``false`` to supress displaying in *Hello, Home*. Defaults to ``true``.
    phone    `String`_ - The phone number to send the SMS message to. Required when the ``method`` is ``"phone"``. If not specified and method is "``both``", then no SMS message will be sent.
    ======== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendNotification("test notification - no params")
    sendNotification("test notification - push", [method: "push"])
    sendNotification("test notification - sms", [method: "phone", phone: "1234567890"])
    sendNotification("test notification - both", [method: "both", phone: "1234567890"])
    sendNotification("test notification - no event", [event: false])

----

.. _smartapp_send_notification_event:

sendNotificationEvent()
-----------------------

Displays a message in *Hello, Home*, but does not send a push notification or SMS message.

**Signature:**
    ``void sendNotificationEvent(String message)``

**Parameters:**
    `String`_ ``message`` - The message to send to *Hello, Home*

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendNotificationEvent("some message")

----

.. _smartapp_send_notification_to_contact:

sendNotificationToContacts()
----------------------------

Sends the specified message to the specified contacts.

**Signature:**
    ``void sendNotificationToContacts(String message, String contact, Map options=[:])``

    ``void sendNotificationToContacts(String message, Collection contacts, Map options=[:])``

**Parameters:**
    `String`_ ``message`` - the message to send

    `String`_ ``contact`` - the contact to send the notification to. Typically set through the ``contact`` input type.

    `Collection`_ ``contacts`` - the collection of contacts to send the notification to. Typically set through the ``contact`` input type.

    `Map`_ ``options`` *(optional)* - a map of additional parameters. The valid parameter is ``[event: boolean]`` to specify if the message should be displayed in the Notifications feed. Defaults to ``true`` (message will be displayed in the Notifications feed).

**Returns:**
    void

**Example:**

.. code-block:: groovy

    preferences {
        section("Send Notifications?") {
            input("recipients", "contact", title: "Send notifications to") {
                input "phone", "phone", title: "Warn with text message (optional)",
                    description: "Phone Number", required: false
            }
        }
    }

    ...
    if (location.contactBookEnabled) {
        sendNotificationToContacts("Your house talks!", recipients)
    }
    ...

.. tip::

    It's a good idea to assume that a user *may not* have any contacts configured. That's why you see the nested ``"phone"`` input in the preferences (user will only see that if they don't have contacts), and why we check ``location.contactBookEnabled``.

.. _smartapp_send_push:

sendPush()
----------

Sends the specified message as a push notification to users mobile devices and displays it in *Hello, Home*.

**Signature:**
    ``void sendPush(String message)``

**Parameters:**
    `String`_ ``message`` - The message to send

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendPush("some message")

----

.. _smartapp_send_push_message:

sendPushMessage()
-----------------

Sends the specified message as a push notification to users mobile devices but does not display it in *Hello, Home*.

**Signature:**
    ``void sendPushMessage(String message)``

**Parameters:**
    `String`_ ``message`` - The message to send

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendPushMessage("some message")

----

.. _smartapp_send_sms:

sendSms()
---------

Sends the message as an SMS message to the specified phone number and displays it in Hello, Home. The message can be no longer than 140 characters.

**Signature:**
    ``void sendSms(String phoneNumber, String message)``

**Parameters:**
    `String`_ ``phoneNumber`` - the phone number to send the SMS message to.

    `String`_ ``message`` - the message to send. Can be no longer than 140 characters.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendSms("somePhoneNumber", "some message")

----

.. _smartapp_send_sms_message:

sendSmsMessage()
----------------

Sends the message as an SMS message to the specified phone number but does not display it in Hello, Home. The message can be no longer than 140 characters.

**Signature:**
    ``void sendSmsMessage(String phoneNumber, String message)``

**Parameters:**
    `String`_ ``phoneNumber`` - the phone number to send the SMS message to.

    `String`_ ``message`` - the message to send. Can be no longer than 140 characters.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendSms("somePhoneNumber", "some message")

----

.. _smartapp_set_location_mode:

setLocationMode()
-----------------

Set the mode for this location.

**Signature:**
    ``void setLocationMode(String mode)``
    ``void setLocationMode(Mode mode)``

**Returns:**
    void

.. warning::

    ``setMode()`` will raise an error if the specified mode does not exist for the location. You should verify the mode exists as in the example below.

**See Also:** :ref:`location.setMode() <location_set_mode>`

----

settings
--------

A map of name/value pairs containing all of the installed SmartApp's preferences.

**Signature:**
    ``Map settings``

**Returns:**
    `Map`_ - a map containing all of the installed SmartApp's preferences.

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "myswitch", "capability.switch"
            input "mytext", "text"
            input "mytime", "time"
        }
    }

    ...

    log.debug "settings.mytext: ${settings.mytext}"
    log.debug "settings.mytime: ${settings.mytime}"

    // if the input is a device/capability, you can get the device object
    // through the settings:
    log.debug "settings.myswitch.currentSwitch: ${settings.myswitch.currentSwitch}"
    ...

----

.. _smartapp-state:

state
-----

A map of name/value pairs that SmartApps can use to save and retrieve data across SmartApp executions.

**Signature:**
    ``Map state``

**Returns:**
    `Map`_ - a map of name/value pairs.

.. code-block:: groovy

    state.count = 0
    state.count = state.count + 1

    log.debug "state.count: ${state.count}"

    // use array notation if you wish
    log.debug "state['count']: ${state['count']}"

    // you can store lists and maps to make more intersting structures
    state.listOfMaps = [[key1: "val1", bool1: true],
                        [otherKey: ["string1", "string2"]]]

.. warning::

    Though ``state`` can be treated as a map in most regards, certain convenience operations that you may be accustomed to in maps will not work with ``state``. For example, ``state.count++`` will not increment the count - use the longer form of ``state.count = state.count + 1``.

----

stringToMap()
-------------

Parses a comma-delimited string into a map.

**Signature:**
    ``Map stringToMap(String string)``

**Parameters:**
    `String`_ string - A comma-delimited string to parse into a map.

**Returns:**
    `Map`_ - a map created from the comma-delimited string.

**Example:**

.. code-block:: groovy

    def testStr = "key1: value1, key2: value2"
    def testMap = stringToMap(testStr)

    log.debug "stringToMap: ${testMap}"
    log.debug "stringToMap.key1: ${testMap.key1}" // => value1
    log.debug "stringToMap.key2: ${testMap.key2}" // => value2

----

.. _smartapp_subscribe:

subscribe()
-----------

Subscribes to the various events for a device or location. The specified ``handlerMethod`` will be called when the event is fired.

All event handler methods will be passed an :ref:`event_ref` that represents the event causing the handler method to be called.

**Signature:**
    ``void subscribe(deviceOrDevices, String attributeName, handlerMethod)``

    ``void subscribe(deviceOrDevices, String attributeNameAndValue, handlerMethod)``

    ``void subscribe(Location location, handlerMethod)``

    ``void subscribe(Location location, String eventName, handlerMethod)``

    ``void subscribe(app, handlerMethod)``

**Parameters:**
    ``deviceOrDevices`` - The :ref:`device_ref` or list of devices to subscribe to.

    `String`_ ``attributeName`` - The attribute to subscribe to.

    `String`_ ``attributeNameAndValue`` - The specific attribute value to subscribe to, in the format ``"<attributeName>.<attributeValue>"``

    ``handlerMethod`` - The method to call when the event is fired. Can be a `String`_ of the method name or the method reference itself.

    :ref:`location_ref` ``location`` - The location to subscribe to

    ``app`` - Pass in the available ``app`` property in the SmartApp to subscribe to touch events in the app.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "mycontact", "capability.contactSensor"
            input "myswitches", "capability.switch", multiple: true
        }
    }
    // subscribe to all state change events for ``contact`` attribute of a contact sensor
    subscribe(mycontact, "contact", handlerMethod)

    // subscribe to all state changes for all switch devices configured
    subscribe(myswitches, "switch", handlerMethod)

    // subscribe to the "open" event for the contact sensor - only when the state changes to "open" will the handlerMethod be called
    subscribe(mycontact, "contact.open", handlerMethod)

    // subscribe to all state change events for the installed SmartApp's location
    subscribe(location, handlerMethod)

    // subscribe to touch events for this app - handlerMethod called when app is touched
    subscribe(app, appTouchMethod)

    // all event handler methods must accept an event parameter
    def handlerMethod(evt) {
        log.debug "event name: ${evt.name}"
        log.debug "event value: ${evt.value}"
    }

----

.. _smartapp_subscribe_to_command:

subscribeToCommand()
--------------------

Subscribes to device commands that are sent to a device. The specified ``handlerMethod`` will be called whenever the specified ``command`` is sent.

**Signature:**
    ``void subscribeToCommand(device, commandName, handlerMethod)``

**Parameters:**

    ``device`` - The :ref:`device_ref` to subscribe to.

    `String`_ ``commandName`` - The command to subscribe to

    ``handlerMethod`` - the method to call when the command is called.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "switch1", "capability.switch"
        }
    }
    ...
    subscribeToCommand(switch1, "on", onCommand)
    ...
    // called when the on() command is called on switch1
    def onCommand(evt) {...}


----

.. _smartapp_timeofdayisbetween:

timeOfDayIsBetween()
--------------------

Find if a given date is between a lower and upper bound.

**Signature:**
    ``Boolean timeOfDayIsBetween(Date start, Date stop, Date value, TimeZone timeZone)``

**Parameters:**
    `Date`_ ``start`` - The start date to compare against.

    `Date`_ ``stop`` - The end date to compare against.

    `Date`_ ``value`` - The date to compare to ``start`` and ``stop``.

    `TimeZone`_ ``timeZone`` - The time zone for this comparison.

**Returns:**
    `Boolean`_ - ``true`` if the specified date is between the ``start`` and ``stop`` dates, false otherwise.

**Example:**

.. code-block:: groovy

    def between = timeOfDayIsBetween(new Date() - 1, new Date() + 1,
                                     new Date(), location.timeZone)
    log.debug "between: $between" => true

----

timeOffset()
------------

Gets a time offset in milliseconds for the specified input.

**Signature:**
    ``Long timeOffset(Number minutes)``

    ``Long timeOffset(String hoursAndMinutesString)``

**Parameters:**
    `Number`_ ``minutes`` - The number of minutes to get the offset in milliseconds for.

    `String`_ ``hoursAndMinutesString`` - A string in the format of ``"hh:mm"`` to get the offset in milliseconds for. Negative offsets are specified by prefixing the string with a minus sign (``"-02:30"``).

**Returns:**
    `Long`_ - the time offset in milliseconds for the specified input.

**Example:**

.. code-block:: groovy

    def off1 = timeOffset(24)       // => 1440000
    def off2 = timeOffset("2:30")   // => 9000000
    def off2again = timeOffset(150) // => 9000000
    def off3 = timeOffset("-02:30") // => -9000000

----

timeToday()
-----------

Gets a `Date`_ object for today's date, for the specified time in the date-time parameter.

**Signature:**
    ``Date timeToday(String timeString [, TimeZone timeZone])``

**Parameters:**
    `String`_ ``timeString`` - Either an ISO-8601 date string as returned from ``time`` input preferences, or a simple time string in ``"hh:mm"`` format ("21:34").

    `TimeZone`_ ``timeZone`` *(optional)* - The time zone to use for determining the current day.

.. warning::

    Although the ``timeZone`` argument is optional, it is *strongly encouraged* that you use it. Not specifying the ``timeZone`` results in the SmartThings platform trying to calculate the time zone based on the date and time zone offsets in the input string.

    To avoid time zone errors, you should specify the ``timeZone`` argument (you can get the time zone from the ``location`` object: ``location.timeZone``)

    Future releases may remove the option to call ``timeToday`` without a time zone.

**Returns:**
    `Date`_ - the Date that represents today's date for the specified time.

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "startTime", "time"
            input "endTime", "time"
        }
    }
    ...
    def start = timeToday(startTime, location.timeZone)
    def end = timeToday(endTime, location.timeZone)

----

timeTodayAfter()
----------------

Returns a `Date`_ of the next occurrence of the time specified in the input, relative to a reference time.

**Signature:**
    ``Date timeTodayAfter(String startTimeString, String timeString [, TimeZone timeZone])``

**Parameters:**
    `String`_ ``startTimeString`` - The reference time. Can be an ISO-8601 date string as returned from ``time`` input preferences, or a simple time string in ``"hh:mm"`` format ("21:34").

    `String`_ ``timeString`` - The time string whose next occurrence is queried. Can be an ISO-8601 date string as returned from ``time`` input preferences, or a simple time string in ``"hh:mm"`` format ("21:34").

    `TimeZone`_ ``timeZone`` *(optional)* - The time zone used for determining the current date and time.

.. warning::

    Although the ``timeZone`` argument is optional, it is *strongly encouraged* that you use it. Not specifying the ``timeZone`` results in the SmartThings platform trying to calculate the time zone based on the date and time zone offsets in the input string.

    To avoid time zone errors, you should specify the ``timeZone`` argument (you can get the time zone from the ``location`` object: ``location.timeZone``)

    Future releases may remove the option to call ``timeToday`` without a time zone.

**Returns:**
    `Date`_ - If time specified by ``timeString`` has already occurred prior to ``startTimeString`` then returns the next day Date object when the ``timeString`` time occurs next. If ``timeString`` time has not yet occurred relative to ``startTimeString``, then returns today's Date object when the ``timeString`` time will occur. Since only the occurrence of ``timeString`` after the elapse of ``startTimeString`` time is considered, the Date returned is guaranteed to be later than the ``startTimeString`` date.

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "time1", "time"
            input "time2", "time"
        }
    }
    ...
    // assume time1 entered as 20:20
    // assume time2 entered as 14:05
    // since 14:05 time today has already elapsed prior to 20:20 reference time today,
    // the nextTime would be tomorrow's date, 14:05 time (the next occurrence of 14:05 time)
    def nextTime = timeTodayAfter(time1, time2, location.timeZone)

----

.. _smartapp_timezone:

timeZone()
----------

Get a `TimeZone` object for the specified time value entered as a SmartApp preference. This will get the current time zone of the mobile app (not the hub location).

**Signature:**
    ``TimeZone timeZone(String timePreferenceString)``

**Parameters:**
    `String`_ ``timePreferenceString`` - The time value string in IS0-8061 format as entered as input in SmartApp time preferences.

**Returns:**
    `TimeZone`_ - the TimeZone for the time value as specified by the ``timePreferenceString``.

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "mytime", "time"
        }
    }

    ...
    def enteredTimeZone = timeZone(mytime)
    ...

----

toDateTime()
------------

Get a `Date`_ object for the specified string.

**Signature:**
    ``Date toDateTime(dateTimeString)``

**Parameters:**
    `String`_ ``dateTimeString`` - the date-time string for which to get a Date object, in ISO-8061 format as used by time preferences

**Returns:**
    `Date`_ - the Date for the specified ``dateTimeString``.

**Example:**

.. code-block:: groovy

    preferences {
        section() {
            input "mytime", "time"
        }
    }
    ...
    Date myTimeAsDate = toDateTime(mytime)
    ...

----

.. _smartapp_unschedule:

unschedule()
------------

Deletes all scheduled jobs for the SmartApp.
If using the optional ``method`` parameter, then it deletes the scheduled job for the specified handler name only.

**Signature:**
    ``void unschedule(String method = '')``

**Returns:**
    void

.. note::

    This can be an expensive operation if unscheduling all scheduled jobs; make sure you need to do this before calling. Typically called in the `updated()`_ method if the SmartApp has set up recurring schedules.


----

.. _smartapp_unsubscribe:

unsubscribe()
-------------

Deletes all subscriptions for the installed SmartApp, or for a specific device or devices if specified.

Typically should be called in the `updated()`_ method, since device preferences may have changed.

**Signature:**
    ``unsubscribe([deviceOrDevices])``

**Paramters:**
    ``deviceOrDevices`` *(optional)* - The device or devices for which to unsubscribe from. If not specified, all subscriptions for this installed SmartApp will be deleted.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    def updated() {
        unsubscribe()
    }

----

.. _BigDecimal: http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html
.. _Boolean: http://docs.oracle.com/javase/7/docs/api/java/lang/Boolean.html
.. _Closure: http://docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html
.. _Date: http://docs.oracle.com/javase/7/docs/api/java/util/Date.html
.. _String: http://docs.oracle.com/javase/7/docs/api/java/lang/String.html
.. _List: http://docs.oracle.com/javase/7/docs/api/java/util/List.html
.. _Map: http://docs.oracle.com/javase/7/docs/api/java/util/Map.html
.. _Number: http://docs.oracle.com/javase/7/docs/api/java/lang/Number.html
.. _Long: https://docs.oracle.com/javase/7/docs/api/java/lang/Long.
.. _GPathResult: http://docs.groovy-lang.org/latest/html/api/groovy/util/slurpersupport/GPathResult.html
.. _TimeZone: http://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html
.. _HttpResponseDecorator: http://javadox.com/org.codehaus.groovy.modules.http-builder/http-builder/0.6/groovyx/net/http/HttpResponseDecorator.html
.. _Collection: https://docs.oracle.com/javase/7/docs/api/java/util/Collection.html
.. _Integer: http://docs.oracle.com/javase/7/docs/api/java/lang/Integer.html
