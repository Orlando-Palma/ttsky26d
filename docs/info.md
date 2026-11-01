<!---
This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.
-->

## How it works

This project implements a fully differential **2nd-order Continuous-Time Sigma-Delta ($\Sigma\Delta$) Modulator** in the `sky130A` CMOS technology node, designed with an **8-tap Finite Impulse Response (FIR) feedback DAC** and **Excess Loop Delay (ELD)** compensation.

### Architecture Highlights
* **Loop Filter:** 2nd-order active-RC continuous-time integrator cascade using ultra-high resistance polysilicon resistors (`sky130_fd_pr__res_xhigh_po`, $R_{\text{in}} = 150\text{ k}\Omega$, $R_{12} = 66.7\text{ k}\Omega$) and Metal-Insulator-Metal (MiM) capacitors ($C_1 = 3.33\text{ pF}$, $C_2 = 2.00\text{ pF}$).
* **Amplifiers (`OTA1` & `OTA2`):** Two-stage Miller-compensated OTAs with PMOS LVT input differential pairs (`sky130_fd_pr__pfet_01v8_lvt`), Class-A NMOS output stages, and continuous-time Common-Mode Feedback (`CMFB`).
* **Quantizer & ELD Path:** Regenerative StrongARM latch comparator clocked at $f_s = 10\text{ MHz}$ with a direct ELD compensation path around the quantizer.
* **8-Tap FIR Feedback DAC:** Attenuates clock jitter sensitivity and relaxes the first integrator slewing requirements using a weighted resistor array (`[79, 17, 15]/32` inner matrix).

### Post-Design PVT Corner Performance ($f_s = 10\text{ MHz}$, $\text{BW} = 19.53\text{ kHz}$, $\text{OSR} = 256$)

| PVT Corner (`sky130A`) | Input Tone ($f_{\text{in}}$) | Input Level | SNDR | Dynamic Range (DR) | ENOB (DR) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Typical (`tt`, $27^\circ\text{C}$)** | $14.65\text{ kHz}$ | $-12.03\text{ dBFS}$ | $97.59\text{ dB}$ | $109.62\text{ dB}$ | $17.92\text{ bits}$ |
| **Slow-Slow (`ss`, $-40^\circ\text{C}$)** | $14.65\text{ kHz}$ | $-12.02\text{ dBFS}$ | $92.99\text{ dB}$ | $105.01\text{ dB}$ | $17.15\text{ bits}$ |
| **Fast-Fast (`ff`, $85^\circ\text{C}$)** | $14.65\text{ kHz}$ | $-12.03\text{ dBFS}$ | $99.86\text{ dB}$ | $111.89\text{ dB}$ | $18.29\text{ bits}$ |

## How to test

1. **Power Supply & Reference Setup:**
   * Apply $1.8\text{ V}$ to `VPWR` and $0.0\text{ V}$ to `VGND`.
   * Apply a clean $0.90\text{ V}$ DC common-mode voltage to `ua[2]` (`vcm`).
2. **Clocking:**
   * Feed a low-jitter $10\text{ MHz}$ square-wave clock ($1.8\text{ V}$ CMOS levels) into the `clk` pin.
3. **Analog Signal Injection:**
   * Apply a differential audio/baseband signal ($f_{\text{in}} \le 19.53\text{ kHz}$) centered around $0.90\text{ V}$ to `ua[0]` (`vin_p`) and `ua[1]` (`vin_n`). A $-12\text{ dBFS}$ test tone corresponds to $V_{\text{diff,peak}} \approx 225\text{ mV}$.
4. **Digital Bitstream Capture:**
   * Capture the 1-bit pulse-density modulated (PDM) output stream at $10\text{ MSps}$ from `uo_out[0]` (`bitstream_p`) and `uo_out[1]` (`bitstream_n`) using a logic analyzer or FPGA, and apply a $\text{sinc}^3$ decimation filter or Hann-windowed FFT ($\text{OSR} = 256$).

## External hardware

* Low-noise $0.90\text{ V}$ voltage reference for `ua[2]` (`vcm`).
* Differential low-distortion audio signal generator (or single-ended source with an op-amp/balun differential driver) for `ua[0]` and `ua[1]`.
* Logic analyzer or FPGA acquisition board capable of sampling `uo_out[0]` at $10\text{ MHz}$.
