# EPQ Revision Notes

# Electrical Power Quality — Midsem Revision

Use the recall sheets first, then cover the answers and attempt the questions. Study **P0 completely → reliability numerical → remaining P1 → P2 applications → P3 if time remains**. Topic numbers refer to [the midsem syllabus](../Syllabus/syllabus.md). Content and established conventions follow [the detailed notes](notes.md); question emphasis follows the [topic-frequency analysis](../PYQ_Analysis/topic_frequency.md), [question mapping](../PYQ_Analysis/question_mapping.md) and [gap analysis](../PYQ_Analysis/gaps.md).

## 1. Magnitude–duration master table — P0, Topics 4–11

$$
V_{\mathrm{pu}}=\frac{V_{\mathrm{RMS}}}{V_{\mathrm{nominal}}}.
$$

Both voltages use the same units. **0.7 pu means 70% voltage remaining**, or a 30% reduction.

| Disturbance | Magnitude / shape | Duration | Main memory anchor |
| --- | --- | --- | --- |
| Transient | Sharp impulse or decaying ringing. | Before 0.5 cycle on the class chart. | A brief departure superimposed on the waveform. |
| Voltage sag | **0.1–0.9 pu** RMS. | **0.5 cycle–1 min**. | Short low voltage; faults or motor starting. |
| Voltage swell | **1.1–1.8 pu** overall; see subclasses below. | **0.5 cycle–1 min**. | Short high voltage; load disconnection or capacitor switching. |
| Under-voltage | **Below 0.9 pu**, outside the interruption region. | **More than 1 min**. | Sustained low voltage; loading or regulation problem. |
| Over-voltage | **Above 1.1 pu**. | **More than 1 min**. | Sustained high voltage; taps or light loading. |
| Very brief interruption | **Below 0.1 pu**, nearly zero. | Before 0.5 cycle on the class chart. | Leftmost near-zero region, labelled “Interruption.” |
| Momentary interruption | **Below 0.1 pu**. | **0.5 cycle–3 s**. | Brief loss of continuity. |
| Temporary interruption | **Below 0.1 pu**. | **Above 3 s–1 min**. | Longer short interruption. |
| Sustained interruption | **Below 0.1 pu**, nearly zero. | **More than 1 min**. | Long loss of continuity. |

**Swell subclasses:** instantaneous = 1.1–1.8 pu for 0.5–30 cycles; momentary = 1.1–1.4 pu for 30 cycles–3 s; temporary = 1.1–1.2 pu for 3 s–1 min.

**Chart to draw:** Magnitude vertically and duration horizontally. Mark **0.1, 0.9 and 1.1 pu**, then **0.5 cycle, 3 s and 1 min**. The approximate normal band is 0.9–1.1 pu. Use the established class-chart convention above for interruptions; transient shape still matters when identifying very brief events.

## 2. Definitions and disturbance recognition — P0, Topics 1–12

**Power quality** describes whether electrical power is suitable for the equipment using it. **Voltage quality** concerns the supplied voltage waveform and operating conditions; **current quality** concerns the current waveform drawn by the load. **Continuity** concerns keeping service available. The **PCC, Point of Common Coupling**, is the shared network connection through which one load’s disturbances can affect others.

**Why PQ matters:** Power-electronic loads can both generate distortion and respond poorly to disturbances. Faults and switching affect transmission and distribution supplies; nonlinear consumer loads affect the common network. A brief controller reset can cause lengthy production loss even after the supply recovers.

| What the waveform shows | Recognition and mechanism |
| --- | --- |
| One sharp positive or negative spike. | **Impulsive transient:** lightning is the anchor cause; the peak can damage electronics. Recall surge protection. |
| Rapid ringing that decays around the normal waveform. | **Oscillatory transient:** switching excites energy exchange between inductance and capacitance; equipment stress follows. Recall switching-transient suppression. |
| Several smaller sine-wave cycles, then recovery. | **Sag:** fault or starting current increases the drop across upstream impedance. Drives may trip; DVR/UPS can protect the load. |
| Several larger sine-wave cycles, then recovery. | **Swell:** load disconnection or capacitor switching raises voltage. Insulation is stressed; suitable series compensation can oppose the rise. |
| A nearly flat zero-voltage interval. | **Interruption:** protective opening or supply failure removes power. Equipment stops; stored energy or a healthy alternative supply is needed. |
| An envelope that repeatedly grows and shrinks. | **Voltage fluctuation:** arc-furnace demand changes network voltage drop. Lamp flicker results; strengthen the supply or compensate varying reactive demand. |
| Narrow inward cuts repeated within successive cycles. | **Notching:** converter commutation creates voltage cuts. Data errors or capacitive-component stress may result; use an input line reactor. |
| Small irregular disturbances superimposed on the wave. | **Noise:** switching electronics, arcing or radio interference disrupt signals. Data/communication errors result; use filtering, isolation and proper grounding. |

**Fluctuation versus flicker:** Fluctuation is electrical amplitude variation; flicker is the resulting visible brightness variation. Typical taught values are **0.95–1.05 pu**, with envelope frequency **below 25 Hz**. The envelope frequency is different from the supply’s fundamental frequency.

**Noise:** Unwanted broadband voltage/current content below **200 kHz** in the taught definition, excluding components better classified as harmonics or transients.

**Sketch rule:** Label voltage and time axes, remaining RMS magnitude and duration. Extend a low/high-voltage region beyond one minute for under-/over-voltage. A notch is a cut within a cycle; a sag lowers RMS voltage across a longer interval.

## 3. Harmonics and THD — P0, Topics 13–15; P2 source recall, Topic 32

A **harmonic** is a sinusoidal component at an integer multiple of the fundamental frequency:

$$
f_h=h f_1,\qquad
\mathrm{THD}(\%)=
\frac{\sqrt{\displaystyle\sum_{h=2}^{h_{\max}}M_h^2}}{M_1}\times100.
$$

Here, (f_1) is fundamental frequency in hertz, (h) is harmonic order, (M_1) is the fundamental RMS magnitude, and (M_h) is a higher-harmonic RMS magnitude. Use all voltage values or all current values in consistent units. THD measures combined higher-harmonic RMS content relative to the fundamental.

**Calculation:** Square higher harmonics → add → square root → divide by fundamental → multiply by 100. **The fundamental is excluded from the numerator; the square root is essential.** Zero THD means no higher harmonics. For unchanged harmonic amperes, a smaller fundamental current gives higher THD.

| Recall point | Exam content |
| --- | --- |
| Generation mechanism | Nonlinear loads draw nonsinusoidal current even from sinusoidal voltage. Adding harmonic components to the fundamental produces a distorted resultant. |
| Harmonic versus interharmonic | Harmonics have integer order; interharmonics have non-integer order. At 50 Hz, 150 Hz is the third harmonic; 175 Hz is an interharmonic. |
| Four industrial sources | **Transformers, arc furnaces, power converters and ASDs.** Also recall SMPS, rotating machines, cycloconverters and AC voltage controllers. |
| Source mechanisms | Transformer saturation gives nonlinear magnetizing current; arcs have nonlinear behaviour; converters conduct/switch over selected portions of a cycle. |
| Main effects | Additional losses and heating, capacitor overloading, vibration, communication interference and protection/metering errors. |
| Main remedy | Suitable passive, active or hybrid harmonic filtering at operating-principle level. |
| IEEE 519 class point | Its purpose is to limit harmonic distortion. **Sir’s simplified exam criterion: current THD should be less than 5%.** This is the taught point, not a universal statement of all IEEE 519 limits. |

