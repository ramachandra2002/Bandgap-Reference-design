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

And if we going to use a specific model, we have to include that in the netlist ( in this case, we are using **sky130_fd_pr__pnp_05v5_W3p40L3p40 model**)

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

## CTAT Simulation

### CTAT Voltage generation with single BJT

The aim of this simulation is to find the voltage variation across BJT with respect to the temp if use current source of 10uA. The spice netlist which we are going to simulate is, 

```
**** ctat voltage generation circuit *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** bjt definition
xqp1	gnd	gnd	qp1	gnd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1

*** supply voltage and current
vsup	vdd	gnd	dc	2
isup	vdd	qp1	dc 	10u
.dc	temp	-40	125	5

*** control statement
.control
run
plot v(qp1)
.endc
.end

```

And to simulate this netlist, type `ngspice ctat_voltage_gen.sp`. Now we the see the corresponding waveform after the simulation

![c2_Picture1](https://user-images.githubusercontent.com/89923461/156916584-e521a3b1-a3fa-4039-8d92-e252f5ca2a2e.png)


### CTAT Voltage genration with multiple BJTs

 The spice netlist which we are going to simulate is, 

```
**** ctat voltage generation circuit with multiple bjt *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** bjt definition
xqp1	gnd	gnd	qp1	gnd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=8

*** supply voltage and current
vsup	vdd	gnd	dc	2
isup	vdd	qp1	dc 	10u
.dc	temp	-40	125	5

*** control statement
.control
run
plot v(qp1)
.endc
.end
```

And to simulate this netlist, type `ngspice ctat_voltage_gen_mul_bjt.sp`. Now we the see the corresponding waveform after the simulation

![ctat_mulbjts_c2_Picture1](https://user-images.githubusercontent.com/89923461/156957442-4513d2b6-7e74-4b59-978c-11a94573785d.png)




















