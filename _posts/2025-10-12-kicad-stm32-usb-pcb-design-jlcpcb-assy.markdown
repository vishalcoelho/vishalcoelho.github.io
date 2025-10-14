---
layout: posts
title:  "First STM32 PCB Design in KiCad and JLCPCB Assembly"
date:   2025-10-12 14:26:00 -0500
categories: kicad pcb-design
hidden:true
---
<!-- excerpts go here, right after the front matter -->
These are my notes from following [Phil's Lab Tutorial](https://www.youtube.com/watch?v=C7-8nUU6e3E) on designing your
first PCB in KiCad 9.0 and using JLCPCB as the assembly house.

---

# Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
  - [JLCPCB PCB Prototyping](#jlcpcb-pcb-prototyping)
- [Software Tools](#software-tools)
- [Necessary Documentation](#necessary-documentation)
- [Deciding the Microcontroller's Pin Layout](#deciding-the-microcontrollers-pin-layout)
- [Creating a Schematic in KiCAD](#creating-a-schematic-in-kicad)
  - [The Microcontroller Circuitry](#the-microcontroller-circuitry)
    - [STM32F405](#stm32f405)
    - [Crystal Resonator](#crystal-resonator)
    - [Organize by Sections](#organize-by-sections)
  - [Power](#power)
- [Conclusion](#conclusion)

---

# Introduction

## JLCPCB PCB Prototyping
There are `basic` and `extended` parts on their [website][lnk_jlcpb]: `basic` meaning you could potentially use an
infinite number of these on the board while `extended` you pay a premium and you can use up to 10 of these (at the
time of writing).

![Basic and Extended Parts][img_basic_extended_parts]

> :memo: If you haven't already, create an account and sign in

---

# Software Tools

Download and install the following:
- [KiCad 9.0][lnk_kicad] - This is the PCB design software
- [STM32CubeIDE][lnk_stm32cubeide] - This is STMicro's IDE
- [SMT32CubeMx][lnk_stm32cubemx] - This is the pin layout tool

# Necessary Documentation
- [STM32F405xx Datasheet][lnk_stm32f405xx]
- [Buck Regulator][lnk_MP2359DJ]
- [USB Protection Diodes][lnk_usblc6]

--
# Deciding the Microcontroller's Pin Layout
The pins on our chosen micro have multiple functionality and we want to be able to plan out our pin layout to make
routing around the board more efficient. To that end,
1. Open STM32CubeMX
1. New Project -> Start My Project from MCU  -> Access to the MCU Selector
    ![New Project][img_stm32cubemx_new_project]

1. Type in the part number, `STM32F405` and select the part from the filtered list
    ![Part Selection][img_stm32cubemx_part_selection]
1. It opens the pin layout tool and we can begin to select the desired functionality of each pin
    ![Pin Layout][img_stm32cubemx_pin_layout]
1. We want the following functionality
   1. System Core
      1.  SYS -> Debug: Trace Asynchronous Sw (allows for printf debugging)
      1. RCC -> HSE: Crystal Resonator
      1. RCC -> LSE: Crystal Resonator (Real Time Clock Source)
   1. Connectivity
      1. USB_OTG_FS -> Device_Only
      1. I2C1 -> I2C
      1. USART3 -> Mode: Asynchronous
   1. Middleware & Software Packs
      1. USB_DEVICE -> Class for FS IP: Communication Device Class (Virtual COM port)

Finally, our chosen layout should look something like this,
![Final Pin Layout][img_stm32cubemx_pin_layout_final]

# Creating a Schematic in KiCAD
1. Create a new project: File -> New Project: STM32F4_Starter_Board
1. Open the Schematic Editor
1. File -> Page Settings: Edit the page settings to fill out the sheet description box in the lower right hand side of the page.

    ![Page Settings][img_kicad_sch_page_settings]

## The Microcontroller Circuitry
### STM32F405
1. Add the Microcontroller symbol to the schematic
    ![Add symbol][img_kicad_add_symbol]
1. Double click on the part name to edit: STM32F405RGTx -> STM32F405RGT6
1. Add power and ground using the (P) shortkey, use (W) to wire them
1. Add global labels (Ctrl + L) for NRST and BOOT0
1. Add capcitors using the Add Symbol (A), the `C_small` in the filter to find small capacitors
    ![Power and Ground][img_kicad_add_pwr_gnd]
1. We want to tie the BOOT0 net to a switch so we can decide whether to put the chip in run mode (pull down), or program mode (pull up)
    ![Boot Mode][img_kicad_boot_mode]
1. Add symbols for the crystal, USB, I2C, USART3 and SWD nets. Place No Connects (Q) at all unused pins
    > :bulb: You can place the first :x: and then hit the `Insert` key to key placing them in the next vertical grid point on the sheet. Careful not to cross out live connections.

    This is what it should look like
    ![All Symbols][img_kicad_all_symbols]

1. We want a bulk decoupling capacitor and smaller decouling capacitors on all our digital power rails, and some LC filtering on our analog rails
    ![Decoupling Capacitor][img_kicad_decoupling_caps]

### Crystal Resonator

We need to find a `basic` part on JLCPCB. If you search for 16 mHz, you come across this 4-pin crystal
![Crystal][img_jlcpcb_crystal]
If you look at the datasheet, you will see that pads 2 and 4 are ground, so when we add the symbol to KiCAD, we look for a `Crystal_GND24`. You need to add load caps and a feed resistor and then connect the HSE nets: `HSE_OUT` connects to the feed resistor and `HSE_IN` to the opposing pad.

![Oscillator][img_kicad_crystal]

To learn more about oscillator design, check out this guide from [ST][lnk_st_osc_design]

The load capacitance (what goes on either side of the crystal) is calculated as
$$
C_{load} = 2 \times (C_{L} - C{stray})
$$
where,


$C_{L}$ - load capacitance of the crystal (see crystal datasheet)
$C_{stray}$ - stray capacitance of the micro, probably 5 or 6uF.
> :memo: $C_{stray}$ is an estimated value

> :memo: The feed resistor isn't always needed; it limits how much current the micro drives into the crystal. Too high and you overdrive the crystal and generator harmonics (bad!!)

### Organize by Sections
Add the status LED circuit and cordon off what we've drawn with a rectangle in the lower left quadrant of the sheet. Mark it as the microcontroller section.

![Microcontroller Section][img_kicad_micro_section]

## Power
We are going to be using this [step-down switch mode converter][lnk_dc_dc] that JLCPCB supports. There is a schematic reference in the datasheet that we will follow.

KiCAD does not have this part so we are going to have to make it ourselves.

1. Click on the Create Symbols icon at the top to open the Symbol Editor
1. New Library -> KiCAD_Custom_Library
1. Browse to the newly created library int he `Libraries` pane
1. Right Click -> New Symbol -> Symbol Name: MP2359DJ-LF-Z
1. Open the Library Symbol Properties
    ![Library Properties][img_kicad_library_properties]
    and
    1.  add a new Fields, `Symbol` with the part name and check the `Show` box
    1. Set the footprint to TSOT-23-6
1. Using the `Draw Pins` (P) tool add the 6 pins to the part
1. Draw a bounding box (Rectangle) around the part

   ![Create New Part][img_kicad_create_new_symbol]


Our Buck regulator can take anywhere from 12-24V at the input, we will use 12V. We need to add a fuse and reverse polarity protection before we feed this in to `BUCK_IN`. We use a polyfuse (resetable temp based fuse). There is a neat tutorial on [Reverse Polarity Protection][lnk_rev_pol] on Youtube but we will use a P-MOSFET, [AO3401A][lnk_p_mosfet] available as a `basic` part on JLCPCB. We add a ferrite bead (very small inductor) and a bulk capacitor in series to smooth out transients into the buck.

We need >1.2V but less than the max 6V at the BUCK_EN pin, so we use a resistor divider from BUCK_IN to get our desired EN voltage. That takes care of everything on the left side of the chip.

Now we add a boost cap (10uF) between `BUCK_BST` and `BUCK_SW`. Next, we are going to be using the [B5819W Shottky diode][lnk_shottky] and an inductor at the switch (`BUCK_SW`) pin; the value of the inductor is specified in the datasheet of the buck converter.

Lastly, we add the feedback resistor divider chain. The value for these resistors can be calculated from the datasheet.

![Power Circuitry][img_kicad_power_circuitry]

# Conclusion


<!-- References -->
[lnk_jlcpb]: https://jlcpcb.com/
[lnk_kicad]: https://www.kicad.org/download/
[lnk_stm32cubemx]: https://www.st.com/en/development-tools/stm32cubemx.html#st-get-software
[lnk_stm32cubeide]: https://www.st.com/en/development-tools/stm32cubeide.html#st-get-software
[lnk_stm32f405xx]: https://www.st.com/resource/en/datasheet/stm32f405zg.pdf
[lnk_MP2359DJ]: https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8579708515633991680
[lnk_usblc6]: https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8579714554416455680
[lnk_st_osc_design]: https://www.st.com/resource/en/application_note/cd00221665-oscillator-design-guide-for-stm8af-al-s-stm32-mcus-and-mpus-stmicroelectronics.pdf
[lnk_dc_dc]: https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8579708515633991680
[lnk_rev_pol]: https://youtu.be/IrB-FPcv1Dc?si=IqBguDkFSW-D_ow1
[lnk_p_mosfet]: https://jlcpcb.com/partdetail/Alpha_OmegaSemicon-AO3401A/C15127
[lnk_shottky]: https://jlcpcb.com/partdetail/MDD_Microdiode_Semiconductor-B5819W/C64885

[img_basic_extended_parts]: /assets/images/2025_10_12_post_kicad_pcb_design/basic_extended_parts.png
[img_stm32cubemx_new_project]: /assets/images/2025_10_12_post_kicad_pcb_design/stm32cubemx_new_project.png
[img_stm32cubemx_part_selection]: /assets/images/2025_10_12_post_kicad_pcb_design/stm32cubemx_part_selection.png
[img_stm32cubemx_pin_layout]: /assets/images/2025_10_12_post_kicad_pcb_design/stm32cubemx_pin_layout.png
[img_stm32cubemx_pin_layout_final]: /assets/images/2025_10_12_post_kicad_pcb_design/stm32cubemx_pin_layout_final.png
[img_kicad_sch_page_settings]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_sch_page_settings.png
[img_kicad_add_symbol]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_add_symbol.png
[img_kicad_add_pwr_gnd]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_add_pwr_gnd.png
[img_kicad_boot_mode]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_boot_mode.png
[img_kicad_all_symbols]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_all_symbols.png
[img_kicad_decoupling_caps]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_decoupling_caps.png
[img_jlcpcb_crystal]: /assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_crystal.png
[img_kicad_crystal]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_crystal.png
[img_kicad_micro_section]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_micro_section.png
[img_kicad_create_new_symbol]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_create_new_symbol.png
[img_kicad_library_properties]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_library_properties.png
[img_kicad_power_circuitry]: /assets/images/2025_10_12_post_kicad_pcb_design/kicad_power_circuitry.png