**Diagrams to draw:** One fundamental cycle alongside three smaller third-harmonic cycles on the same time scale, then a distorted sum. For a typical SMPS input, draw repeated current pulses near voltage peaks; converter waveforms vary with the input arrangement.

## 4. Voltage unbalance and sequence components — P0 Topic 16; P1 Topics 21–22

**Balanced supply:** Equal phase-voltage magnitudes **and 120° separation**. **Unbalance:** Unequal magnitudes and/or non-120° separation. Equal magnitudes alone are insufficient.

**Mechanism:** Unequal single-phase loads → unequal phase currents → unequal drops across real network impedance → unequal load voltages. Other causes are unequal impedances, incomplete line transposition, faults, open phases and blown capacitor fuses. Incomplete transposition leaves the conductors in unequal electrical conditions along the line.

**Effects and remedy:** Negative-sequence components are particularly important for motor heating; unbalance increases losses, degrades performance and shortens life. Redistribute single-phase loads and correct faulty connections or open phases.

**An unbalanced three-phase set can be represented as the sum of positive-, negative- and zero-sequence components.**

| Sequence | Magnitudes | Phase relationship / order |
| --- | --- | --- |
| Positive | Equal. | 120° apart, normal **A–B–C** order. |
| Negative | Equal. | 120° apart, reverse **A–C–B** order. |
| Zero | Equal. | **All in phase**. |

For basic relations, (a=1^), (a2=1), (a^3=1), and (1+a+a^2=0). Multiplication by (a) rotates a phasor through 120°. Let (V_0,V_1,V_2) be the phase-A zero-, positive- and negative-sequence voltage phasors:

$$
\begin{aligned}
V_A&=V_0+V_1+V_2,& V_0&=\frac{V_A+V_B+V_C}{3},\\
V_B&=V_0+a^2V_1+aV_2,& V_1&=\frac{V_A+aV_B+a^2V_C}{3},\\
V_C&=V_0+aV_1+a^2V_2,& V_2&=\frac{V_A+a^2V_B+aV_C}{3}.
\end{aligned}
$$

All voltages are phasors in consistent units; both magnitude and angle matter. Normal balanced supply contains only positive sequence. Check (V_A+V_B+V_C=3V_0). Prioritize definitions and phasor diagrams over full calculations.

## 5. Ferroresonance — P0, Topic 17

**Trigger:** System capacitance interacts with a transformer’s nonlinear iron-core magnetizing inductance. Core saturation makes the inductance dependent on operating condition. Switching can establish or excite a path with magnetizing impedance effectively in series with capacitance.

The taught series-RLC interpretation is:

$$
Z=R+j(X_L-|X_C|),\qquad I=\frac{E}{Z},\qquad
X_L=|X_C|\Rightarrow Z\approx R.
$$

Here, (R) is resistance, (X_L) is inductive reactance, (|X_C|) is capacitive-reactance magnitude, and (Z) is impedance, all in ohms. (E) is applied voltage, (I) is current, and (j) denotes the imaginary operator. When the reactive terms cancel, small resistance permits large current and large voltages across the reactive elements.

**Recall chain:** Nonlinear inductance + capacitance → resonance interaction → high voltage/current → harmonic magnification and irregular waveforms. The RLC model explains cancellation; actual ferroresonance involves nonlinear inductance.

**Distinguish:** Unwanted network ferroresonance stresses equipment. A **ferroresonant/constant-voltage transformer (CVT)** deliberately uses a saturated core and resonant capacitor to maintain approximately constant output within its operating range.

**Diagram to draw:** AC source, resistance, iron-core inductance and capacitor in one series loop. Label current and the voltage across the inductance.

## 6. Reliability formula sheet — P1, Topics 18–20

**Reliability** concerns continuity of utility service, particularly sustained interruptions. Count affected customer service, not merely outage events.

Let (N_T) be **all customers served**, (N_i) the customers interrupted in outage (i), (U_i) its duration, and (N_A) the **distinct customers affected at least once**. Define:

$$
C=\sum_i N_i,\qquad D=\sum_i N_iU_i.
$$

(C) counts customer interruptions, including repeats. (D) is customer-interruption duration: customer-minutes if (U_i) is in minutes. For ASAI below, express (D) in customer-hours and let (H) be the study period in hours.

| Index and full name | Formula | Meaning / units for the study period |
| --- | --- | --- |
| **SAIFI:** System Average Interruption Frequency Index | (C/N_T) | Interruptions per **served** customer. |
| **SAIDI:** System Average Interruption Duration Index | (D/N_T) | Outage minutes or hours per **served** customer. |
| **CAIFI:** Customer Average Interruption Frequency Index | (C/N_A) | Interruptions per **distinct affected** customer. |
| **CAIDI:** Customer Average Interruption Duration Index | (D/C) | Minutes or hours per **customer interruption**. |
| **ASAI:** Average System Availability Index | (1-D/(N_TH)) | Available fraction of demanded customer service; dimensionless. |

**Numerical procedure:**

1. Identify (N_T) and the study period; calculate **(C=N_i)**.
2. Calculate every (N_iU_i), then **(D=N_iU_i)**.
3. Find SAIFI, SAIDI and CAIDI with the correct denominators.
4. For ASAI, convert customer-minutes to customer-hours and use (H=24) for a day or (H=8760) for a non-leap year, as specified.
5. Find CAIFI **only when distinct affected customers are known**, or state a justified assumption explicitly.
6. Check:

$$
\mathrm{SAIDI}=\mathrm{SAIFI}\times\mathrm{CAIDI},\qquad 0\leq\mathrm{ASAI}\leq1.
$$

**Denominator trap:** The number of table rows is not (C); repeated customers count again in (C), but only once in (N_A). SAIFI is an average interruption count, not an outage probability.

**Established correction from the detailed notes:** For the class example, SAIDI is **0.42816 min/customer**, not the printed 0.0428. The printed CAIFI calculation (5/1014) is unsuitable; (N_A) is required. The worked answer is in Question 14.

## 7. Short-theory revision block — P1, Topics 23–28

| Topic | Structure to reconstruct an answer |
| --- | --- |
| **23. Classification** | **Time:** brief events versus continuing/steady-state conditions. **Quantity:** voltage, current or frequency. **Origin:** supply side or load side. A sag is a short RMS variation; an impulse is a transient. |
| **24. Causes** | **Natural/network:** lightning, weather, faults, equipment failure. **Operational:** transformer, capacitor, feeder and heavy-load switching. **Load-related:** nonlinear converters/controllers and rapidly varying arc loads. |
| **25. Effects** | **Operational:** malfunction, control reset, data and production loss. **Electrical:** heating, losses, insulation stress, capacitor overloading and reduced life. **Protection/metering/communication:** false operation, measurement errors and interference. |
| **26. Short versus long duration** | **Up to 1 minute:** sag, swell and short interruption. **More than 1 minute:** under-voltage, over-voltage and sustained interruption. Combine duration with remaining RMS voltage. |
| **27. Waveform distortion** | Five types: **DC offset, harmonics, interharmonics, notching and noise**. DC offset shifts the waveform’s average away from zero; half-wave rectification or converter asymmetry can cause it. Core saturation and heating can follow. Interharmonic sources include cycloconverters, induction furnaces and arcing devices. |
| **28. Power-frequency variation** | The fundamental frequency deviates from nominal. Generation/load imbalance, faults in isolated systems, loss of synchronism or islanding can cause it. Loads/transformers are affected, and generator/turbine shafts may experience mechanical stress. |

**Frequency mechanism:**

