.. _device_handler_ref:

==============
Device Handler
==============

Device Handlers are the virtual representation of a physical device.

A Device Handler defines a `metadata()`_ method that defines the device's definition, UX information, as well as how it should behave in the IDE simulator.

A Device Handler typically also defines a `parse()`_ method that is responsible for transforming raw messages from the device into events for the SmartThings platform.

Device Handlers must also define methods for any supported commands, either through its supported capabilities, or device-specific commands.

For more information about the structure of Device Handlers, refer to the `Device Handler's Guide <../device-type-developers-guide/index.html>`__.

.. tip::

    Writing a Device Handler is considered a somewhat advanced topic. Understanding of how a Device Handler is organized and operates is assumed in this reference documentation. You should be familiar with the contents of the `Device Handler's Guide <../device-type-developers-guide/index.html>`__ to get the most out of this documentation.

----

Methods expected to be defined by Device Handlers:

<command name>()
----------------

.. note::

    This method is expected to be defined by Device Handlers.


The definition for a Command supported by this Device Handler. Every Command that a Device Handler supports, either through its capabilities or custom commands, must have a corresponding command method defined.

Commands are the things that a device can do. For example, the "Switch" capability defines the commands "on" and "off". Every Device that supports the "Switch" capability must define an implementation of these commands. This is done by defining methods with the name of the command. For example, ``def on() {}`` and ``def off()``.

The exact implementation of a command method will vary greatly depending upon the device. The command method is responsible for sending protocol and device-specific commands to the physical device.


**Signature:**
    ``Object <command name([arguments])>``

**Returns:**
    `Object`_ - Commands may return any object, but typically do not return anything since they perform some type of action.

**Example:**

.. code-block:: groovy

    metadata {
        // Automatically generated. Make future change here.
        definition (name: "CentraLite Switch", namespace: "smartthings",    author: "SmartThings") {
            ...
            capability "Switch"
            ...
    }
    ...
    // capability "Switch" declared, so all supported commands
    // of "Switch" must be implemented:
    def on() {
        // device-specific commands to turn the switch on
    }

    def off() {
        // device-specific commands to turn the switch off
    }
    ...

----

parse()
-------

.. note::

    This method is expected to be defined by Device Handlers.


Called when messages from a device are received from the hub. The parse method is responsible for interpreting those messages and returning :ref:`event_ref` definitions. Event definitions are maps that contain, at a minimum, name and value entries. They may also contain unit, displayText, displayed, isStateChange, and linkText entries if the default, automatically generated values of these event properties are to be overridden. See the `createEvent()`_ documentation for a description of these properties.

Because the ``parse()`` method is responsible for handling raw device messages, their implementations vary greatly across different Device Handlers.

The ``parse()`` method may return a map defining the :ref:`event_ref` to create and propagate through the SmartThings platform, or a list of events if multiple events should be created. It may also return a HubAction or list of HubAction objects in the case of LAN-connected devices.

**Signature:**
    ``Map parse(String description)``

    ``List<Map> parse(String description)``

    ``HubAction parse(String description)``

    ``List<HubAction> parse(String description)``


**Example:**

.. code-block:: groovy

    def parse(String description) {
        log.debug "Parse description $description"
        def name = null
        def value = null
        if (description?.startsWith("read attr -")) {
            def descMap = parseDescriptionAsMap(description)
            log.debug "Read attr: $description"
            if (descMap.cluster == "0006" && descMap.attrId == "0000") {
                name = "switch"
                value = descMap.value.endsWith("01") ? "on" : "off"
            } else {
                def reportValue = description.split(",").find {it.split(":")[0].trim() == "value"}?.split(":")[1].trim()
                name = "power"
                // assume 16 bit signed for encoding and power divisor is 10
                value = Integer.parseInt(reportValue, 16).intdiv(10)
            }
        } else if (description?.startsWith("on/off:")) {
            log.debug "Switch command"
            name = "switch"
            value = description?.endsWith(" 1") ? "on" : "off"
        }

        // createEvent returns a Map that defines an Event
        def result = createEvent(name: name, value: value)
        log.debug "Parse returned ${result?.descriptionText}"

        // returning the Event definition map creates an Event
        // in the SmartThings platform, and propagates it to
        // SmartApps subscribed to the device events.
        return result
    }

----

apiServerUrl()
--------------

Returns the URL of the server where this Device Handler can be reached for API calls, along with the specified path appended to it. Use this instead of hard-coding a URL to ensure that the correct server URL for this installed instance is returned.

**Signature:**
    ``String apiServerUrl(String path)``

**Parameters:**
    `String`_ ``path`` - the path to append to the URL

**Returns:**
    The URL of the server for this installed instance of the Device Handler.

**Example:**

.. code-block:: groovy

    // logs <server url>/my/path
    log.debug "apiServerUrl: ${apiServerUrl("/my/path")}"

    // The leading "/" will be added if you don't specify it
    // logs <server url>/my/path
    log.debug "apiServerUrl: ${apiServerUrl("my/path")}"

----

attribute()
-----------

Called within the `definition()`_ method to declare that this Device Handler supports an attribute not defined by any of its declared capabilities.

For any supported attribute, it is expected that the Device Handler creates and sends events with the name of the attribute in the `parse()`_ method.

**Signature:**
    ``void attribute(String attributeName, String attributeType [, List possibleValues])``

**Parameter:**
    `String`_ ``attributeName`` - the name of the attribute

    `String`_ ``attributeType`` - the type of the attribute. Available types are "string", "number", and "enum"

    `List`_ ``possibleValues`` *(optional)* - the possible values for this attribute. Only valid with the ``"enum"`` attributeType.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        definition (name: "Some Device Name", namespace: "somenamespace",
                    author: "Some Author") {
            capability "Switch"
            capability "Polling"
            capability "Refresh"

            // also support the attribute "myCustomAttriute" - not defined by supported capabilities.
            attribute "myCustomAttribute", "number"

            // enum attribute with possible values "light" and "dark"
            attribute "someOtherName", "enum", ["light", "dark"]
         }
         ...
    }

