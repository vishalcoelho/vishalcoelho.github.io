---
layout: posts
title:  "First STM32 PCB Design in KiCad and JLCPCB Assembly: Part II - Board Layout"
date:   2025-11-02 20:08:00 -0500
categories: kicad pcb-design
hidden: false
---
<!-- excerpts go here, right after the front matter -->
These are my notes from following [Phil's Lab Tutorial](https://www.youtube.com/watch?v=C7-8nUU6e3E) on designing your first PCB in KiCad 9.0 and using JLCPCB as the assembly house.

---

# Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Board Setup](#board-setup)
- [Generating a Netlist](#generating-a-netlist)
- [Rough Layout](#rough-layout)
- [Fine Layout](#fine-layout)
  - [Decoupling Caps](#decoupling-caps)
  - [The MicroController](#the-microcontroller)
  - [Power Circuitry](#power-circuitry)
  - [Mounting Holes](#mounting-holes)
- [Board Shape](#board-shape)
- [Conclusion](#conclusion)


---

# Introduction

In Part 1 we ran the Electrical Rules Checker and resolved any outstanding issue with the placement or annotation of parts in our schematic. In this module, we will import the Netlist into our PCB Editor and lay out our board.

# Board Setup
1. Click the `Switch to PCB Editor` icon in the top ribbon to open the PCB Editor
1. Click the `Board Setup` icon to bring up the menu
1. We are going to be making a 4 layer board with a layer dedicated to ground and power each.
`Physical Stackup` -> `Copper Layers` -> 4

    ![Board Stackup][img_board_stackup]

1. Normally, you have to adjust the `Design rules` -> `Constraints` to match the manufacturers capability but KiCad's defaults are within range of [JLCPCB's][lnk_jlcpcb_pcb_drc].

1. Set the grid size to 1.0mm
    ![Grid Size][img_grid_setup]



# Generating a Netlist

> :memo: A `Netlist` is a file which describes electrical connections between symbol pins. These connections are referred to as nets.


1. Update the PCB from the schematic

    ![Update PCB from Schematic][img_update_pcb_from_schematic]
1. This will bring up the import menu. Accept the changes

    ![Import Menu][img_update_pcb_from_schematic_menu]

1. All the component footprints will be imported into your PCB

    > :memo: It looks like a rats nest

    ![Rats Nest][img_rats_nest]

# Rough Layout

- Hover over a part and hit `M` to move, `R` to rotate in place.
- Clicking on a footprint highlights ths component in the schematic, and vice-versa.

Once the parts are roughly laid out, it looks like this:

![Rough Placement][img_rough_placement]

> :warning: Note that the Top and Bottom Fabrication layers are hidden to prevent the component values from showing during placement.

# Fine Layout

Change the grid to 0.25mm before proceeding

## Decoupling Caps
Place each of the decoupling caps--the smaller ones first--near the VCC pins of the micro, one to each pin. Place the final bulk capacitor furthest from the power circuitry but as close to the micro as possible. Think of it as a large reservoir to supply the micro on the far side of the power source.

![Place Decoupling Caps][img_place_decoupling_caps]

## The MicroController
Place all the passives and connectors that hook into the micro near it
- pull-up resistors should be placed closer to the host device (micro) than the bus (connector)
- USB ESD protection should be closer to the connector than the micro; we are going to route the USB lines as a controlled impedance differential pair, which means the + and - traces need to be equal lengths! Placement needs to be symmetric.

![Placing the Microcontroller][img_place_micro]

## Power Circuitry

> :bulb: In Part 1 we placed the fuse right after the 12V screw terminal which essentially left the USB rail unfused. Move the fuse to after the 5V rail
![Fused Rails][img_fused_rails]
Now, update the PCB editor!

Keep the feedback loop on the regulator tight, traces need to be as short as possible as the device is switching in the MHz range.

![Power placement][img_place_power]

## Mounting Holes

1. Place the first mounting hole along the top right, in line with lower pads of the USB connecter (the USB connector will hang off the board slightly)
1. Take the next mounting hole and place it atop the first. Hit the `Space` key to zero out the `dx`, `dy`, `dist` measurements at the bottom of the IDE--you are effectively making this point the origin.
1. Move the mounting hole in just one direction, x or y, a certain distance away to mark the next corner of the board
1. In a similar fashion do the rest of the mounting holes using each mounting hole as a reference and only moving a certain distance in one axis to get to the next corner

    ![Place Mounting Holes][img_place_mounting]

# Board Shape

1. In the `Appearance` panel, under `Layers` click ont he `Edge.Cuts` layer to draw the board outline
1. Start with straight edges 5.5mm from the center of each mounting hole along each edge of the board. Follow up with arcs at each corner. Use the `Line` and `Curve` drawing tools respectively.
1. Now that we have the board outline move the connectors to the edge and space put the components appropriately. Open the `3D Viewer` to see how the board would look,
    ![Board Shape][img_board_shape]

# Conclusion
Our board looks reasonably well laid out and we are ready to wire up the nets in the next module.

<!-- References -->
[lnk_jlcpcb_pcb_drc]: https://jlcpcb.com/capabilities/pcb-capabilities

[img_update_pcb_from_schematic]: /assets/images/2025_10_12_post_kicad_pcb_design/update_pcb_from_schematic.png
[img_update_pcb_from_schematic_menu]: /assets/images/2025_10_12_post_kicad_pcb_design/update_pcb_from_schematic_menu.png
[img_rats_nest]: /assets/images/2025_10_12_post_kicad_pcb_design/rats_nest.png
[img_board_stackup]: /assets/images/2025_10_12_post_kicad_pcb_design/board_stackup.png
[img_grid_setup]: /assets/images/2025_10_12_post_kicad_pcb_design/grid_setup.png
[img_rough_placement]: /assets/images/2025_10_12_post_kicad_pcb_design/rough_placement.png
[img_place_decoupling_caps]: /assets/images/2025_10_12_post_kicad_pcb_design/place_decoupling_caps.png
[img_place_micro]: /assets/images/2025_10_12_post_kicad_pcb_design/place_micro.png
[img_fused_rails]: /assets/images/2025_10_12_post_kicad_pcb_design/fused_rails.png
[img_place_power]: /assets/images/2025_10_12_post_kicad_pcb_design/place_power.png
[img_place_mounting]: /assets/images/2025_10_12_post_kicad_pcb_design/place_mounting.png
[img_board_shape]: /assets/images/2025_10_12_post_kicad_pcb_design/board_shape.png