$$
\text{Generation–load imbalance}
\rightarrow\text{torque imbalance}
\rightarrow\text{generator speed change}
\rightarrow\text{frequency change}.
$$

Islanding separates part of the network into an isolated supply system. In a frequency-variation sketch, change cycle spacing: closer cycles mean higher frequency. Harmonics add frequency components; power-frequency variation changes the fundamental itself.

## 8. Sag mitigation comparison — P2, Topic 29

**Ride-through** means keeping equipment operating through a disturbance without a trip. Choose by sag magnitude/duration, load tolerance, available stored energy and availability of a healthy second source.

| Method | Main idea and operation | When useful / main condition |
| --- | --- | --- |
| **Active series compensation / DVR** | Detect the event and inject the missing voltage **in series**. Series compensation can also oppose excess voltage. | A sensitive load needs near-normal voltage; compensation and energy limits must suit the event. |
| **Static transfer switch** | Solid-state switches transfer the load to an independent healthy source, **break before make**. | A redundant healthy AC supply is available near the critical load. |
| **Fast transfer scheme** | Detect deterioration and rapidly transfer to a standby feeder, possibly using coordinated high-speed breakers. | A suitable standby feeder and fast switching arrangement are available. |
| **UPS / stored energy** | A battery/DC source supports an inverter when input power is inadequate. | Critical electronics need continuity through sags or interruptions; stored energy limits duration. |
| **CVT / ferroresonant transformer** | A **saturated core + resonant capacitor** maintains approximately constant output. | Voltage conditioning is needed within the device’s operating range. |

For a DVR:

$$
\mathbf V_{\mathrm{load}}=\mathbf V_{\mathrm{supply}}+\mathbf V_{\mathrm{injected}}.
$$

These are voltage phasors in volts or a common pu base. Correct injection magnitude and phase restore the required load voltage.

**Online UPS:** AC → rectifier/DC stage → inverter → load; the inverter continuously supplies the load, and the battery supports its DC stage during failure. **Offline UPS:** Utility normally supplies the load; after disturbance detection, the load transfers to the battery-fed inverter.

**Diagram cues:** DVR: supply → series injection → load, with sensing/controller/power circuit. Transfer switch: two sources → separate switch paths → one load. CVT: transformer with capacitor at its regulating winding. Mechanical tap regulation suits sustained voltage problems; it is too slow for a sag lasting only a few cycles.

## 9. Cause–effect–remedy matrix — P2 Topic 30, using P0 definitions

| Disturbance | Anchor cause | Anchor effect | Anchor remedy |
| --- | --- | --- | --- |
| Impulsive transient | Lightning. | Electronic component damage. | Surge arrester/suppressor. |
| Oscillatory transient | Capacitor or line switching. | Equipment stress and data errors. | Switching-transient suppression; pre-insertion resistance/inductance. |
| Sag | Fault or motor starting. | Drive trip or contactor dropout. | DVR; UPS or healthy-source transfer. |
| Swell | Large-load disconnection or capacitor switching. | Insulation/power-supply stress. | Series compensation opposing excess voltage. |
| Interruption | Fault-related protective opening. | Equipment and production stoppage. | UPS or independent healthy-source transfer. |
| Under-voltage | Heavy loading and excessive feeder drop. | Motor heating or electronic-supply failure. | Suitable voltage regulation/tap adjustment; reduce excessive drop. |
| Over-voltage | Incorrect taps or light loading. | Insulation stress and appliance damage. | Correct regulator/tap settings; coordinate capacitors with load. |
| Voltage fluctuation | Arc-furnace demand variation. | Lamp flicker. | Strengthen supply or compensate varying reactive demand. |
| Voltage unbalance | Uneven single-phase loading. | Motor heating and increased losses. | Redistribute loads; correct faulty phase connections. |
| Harmonics | Nonlinear converters/SMPS/ASDs. | Heating and capacitor overloading. | Suitable harmonic filtering. |
| Notching | Converter commutation. | Data errors or capacitive-component stress. | Converter input line reactor. |
| Noise | Switching electronics or arcing. | Communication/data errors. | Filters, isolation and proper grounding. |

For a broad answer, expand each row into **definition → cause/mechanism → effect → remedy**. Match remedy response time to event duration.

## 10. Equipment and industrial applications — P2, Topics 31–33

| Sag-sensitivity category | Examples / explanation |
| --- | --- |
| **Mainly magnitude** | Undervoltage relays, process controls, motor-drive controls and automated/semiconductor-manufacturing machines. A voltage threshold is central. |
| **Magnitude and duration** | Electronic power supplies. They may tolerate a brief dip but fail if the low input persists. |
| **Other characteristics** | Phase shift, three-phase unbalance, point-on-wave at initiation/recovery and superimposed oscillations can change the response. |

These are the taught broad categories; individual equipment settings matter. **Point-on-wave** means the position in the AC cycle when the disturbance begins or ends.

**Industrial harmonic-source recall:** Transformer saturation; arc-furnace nonlinearity; converter switching/rectification; ASD rectifier input. The consolidated source/effect/remedy table is in Section 3.

**Typical ASD:** AC supply → rectifier → **DC-link capacitor** → inverter → motor. Diodes conduct when the available rectified supply voltage exceeds capacitor voltage. Between charging intervals, the capacitor supplies the load.

$$
\text{AC sag}
\rightarrow\text{reduced DC-link capacitor charging}
\rightarrow V_{\mathrm{DC}}\downarrow
\rightarrow\text{undervoltage trip}
\rightarrow\text{process stoppage}.
$$

(V_{}) is DC-link voltage. A short/shallow sag may be tolerated; deeper or longer sags can cross the trip threshold. Rectifier-fed electronic supplies have the same basic loss-of-DC-support mechanism.

**Two directions:** Motor starting current can **cause** a network sag; an incoming sag can **trip** an ASD. For the drive-response diagram, label the four drive blocks, capacitor and DC undervoltage threshold.

## 11. Lower-priority / one-off topics — P3, Topics 34–38

Do this quick pass after P0–P2. It follows the explanations already established in the detailed notes.

| Topic | Minimum recall |
| --- | --- |
| **34. IEC classification** | **Conducted low frequency:** harmonics, voltage/frequency changes, DC components. **Radiated low frequency:** electric/magnetic fields. **Conducted high frequency:** continuous signals and transients on conductors. **Radiated high frequency:** continuous, modulated or pulsed fields. **ESD:** discharge of accumulated charge. |
| **35. Sag with harmonics** | Lower RMS voltage **and** distorted cycles. Sketch normal voltage → smaller distorted cycles → recovery; label magnitude, duration and distortion. |
| **36. Lamps** | Incandescent: approximately resistive, nearly sinusoidal current under sinusoidal voltage, little harmonic generation; voltage fluctuation changes brightness. Fluorescent: ballast-dependent nonlinear current and harmonics; high-frequency electronic ballasts reduce low-frequency light variation. |
| **37. Sensitive process/machine** | One controller/drive upset can stop the process and require a long restart. Draw supply → sensitive controller/drive → machine/process. The older question’s wording is unclear; this is a conceptual diagram, not an identified standard machine. |
| **38. Further ASD response** | Heavier loading uses stored DC energy faster; lighter loading can improve ride-through. Some drives permit reduced- or zero-torque operation temporarily. Recall the basic DC-link mechanism; detailed drive design is unnecessary. |

## 12. Exam-style active recall with model answers