----

capability()
------------

Called in the `definition()`_ method to define that this device supports the specified capability.

.. important::

    Whatever commands and attributes defined by that capability should be implemented by the Device Handler. For example, the "Switch" capability specifies support for the "switch" attribute and the "on" and "off" commands - any Device Handler supporting the "Switch" capability must define methods for the commands, and support the "switch" attribute by creating the appropriate events (with the name of the attribute, e.g., "switch")

**Signature:**
    ``void capability(String capabilityName)``

**Parameters:**
    `String`_ ``capabilityName`` - the name of the capability. This is the long-form name of the Capability name, not the "preferences reference".

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        definition (name: "Cerbco Light Switch", namespace: "lennyv62",
                    author: "Len Veil") {
            capability "Switch"
            ...
        }
        ...
    }

    def parse(description) {
        // handle device messages, determine what value of the event is
        return createEvent(name: "switch", value: someValue)
    }

    // need to define the on and off commands, since those
    // are supported by "Switch" capability
    def on() {
        ...
    }

    def off() {

    }

----

carouselTile()
--------------

Called within the `tiles()`_ method to define a tile often used in conjunction with the Image Capture capability, to allow users to scroll through recent pictures.

**Signature:**
    ``void carouselTile(String tileName, String attributeName [,Map options, Closure closure])``

**Parameters:**
    `String`_ ``tileName`` - the name of the tile. This is used to identify the tile when specifying the tile layout.

    `String`_ ``attributeName`` - the attribute this tile is associated with. Each tile is associated with an attribute of the device. The typical pattern is to prefix the attribute name with ``"device."`` - e.g., ``"device.water"``.

    `Map`_ ``options`` *(optional)* - Various options for this tile. Valid options are found in the table below:

    ======================== =========== ===========
    option                   type        description
    ======================== =========== ===========
    width                    `Integer`_  controls how wide the tile is. Default is 1.
    height                   `Integer`_  controls how tall this tile is. Default is 1.
    canChangeIcon            `Boolean`_  ``true`` to allow the user to pick their own icon. Defaults to ``false``.
    canChangeBackground      `Boolean`_  ``true`` to allow a user to choose their own background image for the tile. Defaults to ``false``.
    decoration               `String`_   specify ``"flat"`` for the tile to render without a ring.
    range                    `String`_   used to specify a custom range. In the form of ``"(<lower bound>..<upper bound>)"``
    ======================== =========== ===========

    `Closure`_ ``closure`` *(optional)* - a closure that defines any states for the tile.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    tiles {
        carouselTile("cameraDetails", "device.image", width: 3, height: 2) { }
    }

----

command()
---------

