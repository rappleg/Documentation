.. _capabilities_taxonomy:

Capabilities Reference
======================

.. warning::

    The Capabilities reference documentation here is no longer being maintained.

    For the current Capabilities documentation, please refer to the `SmartThings developer portal <https://smartthings.developer.samsung.com/develop/api-ref/capabilities.html>`_.

.. only:: latex or epub

    See our `online documentation <http://docs.smartthings.com/en/latest/capabilities-reference.html>`_ for complete and updated capabilities documentation.

.. only:: html

    Capabilities are core to the SmartThings architecture. They allow us to abstract specific devices into their underlying capabilities.

    An application interacts with devices based on their capabilities, so once we understand the capabilities that are needed by a SmartApp, and the capabilities that are provided by a device, we can understand which devices (based on the Device’s declared capabilities) are eligible for use within a specific SmartApp.

    Capabilities themselves are decomposed into both Commands and Attributes. Commands represent ways in which you can control or actuate the device, whereas Attributes represent state information or properties of the device.

    Capabilities are created and maintained by the SmartThings internal development team.

    This page serves as a reference for the supported capabilities.

    ----

    Data Types
    ----------

    Before we present the Capabilities, it's worth covering the various Data Types associated with Capability Attributes and Commands.
    It is essential to understand that these Data Types are guidelines as to how actual values can be represented.
    In most cases, Device Handlers contain the implementation logic and define the actual objects for these Data Types.
    Below is a table outlining the the possible Data Types and what they mean.

    ============== ============================= ======================================
    Data Type      Example                       Description
    ============== ============================= ======================================
    STRING         "This is a String"            Represents character strings
    NUMBER         ``5``, ``10.67``              The Number data type is a guideline indicating that a number should be expected, and not a specific type. Device Handlers contain the implementation of what kind of number object is actually returned.
    VECTOR3        ``(x,y,z)``                   This Data Type is a representation of x,y,z coordinates in space. Device Handlers contain the implementation of the actual data structure, but it is usually as a Map: ``[x: 0, y: 0, z: 0]``.
    ENUM           "one", "two", "three"         The Enum Data Type is a static set of predefined String values that an Attribute can have, or that a Command can accept as an argument.
    DYNAMIC_ENUM   "Any", "value"                Much like the Enum Data Type, Dynamic Enum is a set of String values. However, the set is not static or predefined.
    COLOR_MAP      ``[hue: 50, saturation: 75]`` The Color Map is a Map specifically for the use of color control. As such, the Map should contain a Hue and a Saturation value.
    JSON_OBJECT                                  A standard JSON object. Device Handlers contain the implementation and thus should be consulted when looking for the JSON object structure.
    DATE                                         A Date, usually represented as a java.util.Date object.
    BOOLEAN        true, false                   A boolean data type with a value of true or false.
    ============== ============================= ======================================

    ----

    Capabilities at a glance
    ------------------------

    The Capabilities reference table below lists all capabilities. The various columns are:

    Name:
        The name of the capability that is used by a Device Handler.
    Preferences Reference:
        The string you would use in a SmartApp to allow a user to select from devices supporting this capability.

    {#- First iterate through the capabilities to create a quick reference table #}
    {% for capability in capabilities %}
    {%- set referenceName = capability['reference']['@name'] %}
    {%- if loop.first %}
    {{ '+' }}{{ '-' * 40 }}{{ '+' }}{{ '-' * 40 }}{{ '+' }}
    {{ '|' }} Name{{ ' ' * 35 }}{{ '|' }} Preferences Reference{{ ' ' * 18 }}{{ '|' }}
    {{ '+' }}{{ '=' * 40 }}{{ '+' }}{{ '=' * 40 }}{{ '+' }}
    {% endif -%}
    {{ '|' }} :ref:`{{ referenceName }}`{{ ' ' * (45 - (referenceName|length) - 13) }}{{ '|' }} {{ "capability."+referenceName }}{{ ' ' * (40 - (referenceName|length) - 12)}}{{ '|' }}
    {{ '+' }}{{ '-' * 40 }}{{ '+' }}{{ '-' * 40 }}{{ '+' }}
    {% endfor %}
    ----
    {#- Generate detail section for each capability #}
    {% for capability in capabilities %}
    {% set referenceName = capability['reference']['@name'] %}
    .. _{{ referenceName }}:
    {# Capability name as section header #}
    {{ capability['@name'] }}
    {{ '-' * capability['@name']|length }}
    {# Capability description #}
    {%- if properties[referenceName][referenceName+".note"] %}
    .. note::
    {{ properties[referenceName][referenceName+".note"]|indent(2, true) }}
    {% endif %}
    {%- if properties[referenceName][referenceName+".description"] %}
    {{ properties[referenceName][referenceName+".description"] }}
    {%- endif %}
    {# Capability preference reference #}
    Preferences Reference
    ^^^^^^^^^^^^^^^^^^^^^
    {{ ("``capability."+referenceName+"``")|indent(2, true) }}
    {# Capability attributes section #}
    Attributes
    ^^^^^^^^^^
    {% if capability['attribute'] %}
    {#- check if the attribute key in dict is a list. It will not be a list if there is only one attribute #}
    {%- if capability['attribute'] is a_list %}
    {#- for each attribute, print its name and type followed by its attribute value list if present #}
    {%- for attribute in capability['attribute'] %}
      *{{ attribute['@name'] }}:* {{ attribute['@type'] }}{% if attribute['@optional'] and attribute['@optional'] == "true" %} - Optional{% endif %}
        {%- if properties[referenceName][referenceName+".attr."+attribute['@name']+".description"] %}
        {{ properties[referenceName][referenceName+".attr."+attribute['@name']+".description"]|indent(2, true) }}
        {% else %}
        {% endif %}
        {%- if attribute['value'] %}
        {%- if attribute['value'] is a_list %}
        {%- for value in attribute['value'] %}
	  {% if attribute['@type'] == 'ENUM' %}
	  ``{{ value['@name'] }}``
	  {%- else %}
          ``{{ value['@name'] }}`` - {{ value['@type'] }}
	  {%- endif %}
          {%- if properties[referenceName][referenceName+".attr."+attribute['@name']+"."+value['@name']+".value"] %}
          {{ properties[referenceName][referenceName+".attr."+attribute['@name']+"."+value['@name']+".value"]|indent(4, true) }}
          {% endif %}
        {% endfor %}
        {%- else %}
        {% if attribute['@type'] == 'ENUM' %}
        ``{{ attribute['value']['@name'] }}``
        {%- else %}
            ``{{ attribute['value']['@name'] }}`` - {{ attribute['value']['@type'] }}
        {%- endif %}
            {%- if properties[referenceName][referenceName+".attr."+attribute['@name']+"."+attribute['value']['@name']+".value"] %}
            {{ properties[referenceName][referenceName+".attr."+attribute['@name']+"."+attribute['value']['@name']+".value"]|indent(4, true) }}
            {% endif %}
        {%- endif %}
        {%- else %}
        {%- if properties[referenceName][referenceName+".attr."+attribute['@name']+".value"] %}
        {{ properties[referenceName][referenceName+".attr."+attribute['@name']+".value"] }}
        {% endif %}
        {%- endif %}
    {%- endfor %}
    {#- handle case if we only have one attribute and it wasn't a list in the dict #}
    {%- else %}
    {#- for this attribute, print its name and type followed by its attribute value list if present #}
      *{{ capability['attribute']['@name'] }}:* {{ capability['attribute']['@type'] }}{% if capability['attribute']['@optional'] and capability['attribute']['@optional'] == "true" %} - Optional{% endif %}
	{%- if properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+".description"] %}
	{{ properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+".description"]|indent(2, true) }}
	{% endif %}
        {%- if capability['attribute']['value'] %}
        {%- if capability['attribute']['value'] is a_list %}
        {%- for value in capability['attribute']['value'] %}
	  {% if capability['attribute']['@type'] == 'ENUM' %}
	  ``{{ value['@name'] }}``
	  {%- else %}
	  ``{{ value['@name'] }}`` - {{ value['@type'] }}
	  {%- endif %}
          {%- if properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+"."+value['@name']+".value"] %}
          {{ properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+"."+value['@name']+".value"]|indent(4, true) }}
          {% endif %}
        {% endfor %}
        {%- else %}
        ``{{ capability['attribute']['value']['@name'] }}``
        {%- if properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+"."+capability['attribute']['value']['@name']+".value"] %}
        {{ properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+"."+capability['attribute']['value']['@name']+".value"]|indent(4, true) }}
        {% endif %}
        {%- endif %}
        {%- else %}
        {%- if properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+".value"] %}
        {{ properties[referenceName][referenceName+".attr."+capability['attribute']['@name']+".value"] }}
        {% endif %}
        {%- endif %}
    {%- endif %}
    {%- else %}
      None
    {%- endif %}

    {# Capability commands section #}
    Commands
    ^^^^^^^^
    {% if capability['command'] %}
    {#- check if the command key in dict is a list. It will not be a list if there is only one command #}
    {%- if capability['command'] is a_list %}
    {#- for each command, print its name method signature followed by its description #}
    {%- for command in capability['command'] %}
      *{{ command['@name'] }}({% if command['argument'] %}{% if command['argument'] is a_list %}{% for arg in command['argument'] %}{{ arg['@type'] }} {{ arg['@name'] }}{% if not loop.last %}, {% endif %}{% endfor %}{% else %}{{ command['argument']['@type'] }} {{ command['argument']['@name'] }}{% endif %}{% endif %})*
        {%- if properties[referenceName][referenceName+".cmd."+command['@name']+".description"] %}
          {{ properties[referenceName][referenceName+".cmd."+command['@name']+".description"] }}
        {% else %}
        {% endif %}
        {%- if command['argument'] %}
          {{ "Arguments:"|indent(2, true) }}
          {% if command['argument'] is a_list %}
            {% for arg in command['argument'] %}
              ``{{ arg['@name'] }}`` {% if arg['@required'] and arg['@required'] == "false" %}{% else %}*\*Required*{% endif %} - {{ arg['@type'] }}
              {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+".description"] %}
                {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+".description"]|indent(2, true) }}
              {%- endif %}
              {%- if arg['@type'] == 'ENUM' %}
                {%- if arg['value'] %}
                  {%- if arg['value'] is a_list %}
                    {%- for value in arg['value'] %}
                      ``{{ value['@name'] }}``
                      {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+"."+value['@name']+".value"] %}
                        {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+"."+value['@name']+".value"]|indent(2, true) }}
                      {%- endif %}
                    {%- endfor %}
                  {%- else %}
                    ``{{ arg['value']['@name'] }}``
                    {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+"."+arg['value']['@name']+".value"] %}
                      {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+arg['@name']+"."+arg['value']['@name']+".value"]|indent(2, true) }}
                    {% endif %}
                  {%- endif %}
                {%- endif %}
              {%- endif %}
              {%- if arg['component'] %}
                {%- if arg['component'] is a_list %}
                  {%- for component in arg['component'] %}
                    ``{{ component['@name'] }}`` - {{ component['@type'] }}
                    {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+component['@name']+".value"] %}
                      {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+component['@name']+".value"]|indent(2, true) }}
                    {%- endif %}
                  {%- endfor %}
                {%- endif %}
              {%- endif %}
            {% endfor %}
          {%- else %}
            ``{{ command['argument']['@name'] }}`` {% if command['argument']['@required'] and command['argument']['@required'] == "false" %}{% else %}*\*Required*{% endif %} - {{ command['argument']['@type'] }}
            {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+".description"] %}
              {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+".description"]|indent(2, true) }}
            {% else %}
            {% endif %}
            {%- if command['argument']['@type'] == 'ENUM' %}
              {%- if command['argument']['value'] %}
                {%- if command['argument']['value'] is a_list %}
                  {%- for value in command['argument']['value'] %}
                    ``{{ value['@name'] }}``
                    {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+value['@name']+".value"] %}
                      {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+value['@name']+".value"]|indent(2, true) }}
                    {% endif %}
                  {%- endfor %}
                {%- else %}
                  ``{{ command['argument']['value']['@name'] }}``
                  {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+command['argument']['value']['@name']+".value"] %}
                    {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+command['argument']['value']['@name']+".value"]|indent(2, true) }}
                  {%- endif %}
                {%- endif %}
              {%- endif %}
            {%- endif %}
            {%- if command['argument']['component'] %}
              {%- if command['argument']['component'] is a_list %}
                {%- for component in command['argument']['component'] %}
                  ``{{ component['@name'] }}`` - {{ component['@type'] }}
                  {%- if properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+component['@name']+".value"] %}
                    {{ properties[referenceName][referenceName+".cmd."+command['@name']+"."+command['argument']['@name']+"."+component['@name']+".value"]|indent(2, true) }}
                  {% endif %}
                {%- endfor %}
              {%- else %}
                {{ command['argument']['component']['@name']}}
              {%- endif %}
            {%- endif %}
          {%- endif %}
        {%- else %}
          {%- if properties[referenceName][referenceName+".cmd."+command['@name']+".value"] %}
            {{ properties[referenceName][referenceName+".cmd."+command['@name']+".value"] }}
          {% endif %}
        {%- endif %}
    {%- endfor %}
    {#- handle case if we only have one command and it wasn't a list in the dict #}
    {%- else %}
    {#- for this command, print its name method signature followed by its description #}
      *{{ capability['command']['@name'] }}({% if capability['command']['argument'] %}{% if capability['command']['argument'] is a_list %}{% for arg in capability['command']['argument'] %}{{ arg['@type'] }} {{ arg['@name'] }}{% if not loop.last %}, {% endif %}{% endfor %}{% else %}{{ capability['command']['argument']['@type'] }} {{ capability['command']['argument']['@name'] }}{% endif %}{% endif %})*
      {%- if properties[referenceName][referenceName+".cmd."+capability['command']['@name']+".description"] %}
        {{ properties[referenceName][referenceName+".cmd."+capability['command']['@name']+".description"] }}
      {% endif %}
      {%- if capability['command']['argument'] %}
    	{{ "Arguments:"|indent(2, true) }}
    	{% if capability['command']['argument'] is a_list %}
    	  {% for arg in capability['command']['argument'] %}
    		``{{ arg['@name'] }}`` {% if arg['@required'] and arg['@required'] == "false" %}{% else %}*\*Required*{% endif %} - {{ arg['@type'] }}
    		{%- if properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+arg['@name']+".description"] %}
    		  {{ properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+arg['@name']+".description"]|indent(2, true) }}
    		{% endif %}
    		{%- if arg['component'] %}
    		  {%- if arg['component'] is a_list %}
    			{%- for component in arg['component'] %}
    			  ``{{ component['@name'] }}`` - {{ component['@type'] }}
    			  {%- if properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+component['@name']+".value"] %}
    				{{ properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+component['@name']+".value"]|indent(2, true) }}
    			  {%- endif %}
    			{%- endfor %}
    		  {%- endif %}
    		{%- endif %}
    	  {% endfor %}
        {%- else %}
      	  ``{{ capability['command']['argument']['@name'] }}`` {% if capability['command']['argument']['@required'] and capability['command']['argument']['@required'] == "false" %}{% else %}*\*Required*{% endif %} - {{ capability['command']['argument']['@type'] }}
      	  {%- if properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+capability['command']['argument']['@name']+".description"] %}
      		{{ properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+capability['command']['argument']['@name']+".description"]|indent(2, true) }}
      	  {% endif %}
      	  {%- if capability['command']['argument']['component'] %}
      		{%- if capability['command']['argument']['component'] is a_list %}
      		  {%- for component in capability['command']['argument']['component'] %}
      			``{{ component['@name'] }}`` - {{ component['@type'] }}
      			{%- if properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+capability['command']['argument']['@name']+"."+component['@name']+".value"] %}
      			  {{ properties[referenceName][referenceName+".cmd."+capability['command']['@name']+"."+capability['command']['argument']['@name']+"."+component['@name']+".value"]|indent(2, true) }}
      			{%- endif %}
      		  {%- endfor %}
      		{%- else %}
      		  {{ capability['command']['argument']['component']['@name']}}
      		{%- endif %}
          {% endif %}
        {% endif %}
      {% endif %}
    {%- endif %}
    {%- else %}
      None
    {%- endif %}
    {% if not loop.last %}
    ----
    {%- endif %}
    {%- endfor %}