These are adapted or newly composed practice prompts, not a claim to reproduce every PYQ verbatim. Marks indicate intended answer depth. Try the question before reading its answer; for longer answers, practise adding the indicated diagram. Distribution: **12 P0, 10 P1, 6 P2 and 2 P3**.

### Q1. Define power quality, voltage quality, current quality, continuity and PCC. Why is PQ important today? — P0, 5 marks

**Answer:** Power quality describes whether electrical power is suitable for the equipment using it. A supply must allow the connected equipment to operate correctly, as well as deliver energy. Voltage quality concerns the supplied voltage waveform and operating conditions. Current quality concerns the waveform drawn by the load. Continuity of supply means maintaining satisfactory service with few interruptions.

The Point of Common Coupling, or PCC, is the shared network connection through which one consumer’s disturbance can affect others. A nonlinear load may draw distorted current even when the applied voltage is sinusoidal. This explains why good supply voltage alone does not guarantee good current quality.

PQ has become important because modern systems contain many power converters, computer supplies and adjustable-speed drives. These devices can generate distortion and can themselves be sensitive to disturbances. For example, a brief sag may reset a controller or trip a drive, causing data loss and a lengthy production stoppage. PQ therefore matters throughout transmission, distribution and consumer utilization, including the response of the final equipment.

### Q2. Differentiate impulsive and oscillatory transients, with causes, effects and waveforms. — P0, 5 marks; repeated PYQ form

**Answer:** A transient is a sudden, non-power-frequency change superimposed on the normal voltage or current. It is brief, but its peak or rapid variation can disturb connected equipment.

An impulsive transient has one predominant polarity. It appears as a sharp positive or negative spike with a fast rise and short decay. Lightning is a representative cause. The high peak can damage electronic components or corrupt stored data, so suitable surge protection is used to limit its effects.

An oscillatory transient alternates in polarity about the undisturbed waveform. Capacitor-bank or line switching can start an exchange of stored energy between network inductance and capacitance. This produces rapid ringing that gradually decays as energy is dissipated. The oscillations can stress equipment, cause data errors and shorten equipment life. Switching-transient suppression can reduce this disturbance.

The main distinction is the shape and polarity of the transient component: an impulse is predominantly one-directional, while an oscillatory transient repeatedly changes direction.

**Diagrams to draw:** Show a normal sine wave with one narrow spike, then another with decaying ringing. Label voltage, time and transient duration.

### Q3. A plant experiences 0.65 pu voltage for 20 cycles during motor starting. Identify the disturbance, explain its cause, and give two effects and suitable mitigation. — P0, 5 marks

**Answer:** The event is a voltage sag, which is a temporary reduction in RMS voltage to 0.1–0.9 pu for 0.5 cycle–1 minute. The given 0.65 pu voltage and 20-cycle duration fall within these limits. The voltage remaining is 65% of nominal, so the reduction is 35%.

During starting, a motor draws a large current. The upstream network has impedance, and the increased current produces a larger voltage drop across it. Consequently, the voltage available at the plant falls until the starting condition passes. The supply is still present, but its reduced voltage may be unsuitable for sensitive equipment.

An adjustable-speed drive may trip and a contactor may drop out. Either response can interrupt production, and the process may require restarting even after voltage recovers.

A DVR can inject compensating voltage in series to maintain the sensitive load voltage. A suitably rated UPS can support critical equipment using stored energy during the sag.

**Diagram to draw:** Show normal cycles, 20 smaller cycles and recovery. Label the 0.65 pu remaining voltage and event duration.

### Q4. Identify these events: 0.8 pu for 10 s; 0.8 pu for 2 min; 1.15 pu for 20 s; 1.15 pu for 2 min; and 0.05 pu for 2 s, 10 s and 2 min. — P0, 5 marks

**Answer:** Voltage events are identified from both their remaining RMS magnitude and duration. The same voltage can belong to different classes when its duration changes. A nearly complete loss of voltage is classified as an interruption.

The **0.8 pu event lasting 10 s is a sag**, because its voltage lies within 0.1–0.9 pu and its duration is below one minute. At the same magnitude for **2 min, it is under-voltage**, because the reduced voltage persists beyond the short-duration boundary.

The **1.15 pu event lasting 20 s is a temporary swell**, since both values fit that swell subclass. If 1.15 pu continues for **2 min, it is over-voltage**, because voltage above 1.1 pu persists for more than one minute.

All three **0.05 pu events are interruptions**, since their voltage is below 0.1 pu. The **2 s event is momentary**, within the 0.5-cycle–3-s region. The **10 s event is temporary**, lasting more than 3 s but no more than one minute. The **2 min event is sustained**.

Thus, one minute separates short and sustained conditions, while three seconds separates momentary and temporary interruptions in the working class chart.

### Q5. Distinguish voltage swell from over-voltage, including causes and effects. — P0, 4 marks

**Answer:** A voltage swell is a temporary increase in RMS voltage to 1.1–1.8 pu overall, lasting 0.5 cycle–1 minute. Its maximum magnitude depends on the duration subclass. Over-voltage is a sustained rise above 1.1 pu for more than one minute. Both involve increased voltage, but their duration distinguishes them.

A swell may occur when a large load is disconnected. The current and upstream voltage drop decrease, allowing the load voltage to rise. Capacitor switching or a single-line-to-ground fault can also produce a swell; during the fault, voltage may rise on the unfaulted phases.

Over-voltage may result from incorrect transformer taps or light loading after the voltage has been raised to compensate for heavier demand. The high voltage then remains beyond the short-event period.

Both conditions stress insulation and power-supply components. They may cause overheating, control errors or equipment damage. The cause and duration therefore help determine how the problem should be addressed.

**Diagrams to draw:** Show increased-amplitude sine-wave regions, with the swell ending within one minute and over-voltage extending beyond it.

### Q6. Explain voltage fluctuation and flicker, including the significance of the fluctuation frequency. — P0, 3 marks

**Answer:** Voltage fluctuation is a systematic or random variation in the amplitude envelope of the supply voltage. Changing load demand changes current and the voltage drop across network impedance. Arc furnaces are a typical source because their real and reactive power demand varies.

The taught typical range is approximately 0.95–1.05 pu, with envelope frequency below 25 Hz. This is the rate of amplitude variation; it does not mean that the fundamental supply frequency becomes 25 Hz.

Flicker is the resulting visible variation in lamp brightness. Fluctuation is therefore the electrical cause, while flicker is its lighting effect. Strengthening the supply or compensating rapidly varying reactive demand can reduce the problem.

### Q7. Explain notching and electrical noise. How can their waveforms and remedies be distinguished? — P0, 5 marks

**Answer:** Notching and electrical noise are unwanted disturbances of the voltage or current waveform. Their appearance and originating mechanisms help distinguish them.

Notching consists of periodic narrow cuts in the voltage waveform. It is associated with converter commutation, meaning the transfer of current from one conducting device or phase to another. Because the process repeats during converter operation, the notches recur at corresponding positions in successive cycles. They can cause data errors, system halts and stress in capacitive components. A converter input line reactor can reduce line notching.

Electrical noise is unwanted voltage or current superimposed on the power waveform or carried by neutral and signal conductors. The taught definition covers broadband content below 200 kHz, excluding disturbances better classified as harmonics or transients. Switching electronics, arcing and radio interference are possible sources. Noise may disrupt communication and control signals; filtering, isolation and proper grounding help reduce it.

**Diagrams to draw:** Show repeated inward cuts for notching and small irregular disturbances for noise. The repeated cuts distinguish notching from irregular interference.

### Q8. Explain harmonic generation, distinguish harmonics from interharmonics, and state effects and mitigation. — P0, 5 marks