Called within the `definition()`_ method to declare that this Device Handler supports a command not defined by any of its declared capabilities.

For any supported command, it is expected that the Device Handler define a `<command name>()`_ method with a corresponding name.

**Signature:**
    ``void command(String commandName [, List parameterTypes])``

**Parameter:**
    `String`_ ``commandName`` - the name of the command.

    `List`_ ``parameterTypes`` *(optional)* - a list of strings that defines the types of the parameters the command requires (in order), if any. Typical values are "string", "number", and "enum".

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        definition (name: "Some Device Name", namespace: "somenamespace",
                    author: "Some Author") {
            capability "Switch"
            capability "Polling"
            capability "Refresh"

            // also support the attribute "myCustomCommand" - not defined by supported capabilities.
            command "myCustomCommand"

            // commands can take parameters
            command "myCustomCommandWithParams", ["string", "number"]

         }
         ...
    }

    def myCustomCommand() {
        ...
    }

    def myCustomCommandWithParams(def stringArg, def numArg) {
        ...
    }

----

controlTile()
-------------

Called within the `tiles()`_ method to define a tile that allows the user to input a value within a range. A common use case for a control tile is a light dimmer.

**Signature:**
    ``void controlTile(String tileName, String attributeName, String controlType [, Map options, Closure closure])``

**Returns:**
    void

**Parameters:**
    `String`_ ``tileName`` - the name of the tile. This is used to identify the tile when specifying the tile layout.

    `String`_ ``attributeName`` - the attribute this tile is associated with. Each tile is associated with an attribute of the device. The typical pattern is to prefix the attribute name with ``"device."`` - e.g., ``"device.water"``.

    `String`_ ``controlType`` - the type of control. Either ``"slider"`` or ``"control"``.

    `Map`_ ``options`` *(optional)* - Various options for this tile. Valid options are found in the table below:

    ======================== =========== ===========
    option                   type        description
    ======================== =========== ===========
    width                    `Integer`_  controls how wide the tile is. Default is 1.
    height                   `Integer`_  controls how tall this tile is. Default is 1.
    canChangeIcon            `Boolean`_  ``true`` to allow the user to pick their own icon. Defaults to ``false``.
    canChangeBackground      `Boolean`_  ``true`` to allow a user to choose their own background image for the tile. Defaults to ``false``.
    decoration               `String`_   specify ``"flat"`` for the tile to render without a ring.
    range                    `String`_   used to specify a custom range. In the form of ``"(<lower bound>..<upper bound>)"``
    ======================== =========== ===========

    `Closure`_ ``closure`` *(optional)* - A closure that calls any `state()`_ methods to define how the tile should appear for various attribute values.

**Example:**

.. code-block:: groovy

    tiles {
        controlTile("levelSliderControl", "device.level", "slider", height: 1,
                     width: 2, inactiveLabel: false, range:"(0..100)") {
            state "level", action:"switch level.setLevel"
        }
    }

----

createEvent()
-------------

Creates a Map that represents an :ref:`event_ref` object. Typically used in the `parse()`_ method to define Events for particular attributes. The resulting map is then returned from the ``parse()`` method. The SmartThings platform will then create an Event object and propagate it through the system.

**Signature:**
    ``Map createEvent(Map options)``

**Parameters:**
    `Map`_ ``options`` - The various properties that define this Event. The available options are listed below. It is not necessary, or typical, to define all the available options. Typically only the ``name`` and ``value`` options are required.

    ================    =========== ===========
    Property            Type        Description
    ================    =========== ===========
    name (required)     `String`_   the name of the event. Typically corresponds to an attribute name of a capability.
    value (required)    `Object`_   the value of the event. The value is stored as a string, but you can pass numbers or other objects.
    descriptionText     `String`_   the description of this event. This appears in the mobile application activity for the device. If not specified, this will be created using the event name and value.
    displayed           `Boolean`_  specify ``true`` to display this event in the mobile application activity feed, ``false`` to not display. Defaults to ``true``.
    linkText            `String`_   name of the event to show in the mobile application activity feed.
    isStateChange       `Boolean`_  specify ``true`` if this event caused a device attribute to change state. Typically not used, since it will be set automatically.
    unit                `String`_   a unit string, if desired. This will be used to create the ``descriptionText`` if it (the ``descriptionText`` option) is not specified.
    data                `Map`_      A map of additional information to store with the event
    ================    =========== ===========

