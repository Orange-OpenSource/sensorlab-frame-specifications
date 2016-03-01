# SensorLab log messages documentation

| *document* | SensorLab log messages documentation                         |
|:----------:|:-------------------------------------------------------------|
| *version*  |  1.1                                                         |
| *date*     |  February, 26th 2016                                         |
| *author*   |  Quentin Lampin, <quentin.lampin@orange.com>                 |
| *author*   |  Dominique Barthel, <dominique.barthel@orange.com>           |


## Brief
This document specifies the format of the messages used in SensorLab to debug and monitor
the behavior of the nodes within a network.

## Table of Contents

|               Sections                                            |
|:-----------------------------------------------------------------:|
| [Encoding & Endianness](#encoding-endianness)                     |
| [Reference model](#reference-model)                               |
| [Generic Frame Format](#generic-frame-format)                     |
| [EventID Hexadecimal Codes](#eventid-hexadecimal-codes)           |
| [Node Events Payload Format](#node-events-payload-format)         |
| [Entity Events Payload Format](#entity-events-payload-format)     |
| [Link Events Payload Format](#link-events-payload-format)         |
| [Frame Events Payload Format](#frame-events-payload-format)       |
| [Properties Payload Formats](#properties-payload-formats)         |
| [Properties Units and Unit Prefixes](#properties-units-prefixes)  |


## Encoding & Endianness

Unless specified otherwise and when applicable,

  - numerical fields are transmitted as **little-endian unsigned integers**
  - character arrays are encoded in **US-ASCII**

## Reference model
        _____                    log messages
       |     | --------------------------------------------+
       |node1|   _____                                     |
       |_____|  |     | -----------------------------------+----> monitoring station
                |node2|     _____________________          |
                |_____|    | node3               | --------+
                           |   _______           |
                           |  |entity1|          |
                           |  |_______|          |
                           |   ___|___           |
                           |  |entity2|          |
                           |  |_______|          |
                           |   ___|___           |
                           |  |entity3|rx        |
                           |  |_______|tx        |
                           |_____________________|

Our goal is to observe the behavior of a network of nodes.
We assume that, to that purpose, nodes voluntarily send log messages to a monitoring station in addition to
carrying their usual communication activity.

This specification describes the format of the multiplexed flow of log messages as it reaches the
monitoring station.  Whether these log messages reach the monitoring station
over a dedicated monitoring network or over the regular data network is immaterial to this specification.

The log messages report about events that happen within nodes.

Because nodes usually have complex behaviors, and in order to assist with the classification and visualization
of events, our representation further decomposes nodes into entities that communicate between one another within a node.
Some entities can communicate with similar entities in other nodes, like a radio transceiver would in real life.

Our representation defines a link as a relationship from a given entity of one node to the same entity of another node.

This specification defines a taxonomy of events pertaining to the creation of nodes and entities, to the change of their
internal state, to the communication between entities within a node, to the communication between nodes and to the appearence and
disappearance of links as well as their state changes.

## Generic Frame Format

SensorLab log messages are composed of a header comprising the *node identifier* and the *event identifier*, and an *event payload*.

| `nodeID` | `eventID` |    `eventPayload`    |
|:--------:|:---------:|:--------------------:|
|  32bits  |   8bits   |    variable size     |

where:

  - `nodeID`        :   ID of the node that produced this log message
  - `eventID`       :   ID of the event this log message reports about
  - `eventPayload`  :   further description of the event (dependant on the eventID value)


## EventID hexadecimal codes

SensorLab events are classified in 19 types grouped in 4 categories (node, entity, link and frame events).
The table below provides the hexadecimal value corresponding to each SensorLab event:

|   Event name                      | Hexadecimal value (1 byte) |
|:-------------------------------   |:--------------------------:|
| `EVENT_NODE_ADD`                  |           0x00             |
| `EVENT_NODE_PROPERTY_ADD`         |           0x01             |
| `EVENT_NODE_PROPERTY_UPDATE`      |           0x02             |
| `EVENT_NODE_REMOVE`               |           0x03             |
|                                   |                            |
| `EVENT_ENTITY_ADD`                |           0x10             |
| `EVENT_ENTITY_PROPERTY_ADD`       |           0x11             |
| `EVENT_ENTITY_PROPERTY_UPDATE`    |           0x12             |
| `EVENT_ENTITY_REMOVE`             |           0x13             |
|                                   |                            |
| `EVENT_LINK_ADD`                  |           0x20             |
| `EVENT_LINK_PROPERTY_ADD`         |           0x21             |
| `EVENT_LINK_PROPERTY_UPDATE`      |           0x22             |
| `EVENT_LINK_REMOVE`               |           0x23             |
|                                   |                            |
| `EVENT_FRAME_PRODUCE`             |           0x30             |
| `EVENT_FRAME_PROPERTY_ADD`        |           0x31             |
| `EVENT_FRAME_PROPERTY_UPDATE`     |           0x32             |
| `EVENT_FRAME_DATA_UPDATE`         |           0x33             |
| `EVENT_FRAME_TX`                  |           0x34             |
| `EVENT_FRAME_RX`                  |           0x35             |
| `EVENT_FRAME_CONSUME`             |           0x36             |


## Node Events Payload Formats

Node events describe the life cycle of nodes in the network. As of current revision, there are 4 types of such events:

  - `EVENT_NODE_ADD`                 :   creation of a new node in the network
  - `EVENT_NODE_PROPERTY_ADD`        :   addition of state variables/properties to the node
  - `EVENT_NODE_PROPERTY_UPDATE`     :   update of the value of state variables/properties of the node
  - `EVENT_NODE_REMOVE`              :   removal of a node from the network

`EVENT_NODE_ADD`, `EVENT_NODE_PROPERTY_ADD` and `EVENT_NODE_PROPERTY_UPDATE` payloads structure is:

| `propertiesCount` | `properties [...]` |
|:-----------------:|:------------------:|
|        8bits      |   variable size    |


`EVENT_NODE_REMOVE` payload is empty.

where:

  - `propertiesCount`   :   number of properties describing the node state
  - `properties [...]`  :   properties describing the node state, described in section [Properties Payload Format](#properties-payload-formats)

## Entity Events Payload Format

Entity events describe the life cycle of either hardware or software "modules" of the nodes. As of current revision, there are 4 types of such events:

  - `EVENT_ENTITY_ADD`               :   addition of a new entity (in most cases, at the bootstrap of said entity)
  - `EVENT_ENTITY_PROPERTY_ADD`      :   addition of state variables/properties to the entity
  - `EVENT_ENTITY_PROPERTY_UPDATE`   :   update of the value of a state variable/property of the entity
  - `EVENT_ENTITY_REMOVE`            :   removal of an entity (e.g. in case the software module is swapped)

`EVENT_ENTITY_ADD` payload structure is:

| `entityID` | `entityNameLength` | `propertiesCount` |   `entityName`  | `properties [...]` |
|:----------:|:------------------:|:-----------------:|:---------------:|:------------------:|
|    8 bits  |      8 bits        |    8 bits         | variable size   |  variable size     |

`EVENT_ENTITY_PROPERTY_ADD` and `EVENT_ENTITY_PROPERTY_UPDATE` payload structure is:

| `entityID` | `propertiesCount` | `properties [...]` |
|:----------:|:-----------------:|:------------------:|
|    8 bits  |       8 bits      |   variable size    |

`EVENT_ENTITY_REMOVE` payload structure is:

| `entityID` |
|:----------:|
|    8 bits  |

where:

  - `entityID`          :   ID of the entity that this event relates to
  - `entityNameLength`  :   entity's name length
  - `propertiesCount`   :   number of properties describing the entity state
  - `entityName`        :   entity's name (a character array)
  - `properties`        :   properties describing the entity state, see section [Properties Payload Formats](#properties-payload-formats)

## Link Events Payload Format

Link events relate to the "relations" that are built between identical entities of different nodes. For example, a node could report about a neighbor relationship from itself to another node as seen by its Link layer. However, to accomodate more complex cases, the node reporting about the link can be different from the nodes at either end of the link.

As of current revision, there are 4 types of such events, similar to that of entity events:

  - `EVENT_LINK_ADD`                 :   addition of a new link to the network
  - `EVENT_LINK_PROPERTY_ADD`        :   addition of state variables/properties to the link
  - `EVENT_LINK_PROPERTY_UPDATE`     :   update of the value of state variables/properties of the link
  - `EVENT_LINK_REMOVE`              :   removal of a link

`EVENT_LINK_ADD` event payload structure is:

| `entityID` | `linkID` | `sourcePropertiesCount` | `targetPropertiesCount` | `linkPropertiesCount` | `sourceProperties [...]` | `targetProperties [...]` | `linkProperties [...]` |
|:----------:|:--------:|:-----------------------:|:-----------------------:|:---------------------:|:------------------------:|:------------------------:|:----------------------:|
|   8 bits   |  8 bits  |          8 bits         |           8 bits        |          8 bits       |       variable size      |      variable size       |      variable size     |



`EVENT_LINK_PROPERTY_ADD` and `EVENT_LINK_PROPERTY_UPDATE` payloads structure is:

| `entityID` | `linkID` | `linkPropertiesCount` | `linkProperties [...]` |
|:----------:|:--------:|:---------------------:|:----------------------:|
|    8 bits  |   8 bits  |        8 bits        |      variable size     |

`EVENT_LINK_REMOVE` payload structure is:

| `entityID` | `linkID` |
|:----------:|:--------:|
|  8 bits    |  8 bits  |


where:

  - `entityID`              :   ID of the entity that this event relates to
  - `linkID`                :   ID of the link, unique in the scope of the node and entity reporting about this link
  - `sourcePropertiesCount` :   number of properties provided to identify the source end of the link
  - `targetPropertiesCount` :   number of properties provided to identify the target end of the link
  - `linkPropertiesCount`   :   number of properties describing the link
  - `sourceProperties`      :   properties to be matched by the source end of the link, described below in section [Properties Payload Formats](#properties-payload-formats)
  - `targetProperties`      :   properties to be matched by the target end of the link, described below in section [Properties Payload Formats](#properties-payload-formats)
  - `linkProperties`        :   properties of the link, see section [Properties Payload Formats](#properties-payload-formats)

## Frame Events Payload Format

Frame events describe the life-cycle of "frames" in a node.

In our representation, a "frame" is a piece of information that is produced within a node by a given entity or received at a network interface, modified by subsequent transit through other entities within the same node and eventually transmitted to another node or consumed within the node. For example, a frame would represent a message generated by a chat application, that is subsequently encapsulated by the XMPP protocol, then UDP, then WiFi and transmitted by the hardware entity (here radio) of the node to one of its neighbour.

This notion of "frame" stems from the customary way of altering data "in place" within framebuffers in hardware resource constrained nodes.

As of current revision, there are 7 types of frame events:

  - `EVENT_FRAME_PRODUCE`            :   ex nihilo creation of a new frame within the node.
  - `EVENT_FRAME_PROPERTY_ADD`       :   addition of a property to the frame.
  - `EVENT_FRAME_PROPERTY_UPDATE`    :   update of the value of a property of the frame.
  - `EVENT_FRAME_DATA_UPDATE`        :   update of the frame data, e.g. encapsulation occurred at a given entity within a node.
  - `EVENT_FRAME_TX`                 :   emission of the frame, i.e. radio-emitted with the intent to reach another node.
  - `EVENT_FRAME_RX`                 :   creation of a frame at this node by receiving some data from another node.
  - `EVENT_FRAME_CONSUME`            :   consumption of the frame, either because the current node/entity is the final recipient of the information or because the current node/entity discards the frame, e.g. destination address mismatch.

`EVENT_FRAME_PRODUCE` and `EVENT_FRAME_RX` payload structure is:

| `entityID` | `frameID` |  `dataLength` | `propertiesCount` |     `data`      |  `frameProperties [...]` |
|:----------:|:---------:|:-------------:|:-----------------:|:---------------:|:------------------------:|
|    8 bits  |   8 bits  |    16 bits    |        8 bits     |  variable size  |      variable size       |

`EVENT_FRAME_PROPERTY_ADD` and `EVENT_FRAME_PROPERTY_UPDATE` payload structure is:

| `entityID` | `frameID` | `propertiesCount` |  `frameProperties [...]` |
|:----------:|:---------:|:-----------------:|:------------------------:|
|   8 bits   |  8 bits   |       8 bits      |      variable size       |

`EVENT_FRAME_DATA_UPDATE`, `EVENT_FRAME_TX` and `EVENT_FRAME_CONSUME`  payload structure is:

| `entityID` | `frameID` |  `dataLength` |     `data`      |
|:----------:|:---------:|:-------------:|:---------------:|
|    8 bits  |   8 bits  |    16 bits    |  variable size  |

where:

  - `entityID`              :   ID of the entity that this event relates to
  - `frameID`               :   ID given by NodeID to the frame, unique in the scope of all entities belonging to nodeID
  - `dataLength`            :   length of the included data.
  - `data`                  :   data in its current state (byte array), size **must** match that of dataLength


## Properties Payload Formats

Events payloads allow for an arbitrary number of properties.
The intent of such properties is to provide information on the entity/link/frame being described, e.g. the radio entity TX power is set to 0dBm.
In most cases, it is not mandatory to provide such properties, link events being the exception. Link events indeed require to provide information on the target and/or source of the link, i.e. source and target properties. This information is necessary to match the identities of the node pair of the link. In case either the source or the target properties are missing, `nodeID` is assumed for the corresponding end of the link.


### Properties Addition vs. Properties Update

2 types of property payload exist: the `Property Declaration Payload` and the `Property Reference Payload`.
Using either one or the other is governed by the following policies:

  - Whenever a new object (node, entity, link, frame) is declared, properties describing said object **must** be declared with the `Property Declaration Payload`.
  - Whenever an existing object (node, entity, link, frame) is described by a property that is **new to said object**, the property **must be declared** with the `Property Declaration Payload`.
  - Whenever an existing object (node, entity, link, frame) property is **updated**,  the `Property Reference Payload` **must** be used.
  - Whenever an event description relies on a reference to an **already declared property**,  the `Property Reference Payload` **must** be used.

**Rule of Thumb**:

*Properties describing the object that is declared in events post-fixed with `_ADD` or `_PRODUCE` must be declared with the `Property Declaration Payload`.*

**Warning**:

*When declaring a new link, i.e. `EVENT_LINK_ADD`, **properties describing the source and target of the link do not refer to the link itself** but to the ends of said link, i.e. nodes' entities. These properties **must be already declared** in said nodes' entities and are therefore referred to using the `Property Reference Payload`.*

### Property Declaration Payload

The Property Declaration Payload structure is:

| `propertyID` | `unitPrefix` | `unit` | `dataType` |  `propertyNameLength` | `propertyValueLength` | `propertyName` | `propertyValue` |
|:------------:|:------------:|:------:|:----------:|:---------------------:|:---------------------:|:--------------:|:---------------:|
|    8 bits    |    8bits     |  8bits |    8bits   |         8 bits        |       16 bits         |  variable size |  variable size  |

 where:

  - `propertyID`            :   ID of the property, unique in the scope of the object described by the property, e.g. node, entity, link, frame.
  - `unitPrefix`            :   prefix of the unit, e.g. the **m** of **m**W
  - `unit`                  :   unit used to express the property
  - `dataType`              :   the property value formatting, e.g. float, uint16, etc.
  - `propertyNameLength`    :   length of the property's name
  - `propertyValueLength`   :   property's value length, **must be equal** to the `dataType` length unless `dataType` is `TYPE_ASCII_ARRAY` or `TYPE_BYTE_ARRAY`. In that case, it **must be equal** to the byte length of the array.
  - `propertyName`          :   property's name, ASCII encoded
  - `propertyValue`         :   property's value, dataType encoded

### Property Reference Payload

Property Reference Payload structure is:

| `propertyID` | `propertyValueLength` | `propertyValue` |
|:------------:|:---------------------:|:---------------:|
|     8 bits   |       16 bits         |  variable size  |

where:

  - `propertyID`            :   ID of the property, unique in the scope of the object described by the property, e.g. node, entity, link, frame.
  - `propertyValueLength`   :   property's value length, **must be equal** to the `dataType` length unless `dataType` is `TYPE_ASCII_ARRAY` or `TYPE_BYTE_ARRAY`. In that case, it **must be equal** to the bytes length of the array.
  - `propertyValue`         :   property's value, dataType encoded


### Properties Units & Prefixes

As of current revision, property values can be expressed in 29 different units, e.g. watts, newtons, etc.
The table below provides the hexadecimal values corresponding to units available in the SensorLab frames:

|       Unit name         |  Hexadecimal value (1byte) |
|:------------------------|:--------------------------:|
|    `UNIT_NONE`          |            0x00            |
|    `UNIT_METRE`         |            0x01            |
|    `UNIT_KILOGRAM`      |            0x02            |
|    `UNIT_SECOND`        |            0x03            |
|    `UNIT_AMPERE`        |            0x04            |
|    `UNIT_KELVIN`        |            0x05            |
|    `UNIT_MOLE`          |            0x06            |
|    `UNIT_CANDELA`       |            0x07            |
|    `UNIT_RADIAN`        |            0x08            |
|    `UNIT_STERADIAN`     |            0x09            |
|    `UNIT_HERTZ`         |            0x0A            |
|    `UNIT_NEWTON`        |            0x0B            |
|    `UNIT_PASCAL`        |            0x0C            |
|    `UNIT_JOULE`         |            0x0D            |
|    `UNIT_WATT`          |            0x0E            |
|    `UNIT_COULOMB`       |            0x0F            |
|    `UNIT_VOLT`          |            0x10            |
|    `UNIT_FARAD`         |            0x11            |
|    `UNIT_OHM`           |            0x12            |
|    `UNIT_SIEMENS`       |            0x13            |
|    `UNIT_WEBER`         |            0x14            |
|    `UNIT_TESLA`         |            0x15            |
|    `UNIT_HENRY`         |            0x16            |
|    `UNIT_DEGREECELSIUS` |            0x17            |
|    `UNIT_LUMEN`         |            0x18            |
|    `UNIT_LUX`           |            0x19            |
|    `UNIT_BECQUEREL`     |            0x1A            |
|    `UNIT_GRAY`          |            0x1B            |
|    `UNIT_SIEVERT`       |            0x1C            |
|    `UNIT_KATAL`         |            0x1D            |
|    `UNIT_DB`            |            0x1E            |
|    `UNIT_DBW`           |            0x1F            |
|    `UNIT_DBMW`          |            0x20            |

When applicable, units can be prefixed to represent different ranges of values, e.g. **m**W.
The table below provides the hexadecimal values corresponding to unit prefixes available in the SensorLab frames:

|       Prefix name        |  Hexadecimal value (1byte) |
|:-------------------------|:--------------------------:|
|     `PREFIX_YOTTA`       |            0x00            |
|     `PREFIX_ZETTA`       |            0x01            |
|     `PREFIX_EXA`         |            0x02            |
|     `PREFIX_PETA`        |            0x03            |
|     `PREFIX_TERA`        |            0x04            |
|     `PREFIX_GIGA`        |            0x05            |
|     `PREFIX_MEGA`        |            0x06            |
|     `PREFIX_KILO`        |            0x07            |
|     `PREFIX_HECTO`       |            0x08            |
|     `PREFIX_DECA`        |            0x09            |
|     `PREFIX_NONE`        |            0x0A            |
|     `PREFIX_DECI`        |            0x0B            |
|     `PREFIX_CENTI`       |            0x0C            |
|     `PREFIX_MILLI`       |            0x0D            |
|     `PREFIX_MICRO`       |            0x0E            |
|     `PREFIX_NANO`        |            0x0F            |
|     `PREFIX_PICO`        |            0x10            |
|     `PREFIX_FEMTO`       |            0x11            |
|     `PREFIX_ATTO`        |            0x12            |
|     `PREFIX_ZEPTO`       |            0x13            |
|     `PREFIX_YOCTO`       |            0x14            |


### Properties Data types

Properties values are expressed in one of the 13 data types available:


|     Data type name       |  Hexadecimal value (1byte) |
|:-------------------------|:--------------------------:|
|     `TYPE_BOOLEAN`       |            0x00            |
|     `TYPE_INT8`          |            0x01            |
|     `TYPE_INT16`         |            0x02            |
|     `TYPE_INT32`         |            0x03            |
|     `TYPE_INT64`         |            0x04            |
|     `TYPE_UINT8`         |            0x05            |
|     `TYPE_UINT16`        |            0x06            |
|     `TYPE_UINT32`        |            0x07            |
|     `TYPE_UINT64`        |            0x08            |
|     `TYPE_FLOAT`         |            0x09            |
|     `TYPE_DOUBLE`        |            0x0A            |
|     `TYPE_ASCII_ARRAY`   |            0x0B            |
|     `TYPE_BYTE_ARRAY`    |            0x0C            |
|     `TYPE_INVALID`       |            0x0D            |