**Answer:** A harmonic is a sinusoidal component whose frequency is an integer multiple of the fundamental supply frequency. The relationship is:

$$
f_h=h f_1.
$$

Here, (f_1) is fundamental frequency, (h) is harmonic order and (f_h) is component frequency, measured in hertz. On a 50 Hz supply, the third harmonic is 150 Hz. An interharmonic has a non-integer frequency ratio; 175 Hz is an example on that supply.

Harmonics arise mainly because nonlinear loads draw current that does not follow voltage proportionally. Rectifiers, ASDs and SMPS can draw nonsinusoidal current even from sinusoidal voltage. Transformer saturation and arcing provide other sources of nonlinear behaviour. The distorted periodic waveform can be represented by adding harmonic components to the fundamental.

Harmonics produce additional losses and heating, capacitor overloading and vibration. They may also interfere with communication and cause protection or metering errors. Suitable harmonic filtering reduces the distortion.

Harmonic distortion is a continuing waveform feature associated with repeated load behaviour. A transient, by comparison, is a brief disturbance superimposed on the waveform.

### Q9. A current has a 100 A fundamental, 6 A third harmonic and 8 A fifth harmonic. Calculate THD, neglecting other harmonics. — P0, 5 marks

**Answer:** Current THD measures the combined RMS higher-harmonic current relative to the RMS fundamental current. Since other harmonics are neglected, only the given third and fifth harmonics enter the numerator:

$$
\mathrm{THD}_I(\%)=
\frac{\sqrt{\sum_{h=2}^{h_{\max}}I_h^2}}{I_1}\times100.
$$

Here, (I_1) is fundamental RMS current and (I_h) is the RMS current of harmonic order (h), both in amperes. The 100 A fundamental provides the reference and is excluded from the numerator.

First square the higher-harmonic magnitudes, add them and take the square root:

$$
\sqrt{6^2+8^2}=\sqrt{36+64}=10\ \mathrm{A}.
$$

This gives their combined RMS magnitude; simply adding 6 A and 8 A would not follow the THD formula. Now divide by the fundamental and convert the ratio to a percentage:

$$
\mathrm{THD}_I=\frac{10}{100}\times100=\boxed{10\%}.
$$

The combined harmonic RMS current is therefore 10% of the fundamental current. This exceeds Sir’s simplified class criterion of current THD below 5%.

### Q10. State the purpose of IEEE 519 and the specific point required for this midsem. — P0, 2 marks

**Answer:** IEEE 519 concerns limiting harmonic distortion to acceptable levels so that nonlinear loads do not create excessive distortion in the electrical system. For this midsem, Sir’s simplified stated criterion is that current THD should be less than 5%.

THD depends on the higher-harmonic current magnitudes relative to the fundamental current. The 5% figure should therefore be stated as the taught class criterion, rather than presented as a universal limit for every application.

### Q11. Define a balanced three-phase supply and voltage unbalance. Can equal magnitudes alone establish balance? — P0, 3 marks

**Answer:** A balanced three-phase supply has three equal phase-voltage magnitudes separated by 120°. Both conditions must be satisfied. Voltage unbalance occurs when the magnitudes are unequal, the angular separations differ from 120°, or both.

Equal magnitudes alone cannot establish balance, because equal-length voltage phasors may still have incorrect angular separation. Uneven single-phase loading and unequal supply impedances are common causes. They produce unequal voltage drops and can make the load voltages unbalanced.

Unbalance can cause motor heating, increased losses and reduced equipment life.

**Diagram to draw:** Show three equal phasors 120° apart beside a set with unequal lengths and/or unequal angular separation.

### Q12. Explain ferroresonance and relate it to a ferroresonant transformer. — P0, 5 marks; 2025-style question

**Answer:** Ferroresonance is nonlinear resonance involving system capacitance and an iron-core inductance. The inductance is nonlinear because transformer core saturation changes its magnetizing behaviour. Switching may establish or excite a condition in which magnetizing impedance is effectively in series with capacitance.

The series-RLC interpretation helps explain the resulting high voltage and current:

$$
Z=R+j(X_L-|X_C|),\qquad I=\frac{E}{Z}.
$$

Here, (Z) is impedance, (R) is resistance, (X_L) is inductive reactance and (|X_C|) is capacitive-reactance magnitude, all in ohms. (E) is applied voltage, (I) is current and (j) is the imaginary operator.

When (X_L=|X_C|), the reactive terms cancel, leaving (ZR). If resistance is small, current becomes large and produces high voltages across the reactive elements. The nonlinear iron core can also cause harmonic magnification and irregular waveforms.

Unwanted network ferroresonance stresses or damages equipment. A ferroresonant or constant-voltage transformer deliberately uses a saturated core and resonant capacitor to maintain approximately constant output within its operating range. The device therefore uses controlled ferroresonant action for voltage conditioning.

**Diagram to draw:** Show the source, resistance, iron-core inductance and capacitor in series; label current and inductive voltage.

### Q13. Define the five reliability indices and explain their denominators. — P1, 5 marks

**Answer:** Reliability indices describe interruption frequency, lost service duration and availability. Let (N_T) be all customers served, (N_i) the customers interrupted in outage (i), (U_i) its duration, and (N_A) the distinct customers affected. Define (C=N_i) as customer interruptions and (D=N_iU_i) as customer-interruption duration.

The **System Average Interruption Frequency Index, SAIFI**, is (C/N_T). It gives interruptions per served customer. The **System Average Interruption Duration Index, SAIDI**, is (D/N_T). It gives outage minutes or hours per served customer. Both include customers who experienced no outage.

The **Customer Average Interruption Frequency Index, CAIFI**, is (C/N_A). It gives interruptions per distinct affected customer; each affected customer is counted once in the denominator, even after repeated outages.

The **Customer Average Interruption Duration Index, CAIDI**, is (D/C). It gives minutes or hours per customer interruption, rather than total outage time per distinct customer.

The **Average System Availability Index, ASAI**, is (1-D/(N_TH)), where (H) is study duration in hours and (D) uses customer-hours. It gives the available fraction of demanded customer service and is dimensionless. Keeping these denominators and time units consistent is essential.

### Q14. Calculate reliability indices for the one-day class table below, with 50,000 customers served. — P1, 5 marks; 2025 numerical style

| Outage | Customers interrupted, (N_i) | Duration, (U_i), min |
| --- | --- | --- |
| 1 | 10 | 90 |
| 2 | 1,000 | 20 |
| 3 | 2 | 175 |
| 4 | 1 | 120 |
| 5 | 1 | 38 |

**Answer:** Reliability indices account for how many customers lose service and how long they remain without it. First calculate (C), the total customer interruptions, and (D), the customer-interruption duration. Multiply each outage’s customer count by its duration before adding the products:

$$
\begin{aligned}
C&=\sum N_i=10+1000+2+1+1=1014\ \text{customer interruptions},\\
D&=\sum N_iU_i\\
 &=10(90)+1000(20)+2(175)+1(120)+1(38)\\
 &=900+20{,}000+350+120+38\\
 &=21{,}408\ \text{customer-min}=356.8\ \text{customer-h}.
\end{aligned}
$$

Here, (N_T=50{,}000) customers and the study period is (H=24) hours. SAIFI and SAIDI average over all served customers, while CAIDI averages over customer interruptions. Substituting the totals:

$$
\mathrm{SAIFI}=\frac{C}{N_T}
=\frac{1014}{50{,}000}
=\boxed{0.02028\ \text{interruptions/customer for the day}}.
$$

