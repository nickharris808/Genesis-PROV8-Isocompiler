# IsoCompiler Patent Claims Summary

**Total Claims:** 72
**Structure:** 10 frequency bands x 7 claims each + 2 master cross-band integration claims
**Filing Status:** Drafted provisional patent applications per 35 U.S.C. 111(b)

---

## Claim Architecture

Each frequency band has 7 claims covering the complete automated isolation synthesis workflow:

1. **System claim**: An automated system for synthesizing electromagnetic isolation structures at the specified frequency band.
2. **Method claim**: A method for designing isolation structures using FDTD simulation and adjoint-based inverse design at the specified frequency.
3. **Structure claim**: An isolation structure optimized by topology optimization for the specified frequency band, characterized by a binary (0/1) metal pattern.
4. **Optimization claim**: The use of adjoint gradient computation with SIMP penalization and Heaviside projection for isolation structure synthesis at the specified frequency.
5. **Manufacturing claim**: A method of manufacturing the optimized isolation structure with DRC-compliant GDSII output.
6. **Integration claim**: Integration of the synthesized isolation structure into a multi-die chiplet package operating at the specified frequency.
7. **Verification claim**: A method of verifying isolation performance by re-simulation of the synthesized structure within the substrate model.

---

## Band 01: Sub-1 GHz Isolation (Claims 1-7)

**Application:** IoT, low-speed SerDes
**Target frequency range:** Below 1 GHz
**Primary use case:** Isolation between low-power IoT chiplets and digital processing tiles in consumer electronics packages.

Claims 1-7 cover the automated synthesis of isolation structures for sub-1 GHz substrate coupling, including via fence and EBG structures optimized for long-wavelength isolation where substrate dimensions are a small fraction of the wavelength.

---

## Band 02: 1-3 GHz Isolation (Claims 8-14)

**Application:** USB4, PCIe Gen3
**Target frequency range:** 1 GHz to 3 GHz
**Primary use case:** Isolation between high-speed I/O chiplets (USB4, PCIe) and sensitive analog circuitry.

Claims 8-14 cover isolation synthesis at 1-3 GHz, where substrate modes begin to propagate efficiently and via fence spacing becomes a critical design parameter.

---

## Band 03: 3-6 GHz Isolation (Claims 15-21)

**Application:** WiFi 6, PCIe Gen4
**Target frequency range:** 3 GHz to 6 GHz
**Primary use case:** Isolation between WiFi 6 RF front-ends and baseband processors in mobile and networking chiplet packages.

Claims 15-21 cover isolation synthesis in the WiFi and mid-band PCIe frequency range, including mushroom EBG structures whose bandgap overlaps the 5 GHz WiFi band.

---

## Band 04: 6-10 GHz Isolation (Claims 22-28)

**Application:** High-speed ADC, PCIe Gen6
**Target frequency range:** 6 GHz to 10 GHz
**Primary use case:** Isolation between high-speed data converter chiplets and digital signal processing tiles.

Claims 22-28 cover isolation synthesis at frequencies where substrate wavelengths become comparable to chiplet separation distances, requiring carefully optimized EBG or inverse-designed structures.

---

## Band 05: 10-18 GHz Isolation (Claims 29-35)

**Application:** Ku-band satellite
**Target frequency range:** 10 GHz to 18 GHz
**Primary use case:** Isolation between Ku-band transceiver chiplets and digital processing in satellite communications packages.

Claims 29-35 cover isolation synthesis in the Ku-band range, relevant to aerospace and defense applications integrating RF and digital chiplets on a single package.

---

## Band 06: 18-26.5 GHz Isolation (Claims 36-42)

**Application:** 5G FR2 lower band
**Target frequency range:** 18 GHz to 26.5 GHz
**Primary use case:** Isolation between 5G FR2 RF chiplets and baseband/digital chiplets in base station and handset packages.

Claims 36-42 cover isolation synthesis at 5G FR2 lower frequencies, where substrate coupling becomes severe and traditional via fences lose effectiveness.

---

## Band 07: 26.5-40 GHz Isolation (Claims 43-49)

**Application:** 5G FR2 (n257/n261)
**Target frequency range:** 26.5 GHz to 40 GHz
**Primary use case:** Isolation for 5G mmWave handset and base station chiplet packages operating in the n257 (26.5-29.5 GHz) and n261 (27.5-28.35 GHz) bands.

Claims 43-49 cover isolation synthesis at the primary 5G mmWave bands, including inverse-designed freeform structures that outperform rule-of-thumb via fences at these frequencies.

---

## Band 08: 40-60 GHz Isolation (Claims 50-56)

**Application:** 5G FR2 (n260)
**Target frequency range:** 40 GHz to 60 GHz
**Primary use case:** Isolation for 5G n260 (37-40 GHz) and WiGig (57-66 GHz) chiplet packages.

Claims 50-56 cover isolation synthesis at upper mmWave frequencies, where substrate wavelengths are sub-millimeter and isolation structure geometry becomes extremely fine-featured.

---

## Band 09: 60-90 GHz Isolation (Claims 57-63)

**Application:** 112G SerDes, WiGig
**Target frequency range:** 60 GHz to 90 GHz
**Primary use case:** Isolation between ultra-high-speed SerDes chiplets (112 Gbps PAM4) and adjacent dies in data center interconnect packages.

Claims 57-63 cover isolation synthesis for next-generation data center interconnects, where SerDes data rates push fundamental frequencies into the 60-90 GHz range and substrate isolation is a primary limiter of link performance.

---

## Band 10: 90-110 GHz Isolation (Claims 64-70)

**Application:** Automotive radar (ADAS)
**Target frequency range:** 90 GHz to 110 GHz
**Primary use case:** Isolation between 77 GHz automotive radar transceiver chiplets and digital processing in ADAS sensor modules.

Claims 64-70 cover isolation synthesis at automotive radar frequencies, where the combination of high frequency, tight chiplet spacing, and strict automotive reliability requirements makes manual isolation design especially challenging.

---

## Master Cross-Band Integration (Claims 71-72)

**Claim 71:** A system and method for automated synthesis of electromagnetic isolation structures that simultaneously optimizes across multiple frequency bands, producing a single composite isolation layout that meets isolation targets at all specified frequencies within a multi-die chiplet package.

**Claim 72:** A method of manufacturing a multi-die chiplet package incorporating isolation structures synthesized by the system of Claim 71, wherein the isolation structures are exported as DRC-compliant GDSII and integrated into the package layout alongside chiplet placement and routing.

---

## Claim Statistics

| Metric | Value |
|--------|-------|
| Total claims | 72 |
| Independent claims per band | 7 |
| Number of frequency bands | 10 |
| Cross-band integration claims | 2 |
| Synthesis families covered | 5 (via fence, mushroom EBG, fractal EBG, metasurface, inverse design) |
| Optimization methods covered | Adjoint topology optimization, Nevergrad CMA-ES, SIMP + Heaviside |

---

*All claims are draft provisional patent applications. Filing has not yet occurred. Structure geometries are generated by 2D FDTD inverse design. Independent 3D verification is recommended before filing.*
