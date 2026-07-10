---
title: Manhattan
---

# Introduction

This Tutorial explains how to build a [Manhattan Mobility
Model](https://en.wikipedia.org/wiki/Manhattan_mobility_model) in SUMO.
In this model, a fixed number of vehicles drive randomly on a Manhattan
grid network. Vehicles do not follow predefined routes, instead their movements at intersections are determined by turning probabilities. 
All required files can be found in the {{SUMO}}/docs/tutorial/manhattan directory.

# Creating the network

Creating Manhattan grid networks is supported by the
[netgenerate](../netgenerate.md) application. [netgenerate](../netgenerate.md) is a SUMO tool that automaticlly creates road networks. The option **--grid** creates
grid-shaped networks. The size of the grid can be configured with options such as **--grid.number**, which defines the number of grid cells. Further settings allow changing the cell size, number of lanes, and junction types.
For this tutorial, all network settings are stored in a configuration file (manhattan.netgcfg).
The network is created by calling

```
netgenerate -c manhattan/data/manhattan.netgcfg
```
This command generates the file net.net.xml, which contains the complete road network used in the later simulation steps.

# Generating vehicles

In this section we explain the concept; the actual vehicle generation follows below.

The vehicles in a Manhattan mobility model drive randomly according to
specified turning ratios, which define the probability of turning left, continuing straight, or turning right at intersections. This type of movement is supported by the
[jtrrouter](../jtrrouter.md) application. This application requires
`<flow>`-definitions as input, which specify where and when vehicles enter the network.
The actual routes are generated later by jtrrouter based on the turning ratios.

## Generating random flows for jtrrouter

In this step vehicle flows are created.
The [randomTrips.py](../Tools/Trip.md#randomtripspy) tool generates a flows.xml file
containing the starting points and departure times of the vehicles.

Run the following command
```
 <SUMO_HOME>/tools/randomTrips.py -n net.net.xml -o flows.xml --begin 0 --end 1 \
       --flows 100 --jtrrouter \
       --trip-attributes 'departPos="random" departSpeed="max"'
```
This creates 100 vehicles that enter the network at the beginning of the simulation.
The option **--jtrrouter** must be set to generate flows without destination. Otherwise the generated vehicles might end their trip too early. The option **--end 1** ensures that all vehicles are generated at the start of the simulation. The arguments supplied to option **--trip-attributes** are set to ensure that multiple vehicles may enter the source edge in the first step.

After running the command, a new file called flows.xml should be created. This file is used as input for the next step.

The options are also encoded in the script runner.py.

!!! caution
    The randomTrips option **--jtrrouter** is only available since SUMO version 1.2.0. In earlier versions, the 'to'-attribute must be removed manually from the generated flows before processing them with [jtrrouter](../jtrrouter.md).

## Calling jtrrouter

In this step [jtrrouter](../jtrrouter.md) uses generated vehicle flows to create routes through the Manhatten network. To generate sufficiently long routes, vehicles are allowed to make loops. Since no destination edges are defined, all destinations are accepted. The default turning ratios for the Manhattan Mobility Model (25% right, 50% straight, 25% left) are also defined.

All required options for this tutorial are written in the configuration file manhatten.jtrrcfg. 

Run [jtrrouter](../jtrrouter.md) with

```
jtrrouter -c manhattan/data/manhattan.jtrrcfg
```
This creates the vehicle routes that are used in the simulation.

## Remarks on Vehicle number

The number of vehicles in the first few simulation seconds is limited by
available road space for vehicle insertions. If the number of vehicles
is large with respect to the network size, it may take a few simulation
steps before all vehicles have entered the network.

## Making vehicles run forever

Using JTRRouter, routes of arbitrary length can be generated. However, vehicles will eventually reach the end of their route and exit the simulation. To avoid this, the tool [generateContinuousRerouters.py](../Tools/Misc.md#generatecontinuousrerouterspy) can be used.