$$
\mathrm{SAIDI}=\frac{D}{N_T}
=\frac{21{,}408}{50{,}000}
=\boxed{0.42816\ \text{min/customer for the day}}.
$$

$$
\mathrm{CAIDI}=\frac{D}{C}
=\frac{21{,}408}{1014}
\approx\boxed{21.11\ \text{min/interruption}}.
$$

For ASAI, convert customer-minutes to customer-hours by dividing by 60. Use (50{,}000) demanded customer-hours in the denominator, because the table covers one day:

$$
\mathrm{ASAI}=1-\frac{D_{\mathrm{hours}}}{N_TH}
=1-\frac{356.8}{50{,}000(24)}
=0.9997027
\approx\boxed{99.9703\%}.
$$

Thus, approximately 99.9703% of the demanded customer service was available during the day. The table does not show whether the same customers experienced multiple outages, so **CAIFI cannot be uniquely determined**. Its formula is (C/N_A), where (N_A) is the distinct affected-customer count. If the five groups are explicitly assumed to be disjoint, (N_A=1014) and CAIFI is 1 interruption per affected customer.

As a check, using unrounded values, ((1014/50{,}000)(21{,}408/1014)=0.42816=).

### Q15. Why is CAIFI not the number of outage events divided by customers interrupted? If Question 14 additionally states 1,000 distinct affected customers, find CAIFI. — P1, 3 marks

**Answer:** CAIFI measures the average number of interruptions experienced by customers who were affected at least once. Its numerator must count customer interruptions, not outage events, because an event affecting many customers has a larger impact than an event affecting one customer. Its denominator counts each affected customer only once, even if that customer appears in several outage records.

From Question 14, (C=1014) customer interruptions and (D=21{,}408) customer-minutes. With the additional information (N_A=1000) distinct customers:

$$
\mathrm{CAIFI}=\frac{C}{N_A}
=\frac{1014}{1000}
=\boxed{1.014\ \text{interruptions/affected customer for the day}}.
$$

The value exceeds one because some customers experienced repeated interruptions. It is an average count, not a probability. Without the extra distinct-customer count, this calculation would be underdetermined.

### Q16. Explain positive-, negative- and zero-sequence components and write the basic phase-reconstruction relations. — P1, 5 marks

**Answer:** Symmetrical components provide a way to describe an unbalanced three-phase system using three simpler component sets. Adding their contributions reconstructs the original phase voltages, including their magnitudes and angles.

Positive sequence consists of three equal magnitudes separated by 120° in normal A–B–C order. Negative sequence also has equal magnitudes separated by 120°, but its order is reversed to A–C–B. Zero sequence consists of three equal components that are all in phase, so there is no angular separation between them.

Let (V_0,V_1,V_2) represent the phase-A zero-, positive- and negative-sequence voltage phasors. The operator (a=1^) rotates a phasor through 120°. The phase-reconstruction relations are:

$$
\begin{aligned}
V_A&=V_0+V_1+V_2,\\
V_B&=V_0+a^2V_1+aV_2,\\
V_C&=V_0+aV_1+a^2V_2.
\end{aligned}
$$

All voltages use the same units. Each equation adds the three sequence contributions for that phase.

A normally ordered balanced supply contains only positive sequence. An unbalanced set can contain negative and/or zero sequence. Negative sequence is particularly important because it is associated with motor heating.

**Diagrams to draw:** Show equal phasors in A–B–C order, equal phasors in A–C–B order, and three coincident in-phase phasors.

### Q17. Explain how unequal single-phase loading causes voltage unbalance. State other causes, effects and remedies. — P1, 5 marks

**Answer:** Voltage unbalance is a condition in which phase-voltage magnitudes are unequal, their angular separations differ from 120°, or both. Unequal single-phase loading is a common cause.

When different loads are connected to the phases, they draw different currents. The supply network has impedance, so these currents produce unequal voltage drops. Consequently, the voltages reaching the load become unequal. This explains how unequal external loading can disturb an otherwise balanced supply.

Unequal line or transformer impedances can produce a similar result. Incomplete transposition leaves conductors in different electrical conditions along the line. Faults, open phases, faulty connections and blown capacitor fuses are other possible causes.

An unbalanced set can contain negative- and zero-sequence components. Negative sequence is particularly associated with motor heating. Increased losses and degraded performance may require equipment derating and can shorten its life.

Redistributing single-phase loads more evenly reduces load-related unbalance. Open phases, faulty connections and unequal impedances should also be corrected at their source so that the phase conditions become more nearly equal.

### Q18. Classify PQ problems and explain the main disturbance types with their sources. — P1, 10 marks; 2025-style question

**Answer:** Power-quality problems are disturbances in supply or load behaviour that interfere with satisfactory equipment operation. They can be classified by their behaviour with time, the electrical quantity affected and their origin.

Classification by time separates brief events, such as transients and short-duration RMS variations, from continuing conditions, such as harmonic distortion and unbalance. Classification by quantity distinguishes voltage problems, including sag, swell, fluctuation and distortion, from current problems such as harmonic or unbalanced current. Frequency variation affects the fundamental frequency. Classification by origin separates supply-side faults and switching from load-side nonlinear or rapidly varying demand.

Among brief disturbances, an **impulsive transient** is a sharp one-polarity spike commonly caused by lightning. An **oscillatory transient** is decaying ringing caused when capacitor or line switching excites network inductance and capacitance.

A **sag** is a short RMS voltage reduction. Faults and motor starting increase upstream voltage drop and can cause it. A **swell** is a short voltage rise that may follow large-load disconnection or capacitor switching. An **interruption** reduces voltage to nearly zero when fault clearing or equipment failure disconnects supply. **Under-voltage and over-voltage** persist beyond one minute and may result from heavy loading, incorrect taps or sustained regulation problems.

**Voltage fluctuation** repeatedly changes the envelope as arc furnaces or cyclic loads draw varying power. **Unbalance** means unequal phase magnitudes and/or angular separation, often caused by uneven loading or unequal impedances.

**Harmonics** are integer-multiple frequency components produced by nonlinear converters, SMPS and saturating transformers. **Notching** creates repeated voltage cuts during converter commutation. **Noise** adds unwanted irregular content from switching electronics, arcing or radio interference. **Power-frequency variation** occurs when generation–load imbalance changes generator speed.

These classifications overlap. An arc furnace, for example, is a load-side source whose varying current produces voltage fluctuation at the common supply. Naming the disturbance together with its source explains both what changes and why.

### Q19. Organize the causes and effects of PQ problems, using examples. — P1, 5 marks

**Answer:** Power-quality problems are electrical disturbances that prevent connected equipment from operating satisfactorily. Their causes can be organized into natural or network events, operational changes and load-related behaviour.

Natural and network causes include lightning, severe weather, faults and equipment failure. Operational causes include switching transformers, capacitors, feeders and heavy loads. Load-related causes include nonlinear converters and rapidly varying arc loads, which introduce distortion or changing demand.

Their effects fall into three broad groups. Operational effects include controller resets, equipment malfunction, lost data and production stoppage. For example, a sag can trip a drive and stop a process. Electrical effects include heating, increased losses, capacitor overloading and insulation stress. Harmonics and unbalance cause heating, while high-voltage events stress insulation.

Protection, metering and communication can also be affected through false operation, measurement errors and interference. These problems may occur even when the supply remains connected.

The practical loss depends on equipment sensitivity as well as disturbance severity. A brief electrical event may require a lengthy restart or spoil material, so the production interruption can last much longer than the original disturbance.