**Example:**

.. code-block:: groovy

    def parse(String description) {
        ...

        def evt1 = createEvent(name: "someName", value: "someValue")
        def evt2 = createEvent(name: "someOtherName", value: "someOtherValue")

        return [evt1, evt2]
    }

----

definition()
------------

Called within the `metadata()`_ method, and defines some basic information about the device, as well as the supported capabilities, commands, and attributes.

**Signature:**
    ``void definition(Map definitionData, Closure closure)``

**Parameters:**
    `Map`_ ``definitionData`` - defines various metadata about this Device Handler. Valid options are:

    ============== ========== ===========
    option         type       description
    ============== ========== ===========
    name           `String`_  the name of this Device Handler
    namespace      `String`_  the namespace for this Device Handler. Typically the same as the author's github user name.
    author         `String`_  the name of the author.
    ============== ========== ===========

    `Closure`_ ``closure`` - A closure with method calls to `capability()`_ , `command()`_ , or `attribute()`_ .

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        definition (name: "My Device Name", namespace: "mynamespace",
                    author: "My Name") {
            capability "Switch"
            capability "Polling"
            capability "Refresh"

            command "someCustomCommand"

            attribute "someCustomAttribute", "number"
        }
        ...
    }

----

details()
---------

Used within the `tiles()`_ method to define the order that the tiles should appear in.

**Signature:**
    ``void details(List<String> tileDefinitions)``

**Parameters:**
    `List`_ < `String`_ > ``tileDefinitions`` - A list of tile names that defines the order of the tiles (left-to-right, top-to-bottom)

**Returns:**
    void

**Example:**

.. code-block:: groovy

    tiles {
        standardTile("switchTile", "device.switch", width: 2, height: 2,
                     canChangeIcon: true) {
            state "off", label: '${name}', action: "switch.on",
                  icon: "st.switches.switch.off", backgroundColor: "#ffffff"
            state "on", label: '${name}', action: "switch.off",
                  icon: "st.switches.switch.on", backgroundColor: "#E60000"
        }
        valueTile("powerTile", "device.power", decoration: "flat") {
            state "power", label:'${currentValue} W'
        }
        standardTile("refreshTile", "device.power", decoration: "ring") {
            state "default", label:'', action:"refresh.refresh",
                  icon:"st.secondary.refresh",
        }

        main "switchTile"

        // defines what order the tiles are defined in
        details(["switchTile","powerTile","refreshTile"])
    }

----

device
------

The Device object, from which its current properties and history can be accessed.
As of now this object is a different type than the Device object available in SmartApps.
At some point these will be merged, but for now the properties and methods of the device object available to the Device Handler are discussed in the example below:

.. code-block:: groovy

    ...
    // Gets the most recent State for the given attribute
    def state1 = device.currentState("someAttribute")
    def state2 = device.latestState("someOtherAttribute")

    // Gets the current value for the given attribute
    // Return type will vary depending on the device
    def curVal1 = device.currentValue("someAttribute")
    def curVal2 = device.latestValue("someOtherAttribute")

    // gets the display name of the device
    def displayName = device.displayName

    // gets the internal unique system identifier for this device
    def thisId = device.id

    // gets the internal name for this device
    def thisName = device.name

    // gets the user-defined label for this device
    def thisLabel = device.label

----

fingerprint()
-------------

Called within the `definition()`_ method to define the information necessary to pair this device to the hub.

See the `Fingerprinting Section <../device-type-developers-guide/definition-metadata.html#fingerprinting>`__ of the Device Handler guide for more information.

----

getApiServerUrl()
-----------------

Returns the URL of the server where this Device Handler can be reached for API calls. Use this instead of hard-coding a URL to ensure that the correct server URL for this installed instance is returned.

**Signature:**
    ``String getApiServerUrl()``

**Returns:**
    `String`_ - the URL of the server where this Device Handler can be reached.

----

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
    contentType         Request content type and Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

**Returns:**
    void

----

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
    contentType         Request content type and Accept header.
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
    } catch (e) {
        log.error "something went wrong: $e"
    }

----

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
    contentType         Request content type and Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ ``closure`` - The closure that will be called with the response of the request.

----

httpPost()
----------

