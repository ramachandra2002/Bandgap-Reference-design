# Bandgap Reference design using Sky130 technology node

This repository contains the analysis and design of Bandgap IP Design using Sky130 technology node

![bgr_Banner](https://user-images.githubusercontent.com/89923461/156914849-f38073fb-23c6-4563-affe-b5a564a2c4f6.png)

# Day 1: Bandgap Design Theory

# Introduction to BGR

The Bandgap Reference is a circuit which generates stable voltage which is independent of factors like temperature, supply voltage. A battery cannot be used because it's voltage will drop after a time period. And to avoid noisy voltages and additional components we use bandgap reference which can be integrated in bulk-CMOS, bi-CMOS without external components.

![intro_photo_2022-03-07_23-49-43](https://user-images.githubusercontent.com/89923461/157094522-39bc528f-0e84-426b-b6b6-5379db61fe93.jpg)

## BGR Principle

The principle of BGR circuits is to sum a voltage with negative temprature coefficient with another one exhibiting opposite temperature dependancies. In general, semiconductor diodes act as CTAT, or complement to absolute temperature, which implies that as temperature rises, the voltage across the diode decreases. As a result, we must devise a PTAT circuit capable of cancelling out the CTAT nature, i.e. with an increase in temperature. The voltage across that device will rise, allowing us to get a constant voltage reference with regard to temperature.

![priciple_photo_2022-03-07_23-50-13](https://user-images.githubusercontent.com/89923461/157094527-2b451eb1-42e8-4d22-a5db-71433d778e58.jpg)


##  CTAT Voltage generation circuit

CTAT behaviour is common in semiconductor diodes. When a constant current flows through a forward biassed diode, we can see that the voltage across the diode decreases as the temperature rises. In general, the slope of the VTemp is determined to be -2mV/deg C. The CTAT Voltage generating circuit is made up of a BJT linked as a diode, which demonstrates the CTAT nature described above.

![ctat_photo_2022-03-07_23-50-23](https://user-images.githubusercontent.com/89923461/157094550-8dfe7fd6-318e-4bd5-ba92-9a06ff348c39.jpg)


## PTAT Voltage generation circuit

Diode current equation has two parts, i.e. Vt (Thermal Voltage) which is directly proportional to the temp and  Is (Reverse saturation current) which is directly proportional to the temp.

So, in order to create a PTAT Voltage generating circuit, we must first figure out how to extract Vt from Is. The circuit should consist of N BJTs linked with a series resistance to isolate Vt from Is.

![ptat_photo_2022-03-07_23-50-31](https://user-images.githubusercontent.com/89923461/157094533-6bdd8898-25a3-4733-a2e7-74a02d1947a3.jpg)


## Self-Biased Current Mirror Circuit

The self-biased current mirror is a sort of current mirror that does not need to be biased externally. Without any external current source reference, this current mirror biases itself to the desired current value.

![current mirror_photo_2022-03-07_23-50-49](https://user-images.githubusercontent.com/89923461/157094511-dcb99205-2797-4119-a348-b78b4965c731.jpg)


## Reference Branch Circuit

The final reference voltage is obtained by adding the CTAT and PTAT voltages in the reference circuit branch. In the reference branch, we use a mirror transitor and a BJT as a diode. The reference branch receives the same amount of current as the current mirror branches due to the mirror transistor. We now have PTAT voltage and PTAT current coming from the PTAT circuit branch. In the reference branch, the same PTAT current is flowing. However, the PTAT voltage slope is substantially smaller than the CTAT voltage slope. We must raise the resistance in order to enhance the voltage slope. We now have our constant reference voltage, which is the consequence of CTAT Voltage + PTAT Voltage across the high resistance.

![ref_photo_2022-03-07_23-51-02](https://user-images.githubusercontent.com/89923461/157094539-62e66bfe-4cfd-4736-a230-98a74aa5c666.jpg)


## Start-up circuit

To transfer the self biassed current mirror from the degenerative bias point, the start-up circuit is necessary (zero current). When the current in the current mirror branches is zero, the start-up circuit forcibly flows a small amount of current through the self-biased current mirror. Because the current mirror is self-biased, this small current causes a disturbance, and the current mirror auto-biased to the desired current value.

![startup_photo_2022-03-07_23-51-10](https://user-images.githubusercontent.com/89923461/157094544-25e935bd-89d9-4257-99e6-4c3a9ffb24a5.jpg)

## Complete BGR Circuit

By connecting the subcircuits we get the total circuit,

![final_photo_2022-03-07_23-51-19](https://user-images.githubusercontent.com/89923461/157094517-1a3863e3-417a-4947-ac27-33e27bbd2782.jpg)


# Tools Used

- Ngspice: *for simulation of spice files*
- Magic: *for creating layouts for circuits*
- Netgen: *tool for comparing netlists*

# Specification

- Supply voltage = 1.8V
- Temperature: -40 to 125 Deg Cent.
- Power Consumption < 60uW
- Off current < 2uA
- Start-up time < 2us
- Tempco. Of Vref < 50 ppm

# Day 2:  BGR Labs and post-layout simulations

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

## Resistance tempco.

The voltage across the resistor also increases with increase in the temperature, which makes the resistor similar to PTAT. The main objective of this simulation is to check the tempco. of resistor using ideal current source of 10uA. The spice netlist which we are going to simulate is,

```
**** ctat voltage generation circuit *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** resistor definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** supply current
vsup	vdd	gnd	dc	2
vid     qp2     gnd     dc      0
isup	gnd	ra1	dc 	10u
.dc	temp	-40	125	1	

*** control statement
.control
run
plot v(ra1) 
plot v(ra1)/vid#branch
.endc
.end
```

And to simulate this netlist, type `ngspice res_tempco.sp`. Now we the see the corresponding waveform after the simulation

![tempco_Picture1](https://user-images.githubusercontent.com/89923461/156959904-48844b04-1c0f-4d76-a489-56f48ceb22b2.png)

To find the PTAT voltages across the resistance for different current values from the following curve we have to simulate the netlist `res_tempco_var_current.sp` 

```
**** RES tempco circuit *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** resistor definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** supply current
vsup	vdd	gnd	dc	2
vid     qp2     gnd     dc      0
isup	gnd	ra1	dc 	10u
.dc	temp	-40	125	1	isup	1.25u	10u	1.25u

*** control statement
.control
run
plot v(ra1) 
plot v(ra1)/vid#branch
.endc
.end
```

![var_c_Picture1](https://user-images.githubusercontent.com/89923461/156960129-ead047f8-bcae-45d2-a98c-eb02d78acfc2.png)

## BGR using Ideal Opamp

To simulate the BGR behaviour using one VCVS as an ideal OpAmp, we have to simulate the netlist `bgr_using_ideal_opamp.sp`

```
**** bgr using ideal opamp (vcvs) *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** vcvs definition
e1 net2 gnd ra1 qp1 gain=1000


xmp1    qp1	net2   	vdd	vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp2    ra1 	net2   	vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp3    ref	net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
	
*** bjt definition
xqp1	gnd	gnd	qp1	gnd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     gnd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     gnd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8

xrb1    ref     nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41	l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41 	l=7.8
xrb17   nb16    nb17    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41   	l=7.8
xrb18   nb17    nb18    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41   	l=7.8
xrb19   nb18    nb19    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41   	l=7.8
xrb20   nb19    nb20    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	 l=7.8
xrb21   nb20    nb21    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41   	l=7.8
xrb22   nb21    nb22    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	 l=7.8
xrb23   nb22    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8
xrb24   nb22    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  	l=7.8

*** voltage source for current measurement

*** supply voltage
vsup    vdd     gnd     dc      2
*.dc    vsup    0       3.3     0.3.3

.dc	temp    -40     125     5

*vsup    vdd     gnd     pulse   0       2       10n     1u      1u      1m      100u
*.tran   5n      10u

.control
RUN
plot v(vdd) v(qp1) v(ra1) v(qp2) v(ref) v(qp3)
plot v(ref)


.endc
.end
```

![Picture1](https://user-images.githubusercontent.com/89923461/156960669-00cb01cd-65a2-44d4-808f-1386e693f46f.png)

## BGR with SBCM

Let's replace the ideal Op-Amp with self-biased current mirror which is our proposed design and observe. The expected output is of that which in case of ideal OpAmp based BGR. Now lets check Behaviour of the circuit in TT corner using the spice netlist `bgr_lvt_rpolyh_3p40.sp`

```
**** bandgap reference circuit using self-biase current mirror *****

.lib "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/ramachandra14519/Desktop/ramachandra_bgr/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** circuit definition ***

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
*.dc	vsup	0	3.3	0.3.3
.dc	temp	-40	125	5

*vsup	vdd	gnd	pulse	0	2	10n	1u	1u	1m	100u
*.tran	5n	10u

.control
run

plot v(vref)

.endc
.end

```

![tt_Picture2](https://user-images.githubusercontent.com/89923461/156961059-36383dfa-47ed-4ef6-8dcb-38927b7b12ce.png)

# Layout Design

The Layout Design for the sub circuits are done magic tool. The invoke the magic tool for new design, the command to type is,

```
magic -T <technology_file_name.tech> <layout.mag>
```

## Basic Cell Design

### Design of NFET

In our design we have used two different size nfets:

1. W=5 L=1 

![nfet1_Picture2](https://user-images.githubusercontent.com/89923461/156961553-041fee16-d4dd-4b7d-8018-801ace5de4a2.png)

2. W=1 L=7

![nfet2_Picture2](https://user-images.githubusercontent.com/89923461/156961619-b0bc2cac-c08d-4aef-a901-b8f8734dbbd4.png)

### Design of PFET

Size: W=5 L=2

![pfet_Picture2](https://user-images.githubusercontent.com/89923461/156961713-34fc5181-d7ba-4684-990b-1756a8480897.png)


### Design of Resistor

poly resistors of W=1.41 and L=7.8

![res_Picture2](https://user-images.githubusercontent.com/89923461/156961907-24840eab-f708-4ba2-996e-61b105209874.png)

### Design of PNP (BJT)

![pnp_Picture2](https://user-images.githubusercontent.com/89923461/156961995-9fe94d0c-11ee-4427-9345-4dc4c17ef683.png)

## Block Design

### Design of NFETs

![nfets_Picture1](https://user-images.githubusercontent.com/89923461/157080936-d66fd772-55f9-4e9f-96f2-561c002f56da.png)

### Design of PFETs

![pfets_Picture1](https://user-images.githubusercontent.com/89923461/157081067-cd6dfc95-de84-4783-aaf7-5b42a92254a8.png)

### Design of RESBANK

![Design of RESBANK_Picture1](https://user-images.githubusercontent.com/89923461/157081217-9adc391f-db23-4935-b2ff-e37c8b12f23c.png)

###  Design of PNP10

![pnp10](https://user-images.githubusercontent.com/89923461/157081557-67e61669-39ef-4b45-9829-d9be99565a3b.png)

### Design of STARTERNFET

![Design of STARTERNFET_Picture1](https://user-images.githubusercontent.com/89923461/157095213-d108d246-88d4-4f2d-a52a-b53d384a842d.png)


## Top level design

![top](https://user-images.githubusercontent.com/89923461/157081567-9739c861-bc3b-4087-b237-2634187fa2cc.png)


# Author

Ramachandra T, BE in Electronics and Communication Engineering at Madras Institute of Technology, Anna University

# Acknowledgement

1. Kunal Ghosh, VSD Corp
2. Santunu Sarangi, Silicon Institute of Technology (SIT), Bhubaneswar






















