# Route model used in the 'OGC Routing Pilot'

## Overview
This repository documents the Route model that was developed and implemented in the [OGC Routing Pilot](https://www.ogc.org/projects/initiatives/routingpilot). OGC is the [Open Geospatial Consortium](https://www.ogc.org/).

The information in this repository is derived from the two reports that document the results of the pilot:

* [OGC Routing Pilot Engineering Report, OGC document 19-041r3](http://docs.opengeospatial.org/per/19-041r3.html)
* [Routing API Engineering Report, OGC document 19-040](http://docs.opengeospatial.org/per/19-040.html)

In addition, there are two presentations that provide an overview:

* [OGC Technical Committee, Transportation Ad-hoc, November 2019](191120-Slides-OGC-W3C-Transportation-Adhoc.pdf)
* [W3C Automotive and Transportation Working Group, March 2020](200324-Slides-W3C-Auto-F2F-Routing-API-and-Model.pdf)

This work is input to the [W3C Transportation Ontology Coordination Committee (TOCC)](https://github.com/w3c/tocc).

## Scenario
The pilot consisted of three scenarios. These were:

* Online - fully connected with stability
* Intermittent - unreliable connection
* Offline - no connectivity

### Online scenario
In the Online scenario, an operator uses a routing client (four different clients in the pilot) to request a route from a Routing API provider (four different API providers in the pilot), which in turn retrieves the route from an online routing engine (three different routing engines in the pilot). In this scenario all components have consistent connections between them and out to the wider internet.

This scenario uses the [Routing API](http://docs.opengeospatial.org/per/19-040.html#routing_api) for all request and response handling between the client and the routing infrastructure.

### Intermittent scenario
The intermittent scenario is where the components have connectivity, but it is not necessarily consistent, stable, reliable, or high-speed.

Therefore, the network cannot be relied upon to provide connectivity on demand and compensation actions are likely when connectivity is not available. Intermittent connectivity is unpredictable and it maybe that in the real-world, decisions are made to treat intermittent connectivity as no connectivity, as it is the only sensible course of action, especially if the scenario involves threat to life.

In the pilot, the intermittent scenario was described as a situation where only one of the clients had access to a routing engine. Therefore, the connected client had the ability to create routes, but none of the other clients did. Additionally, there were situations where the clients are able to communicate with each other via some other means (Bluetooth or some other peer-to-peer communication, for example). This scenario could also be a situation where a client had connectivity to a routing engine, but has lost it due to other reasons.

The solution proposed to address intermittent connectivity is to enable the clients to share pre-defined routes, that is, the routing operation has been completed when a connection to the routing engine was established, but has now been lost. 

Another approach to support this scenario is to not transmit the complete route definition to the client, but to return route information segment by segment as the vehicle moves along the route.

### Offline scenario
The Offline scenario assumed that there was no connectivity outside of a device’s local network, this could be a desktop computer, mobile device or a mesh. In the real-world the scenario is modeling an instance where there is no connectivity and there is not going to be any connectivity for the duration of an operation. 

An operator uses the routing functionality provided by the client to create a route. The operator then shares this route with other local clients using the route exchange model. To enable the required functionality, all of the capability has to be tightly coupled in a single location. Practically, this involves installing all of the components on the same machine to remove communication dependencies with the wider network.

## The Route model

![UML diagram](OGC-Open-Routing-Pilot-model.png "UML diagram")

### Destination
* a subtype of `Waypoint`
* a `Feature`
* constraints:
  * `type = 'end'`

### Route
* an `Object` 
* association role `describedBy`
  * multiplicity: 1
  * value: `RouteDefinition`
* attribute `end`
  * definition: The end point of the route.
  * multiplicity: 1
  * value: `Destination`
* attribute `name`
  * definition: Title of the route.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `overview`
  * multiplicity: 0..1
  * value: `RouteOverview`
* attribute `segments`
  * multiplicity: 0..*
  * value: `RouteSegment`
 * attribute `start`
  * definition: The start point of the route.
  * multiplicity: 1
  * value: `Start`
* attribute `status`
  * definition: Processing status of the route.
  * multiplicity: 1
  * default: accepted
  * values:
    * `accepted`: The route is queued for processing.
    * `running`: The route is being computed.
    * `successful`: The route is available.
    * `failed`: The route could not be computed.
* constraints:
  * `overview.duration=segments->collect(duration)->sum()`
  * `overview.length=segments->collect(length)->sum()`
  * `status=successful implies (overview->notEmpty() and segments->notEmpty())`

### RouteComponent
* a supertype of `RouteOverview`, `RouteSegment`, `Waypoint`
* a `Feature`
* is abstract
* attribute `type`
  * multiplicity: 1
  * values:
    * `start`
    * `end` 
    * `overview` 
    * `segment`

### RouteDefinition
* a `Object`
* definition: Information about the definition of the route. At a minimum, a route is defined by two waypoints, the start and end point of the route.
* attribute `algorithm`
  * definition: Select the routing / graph solving algorithm to use for calculating the route.
  * multiplicity: 0..1
  * value: `RoutingAlgorithm`
* attribute `dataset`
  * definition: The source dataset with a transport network for calculating the route.
  * note: In the pilot, the datasets were "NSG", "OSM", and "HERE".
  * multiplicity: 0..1
  * value: `SourceDataset`
* attribute `end`
  * multiplicity: 1
  * value: `Waypoint`
* attribute `engine`
  * definition: Select the routing engine to use for calculating the route.
  * note: In the pilot, the engines were "Skymantics", "Ecere", and "HERE".
  * multiplicity: 0..1
  * value: `RoutingEngine`
* attribute `intermediate`
  * definition: Additional waypoints along the route between start and end to consider when computing the route.
  * multiplicity: 0..*
  * value: `Waypoint`
* attribute `maxHeight`
  * definition: A height restriction for vehicles in meters to consider when computing the route.
  * multiplicity: 0..1
  * value: `Measure`
* attribute `maxWeight`
  * definition: A weight restriction for vehicles in tons to consider when computing the route.
  * multiplicity: 0..1
  * value: `Measure`
* attribute `obstacles`
  * definition: Areas the route should avoid.
  * note: The use of one or more polygons was a simple approach sufficient for the pilot. In general, the list of obstacles could also be a feature collection where every obstacle is a feature. Such a representation would be required, if the routing engine is able to handle obstacles with different characteristics/properties (for example, an obstacle is only valid for a certain time interval).
  * multiplicity: 0..1
  * value: `GM_MultiSurface`
* attribute `preference`
  * definition: The optimization goal for the route calculation (fastest, shortest, etc.). 
  * multiplicity: 1
  * default: `fastest`
  * values:
    * `fastest`
    * `shortest`
    * ...
* attribute `start`
  * multiplicity: 1
  * value: `Waypoint`
* attribute `temporal`
  * definition: The time of departure or arrival. The default value is an immediate departure.
  * multiplicity: 0..1
  * value: `TemporalConstraint`

### RouteOverview
* a subtype of `RouteComponent`
* a `Feature`
* attribute `comment`
  * definition: Explains any minor issues that were encountered during the processing of the routing request, i.e. any issues that did not result in an error.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `duration`
  * definition: Estimated amount of time required to travel the route (in seconds).
  * multiplicity: 1
  * value: `Measure`
* attribute `length`
  * definition: Length of the route (in meters).
  * multiplicity: 1
  * value: `Measure`
* attribute `maxHeight`
  * definition: A known height restriction on the route (in meters).
  * multiplicity: 0..1
  * value: `Measure`
* attribute `maxWeight`
  * definition: A known weight restriction on the route (in tons).
  * multiplicity: 0..1
  * value: `Measure`
* attribute `obstacles`
  * definition: Describes how obstacles were taken into account in the route calculation.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `path`
  * definition: The path from the start point to the end point of the route.
  * multiplicity: 1
  * value: `GM_Curve`
* attribute `processingTime`
  * definition: The time when the route was calculated.
  * multiplicity: 0..1
  * value: `DateTime`
* constraints:
  * `type = 'overview'`

### RouteSegment
* a subtype of `RouteComponent`
* a `Feature`
* attribute `duration`
  * definition: Estimated amount of time required to travel the segment (in seconds).
  * multiplicity: 1
  * value: `Measure`
* attribute `instructions`
  * definition: An instruction for the maneuver at the end of the segment.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `length`
  * definition: Length of the segment(in meters).
  * multiplicity: 1
  * value: `Measure`
* attribute `locationAtEnd`
  * definition: The last position of the segment and be on the path geometry of the route overview.
  * multiplicity: 1
  * value: `GM_Point`
* attribute `maxHeight`
  * definition: A known height restriction(in meters).
  * multiplicity: 0..1
  * value: `Measure`
* attribute `maxWeight`
  * definition: A known weight restriction (in tons).
  * multiplicity: 0..1
  * value: `Measure`
* association role `next`
  * multiplicity: 0..1
  * value: `RouteSegment`
* association role `prev`
  * multiplicity: 0..1
  * value: `RouteSegment`
* attribute `roadName`
  * definition: The road/street name of the segment.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `speedLimit`
  * definition: A known speed limit on the segment.
  * multiplicity: 0..1
  * value: `Measure`
* constraints:
  * `type = 'segment'`

### Start
* a subtype of `Waypoint`
* a `Feature`
* constraints:
  * `type = 'start'`

### TemporalConstraint
* a `Data type`
* attribute `timestamp`
  * multiplicity: 1
  * value: `DateTime`
* attribute `type`
  * multiplicity: 1
  * default: `departure`
  * values:
    * `departure`
    * `arrival `

### Waypoint
* a subtype of `RouteComponent`
* a supertype of `Destination`, `Start`
* a `Feature`
* definition: A waypoint of the route.
* attribute `location`
  * definition: The coordinates of the waypoint.
  * multiplicity: 1
  * value: `GM_Point`
* attribute `name`
  * definition: A name for the waypoint.
  * multiplicity: 0..1
  * value: `CharacterString`
* attribute `timestamp`
  * multiplicity: 0..1
  * value: `DateTime`
  
## Implementation of the Route model in GeoJSON

The OGC Routing Pilot used [GeoJSON](https://geojson.org/) to represent routes according to the model described above. 

A GeoJSON feature collection is used to represent the Route. Each RouteComponent is represented by a GeoJSON feature in the feature collection of the route. The encoding of route information in GeoJSON is specified in detail in the so-called [Route Exchange Model](http://docs.opengeospatial.org/per/19-041r3.html#RoutingExchangeModel).

GeoJSON has been used in the pilot for the following reasons:

* lightweight
* extensible
* widely supported in libraries and tools
* consistent with the emerging [OGC API standards](http://www.ogcapi.org/)
* an [open standard](https://tools.ietf.org/html/rfc7946)

The route [52n-here.json](52n-here.json) is a route generated by the Routing API of 52°North using the HERE routing engine. 

The example route also includes JSON-LD annotations to associate the JSON structures with GeoJSON and Route vocabularies. For use in the Routing API it is essential that a route is still valid GeoJSON.

  