Executes an HTTP POST request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPost(String uri, String body, Closure closure)``

    ``void httpPost(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP GET call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Request content type and Accept header.
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
    contentType         Request content type and Accept header.
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
            log.debug "response contentType: ${resp.    contentType}"
        }
    } catch (e) {
        log.debug "something went wrong: $e"
    }

----

httpPut()
---------

Executes an HTTP PUT request and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

If the response content type is JSON, the response data will automatically be parsed into a data structure.

**Signature:**
    ``void httpPut(String uri, String body, Closure closure)``

    ``void httpPut(Map params, Closure closure)``

**Parameters:**
    `String`_ ``uri`` - The URI to make the HTTP GET call to

    `String`_ ``body`` - The body of the request

    `Map`_ ``params`` - A map of parameters for configuring the request. The valid parameters are:

    =================== ==============
    Parameter           Description
    =================== ==============
    uri                 Either a URI or URL of of the endpoint to make a request from.
    path                Request path that is merged with the URI.
    query               Map of URL query parameters.
    headers             Map of HTTP headers.
    contentType         Request content type and Accept header.
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

httpPutJson()
-------------

Executes an HTTP PUT request with a JSON-encoded boday and content type, and passes control to the specified closure. The closure is passed one `HttpResponseDecorator`_ argument from which the response content and header information can be extracted.

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
    contentType         Request content type and Accept header.
    requestContentType  Content type for the request, if it is different from the expected response content-type.
    body                Request body that will be encoded based on the given contentType.
    =================== ==============

    `Closure`_ `closure` - The closure that will be called with the response of the request.

----

main()
------

Used to define what tile appears on the main "Things" view in the mobile application. Can be called within the `tiles()`_ method.

**Signature:**
    ``void main(String tileName)``

**Parameters:**
    `String`_ ``tileName`` - the name of the tile to display as the main tile.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    tiles {
        standardTile("switchTile", "device.switch", width: 2, height: 2,
                     canChangeIcon: true) {
            state "off", label: '${name}', action: "switch.on",
                  icon: "st.switches.switch.off", backgroundColor: "#ffffff"
            state "on", label: '${name}', action: "switch.off",
                  icon: "st.switches.switch.on", backgroundColor: "#E60000"
        }
        valueTile("powerTile", "device.power", decoration: "flat") {
            state "power", label:'${currentValue} W'
        }
        standardTile("refreshTile", "device.power", decoration: "ring") {
            state "default", label:'', action:"refresh.refresh",
                  icon:"st.secondary.refresh",
        }

        // The "switchTile" will be main tile, displayed in the "Things" view
        main "switchTile"
        details(["switchTile","powerTile","refreshTile"])
    }

----

metadata()
----------

Used to define metadata such as this Device Handler's supported capabilities, attributes, commands, and UX information.

**Signature:**
    ``void metadata(Closure closure)``

**Parameters:**
    `Closure`_ ``closure`` - a closure that defines the metadata. The closure is expected to have the following methods called in it: `definition()`_ , `simulator()`_ , and `tiles()`_ .

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        definition(name: "device name", namespace: "yournamespace", author: "your name") {

            // supported capabilities, commands, attributes,
        }
        simulator {
            // simulator metadata
        }
        tiles {
            // tiles metadata
        }
    }

----

reply()
-------

Called in the `simulator()`_ method to model the behavior of a physical device when a virtual instance of the Device Handler is run in the IDE.

The simulator matches command strings generated by the device to those specified in the ``commandString`` argument of a reply method and, if a match is found, calls the device handler's parse method with the corresponding messageDescription.

For example, the reply method ``reply "2001FF,2502": "command: 2503, payload: FF"`` models the behavior of a physical Z-Wave switch in responding to an Basic Set command followed by a Switch Binary Get command. The result will be a call to the parse method with a Switch Binary Report command with a value of 255, i.e., the turning on of the switch. Modeling turn off would be done with the reply method ``reply "200100,2502": "command: 2503, payload: 00"``.

**Signature:**
    ``void reply(String commandString, String messageDescription)``

**Parameters:**
    `String`_ ``commandString`` - a String that represents the command.

    `String`_ ``messageDescription`` - a String that represents the message description.

**Returns:**
    void

**Example:**

.. code-block:: groovy

     metadata {
        ...

        // simulator metadata
        simulator {
            // 'on' and 'off' will appear in the messages dropdown, and send
            // "on/off: 1 to the parse method"
            status "on": "on/off: 1"
            status "off": "on/off: 0"

            // simulate reply messages from the device
            reply "zcl on-off on": "on/off: 1"
            reply "zcl on-off off": "on/off: 0"
        }
        ...
    }