### Q20. Compare short- and long-duration voltage variations with examples. — P1, 5 marks

**Answer:** Short- and long-duration voltage variations are changes in RMS supply voltage distinguished mainly by the one-minute boundary. Magnitude is also required, because low voltage, high voltage and interruption represent different conditions.

A **sag** leaves 0.1–0.9 pu for 0.5 cycle–1 minute; motor starting is a typical example. If voltage below 0.9 pu persists beyond one minute, outside the interruption region, it is **under-voltage**. Sustained heavy feeder loading can cause this condition.

A **swell** raises voltage to 1.1–1.8 pu overall for 0.5 cycle–1 minute. Large-load disconnection can cause it. If voltage remains above 1.1 pu for more than one minute, it is **over-voltage**, as may occur with incorrect transformer taps.

A **short interruption** reduces voltage below 0.1 pu for up to one minute, for example during brief recloser opening. A **sustained interruption** leaves voltage nearly zero for more than one minute, as in an extended fault-related outage.

Voltage remains present during sag or under-voltage, while an interruption represents a loss of continuity. Similar waveform shapes therefore require magnitude and duration labels for correct identification.

**Diagrams to draw:** Show the low-, high- and nearly zero-voltage intervals, clearly marking their durations.

### Q21. Explain the five types of waveform distortion. — P1, 5 marks

**Answer:** Waveform distortion is a steady-state departure from the ideal power-frequency sine wave. The five main types are DC offset, harmonics, interharmonics, notching and noise. Each changes the waveform differently.

**DC offset** introduces a DC component and makes the waveform’s average nonzero. Half-wave rectification or converter asymmetry can cause it. The resulting magnetic bias can lead to transformer saturation and heating.

**Harmonics** are sinusoidal components at integer multiples of the fundamental. Nonlinear loads generate them, and their presence can increase losses and heating. **Interharmonics** have frequencies that are not integer multiples. Cycloconverters, induction furnaces and arcing devices are representative sources.

**Notching** consists of repeated narrow voltage cuts associated with converter commutation. These cuts can disturb electronic equipment and stress capacitive components. **Noise** is unwanted broadband electrical content superimposed on power or signal conductors. Switching electronics and arcing can introduce it, causing communication and data errors.

The distinguishing feature may therefore be a shifted average, additional frequency components, repeated cuts or irregular interference.

**Diagram cues:** Shift the sine wave for DC offset, distort its shape for harmonics, show repeated cuts for notching and add irregular disturbances for noise.

### Q22. Explain the mechanism, causes and effects of power-frequency variation. — P1, 3 marks

**Answer:** Power-frequency variation is a deviation of the fundamental supply frequency from its nominal value. Frequency is linked to generator speed. When generation and load become unbalanced, the resulting torque imbalance changes generator speed and therefore electrical frequency.

Faults in an isolated system, loss of synchronism or islanding can cause such disturbances. Islanding means that part of the network separates and operates as an isolated supply system. Frequency deviations affect loads and transformers and can impose mechanical stress on generator and turbine shafts.

Harmonics add frequency components, whereas this disturbance changes the fundamental itself.

**Diagram to draw:** Change the spacing between successive cycles to show the frequency variation.

### Q23. Explain and compare voltage-sag mitigation methods with operating principles and suitable applications. — P2, 10 marks; repeated in 2023/2025

**Answer:** Voltage-sag mitigation aims to keep sensitive equipment operating when incoming voltage falls. The main approaches are voltage injection, transfer to a healthy source, stored-energy support and voltage conditioning. Selection depends on sag magnitude and duration, load sensitivity and the available supply arrangement.

An **active series compensator** adds controlled voltage between supply and load. A **Dynamic Voltage Restorer (DVR)** detects the sag and commands its power circuit to inject compensation:

$$
\mathbf V_{\mathrm{load}}
=\mathbf V_{\mathrm{supply}}+\mathbf V_{\mathrm{injected}}.
$$

These are voltage phasors in volts or a common pu base. The injected voltage requires the correct magnitude and phase to maintain near-normal load voltage. A DVR suits sensitive loads exposed to sags, provided its compensation and energy limits are sufficient.

A **static transfer switch** uses solid-state devices to move the load to an independent healthy source. It operates on a break-before-make basis, disconnecting the disturbed source before connecting the alternative. It provides rapid transfer near a critical load but requires a healthy redundant supply.

A **fast transfer scheme** detects source deterioration and rapidly transfers to a standby feeder, potentially using coordinated high-speed breakers. “Fast” describes transfer performance, while “static” identifies the switching technology. Both arrangements depend on a suitable alternative source.

A **UPS** uses stored energy to support an inverter when incoming power is inadequate. In an online UPS, the inverter continuously supplies the load. In an offline UPS, utility power normally supplies it, followed by transfer to the battery-fed inverter after disturbance detection. UPS support protects critical electronics through sags and interruptions, with available energy limiting duration.

A **constant-voltage transformer (CVT)** uses a saturated core and resonant capacitor to maintain approximately constant output within its operating range. It conditions the existing supply through its magnetic-capacitor arrangement.

Thus, a DVR injects voltage, transfer schemes change source, a UPS supplies stored energy and a CVT stabilizes voltage.

**Diagrams to draw:** Show DVR series injection, two source paths feeding a transfer-switch load, an online UPS rectifier/DC/inverter path, or a CVT with its resonant capacitor.

### Q24. Give a cause, effect and remedy for each major PQ disturbance. — P2, 10 marks; repeated reason/remedy form

**Answer:** Power-quality disturbances affect equipment through abnormal voltage, distorted waveforms or lost continuity. A suitable remedy must address the cause and act within the event’s duration.

An **impulsive transient** from lightning produces a sharp peak that can damage electronics. Surge arresters or suppressors limit its effects. An **oscillatory transient** results from capacitor or line switching and causes ringing and equipment stress. Switching-transient suppression, including pre-insertion resistance or inductance, can reduce it.

A **sag** occurs when faults or motor starting increase upstream voltage drop, potentially tripping drives. DVR compensation, UPS support or healthy-source transfer can protect the load. A **swell** following load disconnection or capacitor switching raises voltage and stresses insulation. Series compensation can oppose the excess voltage.

An **interruption** caused by fault-related protective opening removes supply and stops equipment. Stored-energy support or an independent healthy supply can maintain operation.

**Under-voltage** from excessive loading maintains low voltage and can cause motor heating. Suitable regulation, tap adjustment and reduced feeder drop address it. **Over-voltage** from incorrect taps or light loading maintains high voltage and stresses components. Correcting regulator or tap settings and coordinating capacitor switching with load can reduce it.

**Voltage fluctuation** from changing arc-furnace demand causes lamp flicker. Strengthening the supply or compensating changing reactive demand helps. **Unbalance** from uneven phase loading produces unequal voltage drops and motor heating. Redistributing single-phase loads and correcting faulty phase connections reduces it.

**Harmonics** from nonlinear loads cause distorted current, heating and capacitor overloading; suitable harmonic filtering reduces them. **Notching** during converter commutation creates voltage cuts that disrupt equipment; a converter input line reactor can reduce the disturbance. **Noise** from switching or arcing causes interference and data errors; filtering, isolation and proper grounding help control it.

Response time matters: mechanical tap regulation can correct sustained voltage problems but cannot effectively address a sag lasting only a few cycles.

### Q25. Explain the three categories of equipment sensitivity to voltage sags, with examples. — P2, 5 marks

**Answer:** Equipment sensitivity to voltage sags describes how the characteristics of a sag determine whether equipment continues operating or malfunctions. The course divides equipment into three broad categories.

