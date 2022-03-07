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

 In this simulation, we are going to see calculate voltage across 8 BJTs which are parllely connectd. The spice netlist which we are going to simulate is, 

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

### CTAT Voltage generation with different current source values

In this simulation, the main objective is to check the CTAT voltage dependency on current. The spice netlist which we are going to simulate is, 

```
**** ctat voltage generation circuit with variable current *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"


.global vdd gnd
.temp 27

*** bjt definition
xqp1	gnd	gnd	qp1	gnd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1

*** supply current
vsup	vdd	gnd	dc	2
isup	vdd	qp1	dc 	10u
.dc	temp	-40	125	5	isup	1.25u	10u	1.25u

*** control statement
.control
run
plot v(qp1)
.endc
.end
```

And to simulate this netlist, type `ngspice ctat_voltage_gen_var_current.sp`. Now we the see the corresponding waveform after the simulation

![var_current_Picture1](https://user-images.githubusercontent.com/89923461/156958493-2b52ad30-1bff-4ee1-9da7-fc03c2fbf6cc.png)


## PTAT Simulation

### PTAT Voltage generation with ideal current source

In this simulation, the main objective is to find the voltage difference between the two terminals of the resistance. The spice netlist which we are going to simulate is, 

```
**** ptat voltage generation circuit *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

	
*** bjt definition
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8

*** supply voltage and current
isup	vdd	ra1	dc	10u
vsup	vdd	gnd	dc 	2
.dc	temp	-40	125	1

*** control statement
.control
run
plot v(ra1) v(qp2) v(ra1)-v(qp2)
.endc
.end
```

And to simulate this netlist, type `ngspice ptat_voltage_gen_ideal_current_source.sp`. Now we the see the corresponding waveform after the simulation

![ideal_Picture1](https://user-images.githubusercontent.com/89923461/156958914-3ed7a807-423e-4382-b8ed-bcb367e7a428.png)

### PTAT Voltage generation with VCVS

In this simulation, the main objective is to check the amplified PTAT voltage using one VCVS. The spice netlist which we are going to simulate is, 

```
**** ptat voltage generation circuit *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** vcvs definition
e1	net2	gnd	ra1	qp1	gain=1000
xmp1    q1	net2   	vdd	vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp2    q2    	net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
	
*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8

*** voltage sources for current measurement
vid1	q1	qp1	dc	0
vid2	q2	ra1	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
.dc	temp	-40	125	5

*** control statement
.control
run
plot v(qp1) v(ra1) v(qp2) v(net2)
plot vid1#branch vid2#branch
.endc
.end
```

And to simulate this netlist, type `ngspice ptat_voltage_gen.sp`. Now we the see the corresponding waveform after the simulation

![vcvs_Picture1](https://user-images.githubusercontent.com/89923461/156959196-4a174ca4-3b37-4cc2-ad22-576e2e049245.png)














