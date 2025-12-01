---
layout: posts
title:  "First STM32 PCB Design in KiCad and JLCPCB Assembly: Part III - Routing"
date:   2025-11-30 21:27:00 -0500
categories: kicad pcb-design
hidden: true
---
<!-- excerpts go here, right after the front matter -->
These are my notes from following [Phil's Lab Tutorial](https://www.youtube.com/watch?v=C7-8nUU6e3E) on designing your first PCB in KiCad 9.0 and using JLCPCB as the assembly house.

---

# Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Track Sizes](#track-sizes)
- [Power Planes](#power-planes)
- [Routing](#routing)
  - [MicroController](#microcontroller)
  - [Power Circuitry](#power-circuitry)
  - [USB](#usb)
  - [Ground and Power Nets](#ground-and-power-nets)
  - [Crystal Needs a Keepout](#crystal-needs-a-keepout)
- [Design Rule Check](#design-rule-check)
- [Conclusion](#conclusion)


---

# Introduction

In Part 2 we laid out the components on our PCB and finalized the shape of the board. In this module we wire up the nets.

# Track Sizes

In the PCB Editor add the following track sizes

 ![Add Wire Sizes][img_add_wire_sizes]

| Size | Need|
|:-----|-----|
| 0.3mm| LQFP pins of the micro|
| 0.5 - 1mm | Power |

# Power Planes
This is a four layer board and we will use the inner two layers as power and ground.

We will start with `Ground` on `In1.Cu`.

1. Click on the `Draw Filled Zones` icon
2. Click somewhere to the top left of the board to bring up the properties
3. Select the `In1.Cu` layer
4. Choose the `GND` net, default values for this menu are OK
    ![Ground Fill][img_ground_fill]
5. Draw a rectangle around the board
6. `Edit` -> `Fill all Zones`

Click the `Draw Zone Fills` icon to see the ground layer fill. Notice how the ground pads are connected to the plane through thermal reliefs.

We will do the same with the Power plane

1. Select the existing rectangle
2. Hit `Ctrl+D` then click to make a copy of the rectangle
> :warning: Note that this rectrangle will be defined for the same layer and net, `In1.Cu` and `GND`, until you change it
3. With the new rectangle selected, hit `E` to bring up the properties
4. Change the layer to `In2.Cu` and the net to `+3.3V`
5. `Edit` -> `Fill all Zones`
    ![Power Plane][img_power_fill]

# Routing

Lets start by turning off the Zone fills to make it easier to see things. In the left ribbon of the PCB editor hit `Draw Zone Outlines`.

Start with the critical components: decoupling caps, crystal, high-speed traces, etc.

## MicroController

1. Choose the 0.3mm track size
2. Set the Grid size to 0.25mm
3. Select the `F.Cu` (top copper) layer
4. Start routing by hovering over the pad, hitting `X` and making the connection
5. As you are dragging out a trace hitting `V` will change the cursor to a via, click to place it and automatically switch to the bottom layer trace. Placing another via will switch it back to the top layer.
    ![Microcontroller Routing][img_route_micro]

## Power Circuitry

1. Instead of using larger (say, 1.0mm) track we will use polygon fills for most of the nets
2. For example, the `BUCK_SW` net, we will use the `Draw Filled Zones` tool to circumscribe the net in a polygon that we fill with copper (F.Cu)
    ![Zone filling a net][img_net_zone]

The power circuitry section looks like this

![Power Section][img_power_routing]

Power and ground pads will be routed to the power planes with vias.

## USB
We will use [JLCPCB's impedance calculator tool][lnk_impedance_calc] to figure out how to set up our USB differential pair.

> :memo: We need to specify our units in mils (1 mil = 1/1000th of an inch).

![Impedance Calculator][img_jlcpcb_impedance_calc]

> :memo: I went with the 7628 process as i assume this is what Phil was referencing in the video, and it gave me the closest trace width to his calculation.

Back in the PCB editor select the Track drop-down menu and select `Edit Pre-Defined Sizes...`. Add in the calculated values in `mil`s and the tools will convert to mm.

![Differential Pair Sizing][img_differential_track_size]

Next, click on `Route` -> Differential Pair` and then right click in the editor to bring up the context menu, `Select Differential Pair Dimensions` and choose the diff width we just defined

![Choosing the Diff Pair Size][img_use_diff_pair_size]

Now route the D+/D- pair from USB connector to the ESD chip and onto the micro

![Routing Differential Pair][img_route_diff_pair]

## Ground and Power Nets

Finally, we will connect the `+3V3` and `GND` nets to their respective power planes with vias

1. Track drop-down menu -> Edit Pre-Selected... -> Add via sizing
    ![Via Sizing][img_via_sizes]
1. Select the `Via` tool and drop one right atop a `+3V3` pad, it will adopt the net of the pad
2. Drag the via just outside the pad
    > :warning: vias on pads will cause solder to get sucked through. Might end up with a dry joint
3. Repeat this for all the `+3V3` pads
4. Link all the vias with 0.5mm traces to the pads
5. Power circuitry section gets extra vias per pad to ensure minimal parasitic inductance.

## Crystal Needs a Keepout

We need to exclude the ground plane pour underneath the crystal section, we do this by:
1. selecting the `Draw Rules Area` tool
2. creating a polygon on the `In2.Cu` layer to keep out zone fills
3. draw the polygon around the desired area (Xtal and passives).

    ![Crystal Keepout Zone][img_crystal_keepout]

4. `Edit` -> `Fill all Zones` (or Hit `B`)
5. Attach the ground pads to other ground pads or through vias with thin traces (0.3 - 0.5mm)
6. Power circuitry gets extra vias per pad.

We are now done routing and ready to move onto the Design Rule Check

![Done Routing][img_done_routing]

# Design Rule Check

# Conclusion


<!-- References -->
[lnk_impedance_calc]: https://jlcpcb.com/pcb-impedance-calculator

[img_add_wire_sizes]: /assets/images/2025_10_12_post_kicad_pcb_design/add_wire_sizes.png
[img_ground_fill]: /assets/images/2025_10_12_post_kicad_pcb_design/ground_fill.png
[img_show_zone_fill]: /assets/images/2025_10_12_post_kicad_pcb_design/show_zone_fill.png
[img_power_fill]: /assets/images/2025_10_12_post_kicad_pcb_design/power_fill.png
[img_route_micro]: /assets/images/2025_10_12_post_kicad_pcb_design/route_micro.png
[img_net_zone]: /assets/images/2025_10_12_post_kicad_pcb_design/net_zone.png
[img_power_routing]: /assets/images/2025_10_12_post_kicad_pcb_design/power_routing.png
[img_jlcpcb_impedance_calc]: /assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_impedance_calc.png
[img_differential_track_size]: /assets/images/2025_10_12_post_kicad_pcb_design/differential_track_size.png
[img_use_diff_pair_size]: /assets/images/2025_10_12_post_kicad_pcb_design/use_diff_pair_size.png
[img_route_diff_pair]: /assets/images/2025_10_12_post_kicad_pcb_design/route_diff_pair.png
[img_via_sizes]: /assets/images/2025_10_12_post_kicad_pcb_design/via_sizes.png
[img_crystal_keepout]: /assets/images/2025_10_12_post_kicad_pcb_design/crystal_keepout.png
[img_done_routing]: /assets/images/2025_10_12_post_kicad_pcb_design/done_routing.png