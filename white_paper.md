# White Paper – DTQEM v12.1

## Dual‑Time Quantum Entanglement Model: From Numerical Simulation to Inverse Calibration

**Author:** Redouane BERRAMDANE  
**Date:** 2025  
**Repository:** [DTQEM-v12.1](https://github.com/your-username/DTQEM-v12.1)

---

## Abstract

DTQEM v12.1 provides an **exact, calibrated, open‑source** simulation of two‑qubit entanglement under realistic decoherence channels (thermal, dephasing, relaxation) and external magnetic fields. The model solves the Lindblad master equation via Liouvillian superoperator exponentiation (`expm(L·t)`), ensuring machine‑precision accuracy and unconditional stability. It implements all standard entanglement and coherence metrics (visibility, concurrence, negativity, purity, von Neumann entropy, fidelity, l1‑norm coherence). An interactive ipywidgets‑based GUI allows real‑time parameter exploration. **Unique inverse‑calibration tools** derive decoherence rates, temperature, or launch angle directly from target visibility – making the model directly usable with experimental data. A **testable unique prediction** is proposed: at launch angle θ = 90°, visibility equals distinguishability for any temperature. Benchmarks verify pure dephasing, relaxation, entropy increase, and Bohr complementarity (V² + D² ≤ 1) with errors below 1e‑12.

---

## 1. Introduction

Quantum entanglement is a key resource for quantum information, but its fragility under environmental decoherence remains a major challenge. DTQEM was developed as a **numerical laboratory** that bridges microscopic quantum dynamics (Lindblad master equation) with macroscopic observables (double‑slit fringe visibility, Bloch vector, density matrix). The model is **not** a heuristic: it is an exact solver of the Lindblad equation for two qubits, with parameters that can be calibrated **inversely** from experimental visibility data.

---

## 2. Core Physical Model

We consider two qubits (four‑dimensional Hilbert space). The system state is described by a density matrix `ρ` (4×4). The Hamiltonian:

\[
H = \frac{g\mu_B B}{2}\, \sigma_z^{(1)}\otimes I_2
\]

describes a magnetic field coupling only to the first qubit (Zeeman effect). Decoherence is introduced via three Lindblad jump operators:

- **Pure dephasing**: \( L_{\phi} = \sqrt{\gamma_{\phi}(T)}\, \sigma_z^{(1)}\otimes I_2 \)
- **Relaxation (emission)**: \( L_{\downarrow} = \sqrt{\gamma_{\downarrow}(T)}\, \sigma_-\otimes I_2 \)
- **Excitation (absorption)**: \( L_{\uparrow} = \sqrt{\gamma_{\uparrow}(T)}\, \sigma_+\otimes I_2 \)

The thermal rates follow Bose–Einstein statistics:

\[
n_{\text{th}} = \frac{1}{\exp(\hbar\omega/k_BT)-1},
\]
\[
\gamma_{\phi}(T) = \frac{\gamma_{\phi0}}{2}\bigl(2n_{\text{th}}+1\bigr),\quad
\gamma_{\downarrow}(T) = \gamma_{\text{rel}0}(n_{\text{th}}+1),\quad
\gamma_{\uparrow}(T) = \gamma_{\text{rel}0}\, n_{\text{th}}.
\]

The Lindblad master equation:

\[
\frac{d\rho}{dt} = -\frac{i}{\hbar}[H,\rho] + \sum_k \bigl(L_k\rho L_k^\dagger - \tfrac12\{L_k^\dagger L_k,\rho\}\bigr).
\]

---

## 3. Exact Solution via Liouvillian Superoperator

Because the master equation is linear in `ρ`, we vectorise it into a 16‑dimensional vector `vec(ρ)`. The Liouvillian superoperator `L` (16×16) is constructed such that:

\[
\frac{d}{dt}\,\text{vec}(\rho) = \mathcal{L}\,\text{vec}(\rho).
\]

The exact evolution is:

\[
\text{vec}(\rho(t)) = \exp\bigl(\mathcal{L}\,t\bigr)\,\text{vec}(\rho(0)).
\]

We use `scipy.linalg.expm` to compute the matrix exponential once for each set of parameters. This is **exact**, **numerically stable**, and **avoids ODE integration errors**.

---

## 4. Key Observables

| Metric | Definition | Range |
|--------|------------|-------|
| Visibility V | \(2|\rho_{00,11}|\) | 0 → 1 |
| Distinguishability D | \(|\operatorname{Tr}(\rho_A\sigma_z)|\) | 0 → 1 |
| Concurrence C | standard two‑qubit formula | 0 → 1 |
| Negativity N | sum of negative eigenvalues of partial transpose | 0 → 1 |
| Purity Pur | \(\operatorname{Tr}(\rho^2)\) | 1/4 → 1 |
| Entropy S | von Neumann \(-\operatorname{Tr}(\rho\log\rho)\) | 0 → ln4 |
| Fidelity F_Bell | overlap with Bell state \(|\Phi^+\rangle\) | 0 → 1 |
| l1‑coherence | sum of off‑diagonal magnitudes / 6 | 0 → 1 |

Bohr’s complementarity: **V² + D² ≤ 1** is automatically satisfied.

---

## 5. Benchmarks

Three independent tests confirm numerical correctness:

1. **Pure dephasing** (initial Bell state): \( \rho_{00,11}(t) = 0.5\,e^{-\gamma_{\phi0}t} \).  
   Error < 1e‑12.

2. **Relaxation at T=0** (initial |01⟩): \( \rho_{11,11}(t) = e^{-\gamma_{\text{rel}0}t} \).  
   Error < 1e‑12.

3. **Entropy increase** (initial pure |00⟩, T=100 K): entropy rises; purity falls.  
   Success.

---

## 6. Inverse Calibration (Key Innovation)

Given a **target visibility** (e.g., measured in a laboratory), the model can recover hidden parameters:

- **`Inv γφ₀`** : finds the pure dephasing rate needed to achieve the target visibility at given θ, T.
- **`Inv T`** : finds the temperature consistent with the target visibility.
- **`Inv θ`** : finds the launch angle from visibility.

Additionally, a **full calibration** (`calibrate_from_data`) uses multiple (θ, T, V) points to fit γφ₀, γrel₀, ω simultaneously – enabling direct comparison with experimental datasets.

---

## 7. Unique Testable Prediction

The model predicts that **at θ = 90°, visibility equals distinguishability for any temperature**.  
This is a direct consequence of the entangled state `cos(45°)|00⟩+sin(45°)|11⟩` and the decoherence structure. It can be tested in photonic entanglement experiments and distinguishes DTQEM from alternative phenomenological models.

---

## 8. Interactive GUI

Built with `ipywidgets`, the interface is **fully responsive** on desktops, tablets, and smartphones. It includes:

- Real‑time sliders for all physical parameters.
- Live 1D/2D double‑slit fringes.
- Bloch vector of the reduced qubit.
- Real part of the density matrix (coloured matrix plot).
- Thermal response curves (V, C, N, P vs T).
- Inverse calibration buttons.
- One‑click saving of figures and CSV data.

---

## 9. Conclusion

DTQEM v12.1 is a **self‑contained, open‑source, exact simulator** of two‑qubit entanglement in realistic environments. It combines:

- **Physics**: Lindblad master equation, Bose–Einstein statistics, magnetic coupling.
- **Numerical accuracy**: machine‑precision via superoperator exponentiation.
- **Usability**: interactive ipywidgets GUI for desktop and mobile.
- **Data‑driven features**: inverse calibration from experimental visibility.
- **Testable science**: unique prediction V = D at θ=90°.

The code is ready for use in education, research, and as a foundation for extensions (non‑Markovian noise, gravitational effects, multi‑qubits).

---

## 10. References

1. Lindblad, G. (1976). On the generators of quantum dynamical semigroups. *Commun. Math. Phys.* **48**, 119.
2. Breuer, H.‑P. & Petruccione, F. (2002). *The Theory of Open Quantum Systems*.
3. Gisin, N. & Zbinden, H. (1998). Lower bound for the speed of quantum non‑locality. *Phys. Lett. A* **248**, 1.
4. Aspect, A., Dalibard, J. & Roger, G. (1982). Experimental test of Bell’s inequalities. *Phys. Rev. Lett.* **49**, 1804.

---

**End of White Paper**
