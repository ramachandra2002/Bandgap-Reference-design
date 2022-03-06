# Bandgap Reference design using Sky130 technology node

This repository contains the analysis and design of Bandgap IP Design using Sky130 technology node

![bgr_Banner](https://user-images.githubusercontent.com/89923461/156914849-f38073fb-23c6-4563-affe-b5a564a2c4f6.png)

# Day 1: Bandgap Design Theory

# Day 2:  BGR Labs and post-layout simulations

# Circuit Design Calculations

## 1. Current Calculation

- Maximum power consumption < 60 uW
- Maximum total current = 60 uW / 1.8 V = 33.34 uA


# Pre-Layout Simulation

To simulate these sub circuits, we are going to use netlist and simulate it using *ngspice*. To create a netlist, first create a file with a `filename.sp` extension. Use the first line as a comment to describe about the circuit for easier identification. First to create a valid spice netlist, we have to include a library with its path and its corner name.

```
.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
```

And if we going to use a specific model, we have to include that in the netlist ( in this case, we are using `sky130_fd_pr__pnp_05v5_W3p40L3p40 model`)

```
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"
```

The syntax which we are going to use for independent voltage/current sources is,

```
Vxx n1 n2 dc 1.8 : *Vxx* - Voltage source, *n1* - Node-1 of voltage source, *n2* - Node-2 of voltage source, *dc* - Type (can be dc/ac) *1.8* - value
Ixx n1 n2 dc 10u : *Ixx* - Current source, *n1* - Node-1 of current source, *n2* - Node-2 of current source, *dc* - Type (can be dc/ac) *1.8* - value
```
For simulating the design in DC the syntax is,

```
.dc temp -40 125 5 : Simulate for temp varying from -40 to 125 with 5 dec cent step
.dc Vs1 0 1.8 0.01 : Simulate for Vs1 varying from 0V to 1.8V with 0.01V step
```