----

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

runEvery5Minutes()
------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every five minutes.
Using this method will pick a random start time in the next five minutes, and run every five minutes after that.

**Signature:**
    ``void runEvery5Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the 5 minute period.

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

runEvery10Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every ten minutes.
Using this method will pick a random start time in the next ten minutes, and run every ten minutes after that.

**Signature:**
    ``void runEvery10Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the ten minute period.

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

runEvery15Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every fifteen minutes.
Using this method will pick a random start time in the next five minutes, and run every five minutes after that.

**Signature:**
    ``void runEvery15Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the fifteen minute period.

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

runEvery30Minutes()
-------------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every thirty minutes.
Using this method will pick a random start time in the next thirty minutes, and run every thirty minutes after that.

**Signature:**
    ``void runEvery30Minutes(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the thirty minute period.

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

runEvery1Hour()
---------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every hour.
Using this method will pick a random start time in the next hour, and run every hour after that.

**Signature:**
    ``void runEvery1Hour(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the one hour period.

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

runEvery3Hours()
----------------

Creates a recurring schedule that executes the specified ``handlerMethod`` every three hours.
Using this method will pick a random start time in the next hour, and run every three hours after that.

**Signature:**
    ``void runEvery3Hours(handlerMethod[, options])``

.. tip::

    This is preferred over using ``schedule(cronExpression, handlerMethod)`` for a regular schedule like this because with a cron expression all installations of a SmartApp will execute at the same time.
    With this method, the executions will be spread out over the three hour period.

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

schedule()
----------

Creates a scheduled job that calls the ``handlerMethod`` once per day at the time specified, or according to a cron schedule.

**Signature:**
    ``void schedule(dateTime, handlerMethod)``

    ``void schedule(cronExpression, handlerMethod)``

**Parameters:**

    ``dateTime`` - A `Date`_ object, an ISO-8601 formatted date time string.

    `String`_ ``cronExpression`` - A cron expression that specifies the schedule to execute on.

    ``handlerMethod`` - The method to call. This can be a reference to the method itself, or the method name as a string.

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

sendEvent()
-----------

Create and fire an :ref:`event_ref` . Typically a Device Handler will return the map returned from `createEvent()`_ , which will allow the platform to create and fire the event. In cases where you need to fire the event (outside of the `parse()`_ method), ``sendEvent()`` is used.

**Signature:**
    ``void sendEvent(Map properties)``

**Parameters:**
    `Map`_ ``properties`` - The properties of the event to create and send.

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

.. tip::

    Not all event properties need to be specified. ID properties like ``deviceId`` and ``locationId`` are automatically set, as are properties like ``isStateChange``, ``displayed``, and ``linkText``.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    sendEvent(name: "temperature", value: 72, unit: "F")


----

simulator()
-----------

Defines information used to simulate device interaction in the IDE. Can be called in the `metadata()`_ method.

**Signature:**
    ``void simulator(Closure closure)``

**Parameters:**
    `Closure`_ ``closure`` - the closure that defines the `status()`_ and `reply()`_ messages.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    metadata {
        ...

        // simulator metadata
        simulator {
            // 'on' and 'off' will appear in the messages dropdown, and send
            // "on/off: 1 to the parse method"
            status "on": "on/off: 1"
            status "off": "on/off: 0"

            // simulate reply messages from the device
            reply "zcl on-off on": "on/off: 1"
            reply "zcl on-off off": "on/off: 0"
        }
        ...
    }

----

standardTile()
--------------

Called within the `tiles()`_ method to define a tile to display current state information. For example, to show that a switch is on or off, or that there is or is not motion.

**Signature:**
    ``void standardTile(String tileName, String attributeName [, Map options, Closure closure])``

**Returns:**
    void

**Parameters:**
    `String`_ ``tileName`` - the name of the tile. This is used to identify the tile when specifying the tile layout.

    `String`_ ``attributeName`` - the attribute this tile is associated with. Each tile is associated with an attribute of the device. The typical pattern is to prefix the attribute name with ``"device."`` - e.g., ``"device.water"``.

    `Map`_ ``options`` *(optional)* - Various options for this tile. Valid options are found in the table below:

    ======================== =========== ===========
    option                   type        description
    ======================== =========== ===========
    width                    `Integer`_  controls how wide the tile is. Default is 1.
    height                   `Integer`_  controls how tall this tile is. Default is 1.
    canChangeIcon            `Boolean`_  ``true`` to allow the user to pick their own icon. Defaults to ``false``.
    canChangeBackground      `Boolean`_  ``true`` to allow a user to choose their own background image for the tile. Defaults to ``false``.
    decoration               `String`_   specify ``"flat"`` for the tile to render without a ring.
    ======================== =========== ===========

    `Closure`_ ``closure`` *(optional)* - A closure that calls any `state()`_ methods to define how the tile should appear for various attribute values.

**Example:**

.. code-block:: groovy

    tile {
         standardTile("water", "device.water", width: 2, height: 2) {
            state "dry", icon:"st.alarm.water.dry", backgroundColor:"#ffffff"
            state "wet", icon:"st.alarm.water.wet", backgroundColor:"#53a7c0"
        }
    }

----

state
-----

A map of name/value pairs that a Device Handler can use to save and retrieve data across executions.

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

state()
-------

Called within any of the various tiles method's closure to define options to be used when the current value of the tile's attribute matches the value argument.

**Signature:**
    ``void state(stateName, Map options)``

**Parameters:**
    `String`_ ``stateName`` - the name of the attribute value for which to display this state for.

    `Map`_ ``options`` - a map that defines additional information for this state. The valid options are:

    ==================== =========== ===========
    option               type        description
    ==================== =========== ===========
    action               `String`_   the action to take when this tile is pressed. The form is <capabilityReference>.<command>.
    backgroundColor      `String`_   a hexadecimal color code to use for the background color. This has no effect if the tile has decoration: "flat".
    backgroundColors     `String`_   specify a list of maps of attribute values and colors. The mobile app will match and interpolate between these entries to select a color based on the value of the attribute.
    defaultState         `Boolean`_  specify true if this state should be the active state displayed for this tile.
    icon                 `String`_   the identifier of the icon to use for this state. You can view the icon options here.
    label                `String`_   the label for this state.
    ==================== =========== ===========

**Returns:**
    void

**Example:**

.. code-block:: groovy

    ...
    standardTile("water", "device.water", width: 2, height: 2) {
        // when the "water" attribute has the value "dry", show the
        // specified icon and background color
        state "dry", icon:"st.alarm.water.dry", backgroundColor:"#ffffff"

        // when the "water" attribute has the value "wet", show the
        // specified icon and background color
        state "wet", icon:"st.alarm.water.wet", backgroundColor:"#53a7c0"
    }

    valueTile("temperature", "device.temperature", width: 2, height: 2) {
        state("temperature", label:'${currentValue}°',
            backgroundColors:[
                [value: 31, color: "#153591"],
                [value: 44, color: "#1e9cbb"],
                [value: 59, color: "#90d2a7"],
                [value: 74, color: "#44b621"],
                [value: 84, color: "#f1d801"],
                [value: 95, color: "#d04e00"],
                [value: 96, color: "#bc2323"]
            ]
        )
    }
    ...

----

status()
--------

The status method is called in the `simulator()`_ method, and populates the select box that appears under virtual devices in the IDE. Can be called in the `simulator()`_ method.

**Signature:**
    ``void status(String name, String messageDescription)``

**Parameters:**
    `String` ``name`` - any unique string and is used to refer to this status message in the select box.

    `String` ``messageDescription`` - should be a parseable message for this Device Handler. It's passed to the Device Handler's parse method when select box entry is sent in the simulator. For example, ``status "on": "command: 2003, payload: FF"`` will send a Z-Wave Basic Report command to the device handler's parse method when the on option is selected and sent.

**Returns:**
    void

**Example:**

.. code-block:: groovy

     metadata {
        ...

        // simulator metadata
        simulator {
            // 'on' and 'off' will appear in the messages dropdown, and send
            // "on/off: 1 to the parse method"
            status "on": "on/off: 1"
            status "off": "on/off: 0"

            // simulate reply messages from the device
            reply "zcl on-off on": "on/off: 1"
            reply "zcl on-off off": "on/off: 0"
        }
        ...
    }

----

tiles()
-------

Defines the user interface for the device in the mobile app. It's composed of one or more `standardTile()`_ , `valueTile()`_ , `carouselTile()`_ , or `controlTile()`_ methods, as well as a `main()`_ and `details()`_ method.

**Signature:**
    ``void tiles(Closure closure)``

**Parameters:**
    `Closure`_ ``closure`` - A closure that defines the various tiles and metadata.

**Returns:**
    void

**Example:**

.. code-block:: groovy

    tiles {
        standardTile("switchTile", "device.switch", width: 2, height: 2,
                     canChangeIcon: true) {
            state "off", label: '${name}', action: "switch.on",
                  icon: "st.switches.switch.off", backgroundColor: "#ffffff"
            state "on", label: '${name}', action: "switch.off",
                  icon: "st.switches.switch.on", backgroundColor: "#E60000"
        }
        valueTile("powerTile", "device.power", decoration: "flat") {
                  state "power", label:'${currentValue} W'
        }
        standardTile("refreshTile", "device.power", decoration: "ring") {
            state "default", label:'', action:"refresh.refresh",
                  icon:"st.secondary.refresh",
        }

        main "switchTile"
        details(["switchTile","powerTile","refreshTile"])
    }

----

valueTile()
-----------

Defines a tile that displays a specific value. Typical examples include temperature, humidity, or power values. Called within the `tiles()`_ method.

**Signature:**
    ``void valueTile(String tileName, String attributeName [, Map options, Closure closure])``

**Returns:**
    void

**Parameters:**
    `String`_ ``tileName`` - the name of the tile. This is used to identify the tile when specifying the tile layout.

    `String`_ ``attributeName`` - the attribute this tile is associated with. Each tile is associated with an attribute of the device. The typical pattern is to prefix the attribute name with ``"device."`` - e.g., ``"device.power"``.

    `Map`_ ``options`` *(optional)* - Various options for this tile. Valid options are found in the table below:

    ======================== =========== ===========
    option                   type        description
    ======================== =========== ===========
    width                    `Integer`_  controls how wide the tile is. Default is 1.
    height                   `Integer`_  controls how tall this tile is. Default is 1.
    canChangeIcon            `Boolean`_  ``true`` to allow the user to pick their own icon. Defaults to ``false``.
    canChangeBackground      `Boolean`_  ``true`` to allow a user to choose their own background image for the tile. Defaults to ``false``.
    decoration               `String`_   specify ``"flat"`` for the tile to render without a ring.
    ======================== =========== ===========

    `Closure`_ ``closure`` *(optional)* - A closure that calls any `state()`_ methods to define how the tile should appear for various attribute values.

**Example:**

.. code-block:: groovy

    tiles {
        valueTile("power", "device.power", decoration: "flat") {
            state "power", label:'${currentValue} W'
        }
    }


----

zigbee
------

A utility class for parsing and formatting ZigBee messages.

**Signature:**
    ``Zigbee zigbee``

**Returns:**
    A reference to the :ref:`ZigBee utility class <zigbee_ref>`.



----

zwave
-----

The utility class for parsing and formatting Z-Wave command messages.

**Signature:**
    ``ZWave zwave``

**Returns:**
    A reference to the ZWave helper class. See the :ref:`z_wave_ref` for more information.

**Example:**

.. code-block:: groovy

    // On command implementation for a Z-Wave switch
    def on() {
        delayBetween([
            zwave.basicV1.basicSet(value: 0xFF).format(),
            zwave.switchBinaryV1.switchBinaryGet().format()
        ])
    }


.. _BigDecimal: http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html
.. _Boolean: http://docs.oracle.com/javase/7/docs/api/java/lang/Boolean.html
.. _Closure: http://docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html
.. _Date: http://docs.oracle.com/javase/7/docs/api/java/util/Date.html
.. _String: http://docs.oracle.com/javase/7/docs/api/java/lang/String.html
.. _Map: http://docs.oracle.com/javase/7/docs/api/java/util/Map.html
.. _Integer: https://docs.oracle.com/javase/7/docs/api/java/lang/Integer.html
.. _List: http://docs.oracle.com/javase/7/docs/api/java/util/List.html
.. _Number: http://docs.oracle.com/javase/7/docs/api/java/lang/Number.html
.. _Long: https://docs.oracle.com/javase/7/docs/api/java/lang/Long.
.. _Object: http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html
.. _HttpResponseDecorator: http://javadox.com/org.codehaus.groovy.modules.http-builder/http-builder/0.6/groovyx/net/http/HttpResponseDecorator.html
