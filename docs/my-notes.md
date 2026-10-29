# My notes

## Prerequisite Knowledge

- Voltage: Electrical pressure (height)
- Current: How much flows past a point at each second (people going down)
- Power: Total energy (people x height)

<br>

- Wire: Needs to be wide (sliding causes friction/heat)
- Breaker: Stops current (Attendant who shuts slide if too many people pile)
- Load: How much power device is drawing from supply

Mains power: Electricity supplied to buildings from public grid

1. Source => flow out through live wire
2. Through device
3. Neutral wire => source

- Phase: Live wire (with AC voltage wave), compared to neutral wire

Alternating Current: Keeps reversing, changing voltage
Direct Current: Flows steadily in one direction, constant voltage

## More Prerequisite Knowledge

### POWER

Real vs Apparent Power:

- Volt-amps: Everything flowing through wire
- Watts: Actual power used
- Power factor:

RMS Voltage & Current:

1. Plain average = 0
2. RMS voltage: steady (DC) voltage that would heat something just as much
   1. Voltage swings 50Hz between -325 to +325V
   2. 230V in Aus

PSU efficiency vs load:

- Power supply unit has fixed running costs: Control chips, fan, internal switching
- Most power supplies: 40-60% load

### HARMONICS

Harmonics and THD:

1. Ideal case: Smooth wave, matches voltage
2. Basic supplies: Short/sharp spikes, near top of each voltage wave
   1. Spike wave: 50Hz plus multiples
      1. Harmonics: Multiples (add heat with no use / pile in neutral wire)
      2. Total harmonic distortion: How big they are, compared to clean wave

Active power factor correction:

1. PFC circuit (at input): Forces current to follow smooth voltage wave instead of spiking
   1. PF: near 0.99 , harmonics are low
   2. Low load => input filter/control circuits draw current => PF drops

### WIRES/BREAKERS

Single vs 3 Phase

1. Single phase: One live wire, 230V, neutral
2. Three phase: 3 live wires, 230V, neutral (staggered waves, third of cycle behind)

- 400V: between two live wires

Breaker derating and phase balance

- Derating: Breaker has small metal inside => heats up as current flows
  - Continuous load: 80% of rating
- Phase balance: Spread load evenly across L1/2/3
  - If not: L1's breaker trips even though feed has plenty of spare capacity AND extra current through neutral wire

2N and dual-corded racks:

- Each server has 2 power suppliers => 2 feeds
- If feed A dies, feed B must be able to carry whole rack (current)

Power becomes heat => need cooling

## Observations around PSUS

- PF rises with load. At 10% load it ranges from 0.72 to 0.95 across the three PSUs; at 50% load it is 0.96 to 0.99.
- Current THD falls with load overall: 17.17% to 6.63% for the 2000 W unit, and 16.26% to 3.16% for the 750 W unit. The 1300 W unit stays near 9–10% up to 50% load before falling to 5.72%.
- YSEC2000AM at 20% load: current calculated from input power and PF (430 W ÷ (230 V × 0.95) = 1.97 A) is about 2.5% above the reported Irms (1.92 A). Data kept as published.
- Efficiency peaks at 50% load for all three PSUs (94.04% to 94.61%) and is lowest at 10% load (85.71% to 89.26%). The 1300 W unit is the least efficient at 10% load, even though its PF there (0.93) is better than the 750 W unit's (0.72).