Equipment sensitive mainly to **magnitude** responds when voltage crosses a critical level. Examples include undervoltage relays, process controls, motor-drive controls and automated or semiconductor-manufacturing machines. These are the taught examples for a question asking about equipment sensitive “only” to sag magnitude.

Equipment sensitive to **both magnitude and duration** includes electronic power supplies. Such a supply may tolerate a brief reduction using its internal support. If the voltage is too low for too long, that support becomes inadequate, the output falls and the equipment may reset or trip.

Equipment sensitive to **other characteristics** may respond to phase shift, three-phase unbalance, superimposed oscillations or the point on the waveform when the sag begins or ends. Events with identical RMS magnitude and duration can therefore produce different responses.

These categories describe the main influences rather than identical behaviour in every device. Actual tolerance also depends on equipment settings and its application.

### Q26. Name four industrial harmonic sources and state how they generate distortion. — P2, 2 marks

**Answer:** Industrial harmonic sources are devices whose nonlinear behaviour produces distorted current. Four examples are transformers, arc furnaces, power converters and adjustable-speed drives.

Transformer core saturation produces nonlinear magnetizing current. Arc furnaces generate distortion because the arc has nonlinear and varying electrical behaviour. Power converters draw nonsinusoidal current by conducting or switching during selected portions of the AC cycle. Adjustable-speed drives can also draw distorted current through their rectifier inputs.

### Q27. Explain the response of a rectifier-fed ASD to a voltage sag. — P2, 5 marks; recurring application

**Answer:** A rectifier-fed adjustable-speed drive converts the AC supply through a rectifier and DC link before an inverter supplies the motor. The DC-link capacitor smooths the rectified voltage and stores energy for the load.

During normal operation, input diodes conduct when the available rectified supply voltage exceeds the capacitor voltage. Between charging intervals, the capacitor supplies the inverter and motor.

During a sag, reduced AC voltage may prevent normal capacitor recharging for part of the event. The capacitor continues supplying energy, so the DC-link voltage falls. A brief or shallow sag may end before this reduction becomes serious, allowing the drive to continue operating.

A deeper or longer sag can lower the DC-link voltage below the undervoltage trip threshold and stop the drive. The motor need not have lost all speed before the protection operates. Rectifier-fed electronic supplies behave similarly and may reset or shut down when DC support becomes inadequate.

Thus, sag magnitude and duration influence whether the equipment rides through or interrupts the process.

**Diagram to draw:** Show AC → rectifier → DC-link capacitor → inverter → motor, with the AC sag and falling DC voltage crossing a labelled trip threshold.

### Q28. Choose protection for: (a) critical controls needing continuity through brief outages, (b) a load with two independent feeders, and (c) a sensitive load exposed to sags on one feeder. Compare online and offline UPS operation. — P2, 5 marks

**Answer:** Protection should match the load’s continuity requirement and the available sources. The three situations require different ways of supporting the load when its normal supply becomes inadequate.

For **critical controls needing continuity through brief outages**, a suitably rated UPS is appropriate. Its stored energy supports the inverter and prevents a brief loss of input power from stopping the controls. The available energy must cover the required support interval.

For **a load with two independent feeders**, a static transfer switch or suitable fast transfer scheme can move the load from the disturbed feeder to the healthy one. The alternative feeder must remain suitable during the event.

For **a sensitive load exposed to sags on one feeder**, a DVR can inject the missing series voltage to maintain the required load voltage. Its compensation rating and energy capability must suit the sag.

In an **online UPS**, the inverter supplies the load continuously through the DC stage, which the battery supports during failure. In an **offline UPS**, utility power normally supplies the load, followed by transfer to the battery-fed inverter after disturbance detection. This difference helps assess suitability for the equipment’s continuity needs.

### Q29. Give the basic IEC classification of electromagnetic disturbances, with examples. — P3, 5 marks; one-off PYQ

**Answer:** Electromagnetic disturbances are unwanted electrical or electromagnetic effects that can interfere with equipment operation. The basic IEC classification groups them by frequency range and by how they reach the equipment. Electrostatic discharge is treated separately.

**Conducted low-frequency disturbances** travel through electrical connections. Examples include harmonics, supply-voltage and frequency changes, and DC components in AC networks. **Radiated low-frequency disturbances** involve electric or magnetic fields coupling to equipment.

**Conducted high-frequency disturbances** include continuous high-frequency signals and transient disturbances carried by conductors. **Radiated high-frequency disturbances** reach equipment through electromagnetic fields, which may be continuous, modulated or pulsed.

**Electrostatic discharge** occurs when accumulated electric charge is suddenly discharged. It is identified separately from the four groups formed by combining coupling path and frequency range.

The important distinction is therefore whether a disturbance is carried through conductors or coupled through fields, followed by its frequency category. Giving each group with a representative phenomenon explains the classification clearly without extending the answer into test procedures.

### Q30. Write short notes on any two: sag with harmonics; lamp PQ comparison; sensitive process/machine; loading and ASD ride-through. — P3, 4 marks

**Answer:** Choose any two of the following notes for the four-mark answer.

**Sag with harmonics:** A sag with harmonics is a reduction in RMS voltage accompanied by waveform distortion. The voltage during the event is lower, but its cycles are also distorted rather than being clean smaller sine waves. Both features must therefore appear in the waveform sketch. Draw normal voltage, a lower-amplitude distorted interval and recovery. Label the remaining RMS voltage, sag duration and harmonic distortion so that the combined event is clearly distinguished from a clean sag.

**Lamp comparison:** An ordinary incandescent lamp is approximately resistive in steady operation. Under sinusoidal voltage, its current is approximately sinusoidal and produces little harmonic distortion. A fluorescent lamp needs a ballast, and its current distortion depends on ballast design. Voltage fluctuation can vary incandescent brightness, while high-frequency electronic ballasts reduce low-frequency light variation in fluorescent operation. Thus, the comparison involves both the current drawn by the lamp and its lighting response to the supply conditions.

**Sensitive process/machine:** A sensitive process can stop when a PQ event upsets an essential controller or drive. The electrical supply may recover quickly, but lost material and restart requirements can prolong the production interruption. Draw supply → sensitive controller/drive → machine/process, and label the disturbance and resulting trip. The older question’s wording does not identify a named standard machine. Use this as a conceptual sensitivity diagram showing how one equipment malfunction affects the complete process.

**Loading and ASD ride-through:** Ride-through means continuing operation during a disturbance without a trip. During a sag, the DC-link capacitor supports the drive while its normal charging is reduced. Heavier loading uses the stored energy more quickly, while lighter loading can extend the support time. Some drives permit temporary reduced- or zero-torque operation. Whether a drive continues operating therefore depends on loading, stored energy and control strategy, as well as the magnitude and duration of the incoming voltage sag.

## Final self-check

- [ ]  I can reproduce the magnitude–duration boundaries and draw the main waveforms.
- [ ]  I can explain all P0 definitions, THD, unbalance and ferroresonance without reading.
- [ ]  I can compute (C), (D), all identifiable reliability indices and the ASAI time conversion.
- [ ]  I can distinguish all three sequence sets and explain unbalance and frequency mechanisms.
- [ ]  I can write a complete sag-mitigation answer and a cause–effect–remedy table.
- [ ]  I can explain sensitivity categories, name four harmonic sources and reconstruct ASD sag response.
- [ ]  I have attempted the questions before reading the answers and marked weak topics for another pass.
- [ ]  After core revision, I can attempt all three supplied midsem papers; P3 receives the final quick pass.