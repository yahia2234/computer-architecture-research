# BranchNet: A Convolutional Neural Network to Predict Hard-To-Predict Branches (MICRO 2020)

**Topic:** Machine Learning in Computer Architecture / Branch Prediction  
**Analysis By:** Yahia Rady  
**Context:** CSEN 702 Microprocessors Research Review

## 📄 Resources
- [📄 Read the Full Academic Summary (PDF)](./BranchNet_Summary.pdf)
- [📊 View the Research Presentation (PDF)](./BranchNet_Presentation.pdf)

---

## 🔍 Executive Summary
Modern processors rely on Branch Predictors (like TAGE) to maintain high pipeline throughput. However, complex workloads (e.g., SPEC2017) introduce "noise" into the global history, causing TAGE to fail on **"Hard-to-Predict" (H2P)** branches.

This research analyzes **BranchNet**, a Convolutional Neural Network (CNN) designed to solve this by exploiting **Shift Invariance**. Unlike TAGE, which relies on exact bit-matching, BranchNet can identify semantic patterns in execution history regardless of where they appear or how much noise surrounds them.

## 🛠️ Key Architectural Insights

### 1. The Problem: Noise in Global History
Traditional predictors index tables using specific history bits. In complex integer workloads, thousands of irrelevant branches ("noise") can shift the relevant correlating branch in the history buffer. This changes the index, causing a prediction miss.

### 2. The Solution: CNNs & Sum-Pooling
The paper introduces a hardware-software co-design:
*   **1D Convolution:** Detects specific sequences of Taken/Not-Taken branches (features).
*   **Sum-Pooling (Key Innovation):** Aggregates feature activations, effectively counting *how many times* a pattern occurred while discarding *positional* information. This makes the predictor robust to history noise.

### 3. Implementation: Mini-BranchNet
A standard CNN is too slow for a CPU fetch stage (1 cycle). The authors propose **Mini-BranchNet**:
*   4-cycle latency (pipelined).
*   Quantized to 8-bit integers.
*   Profile-Guided Optimization (PGO) to train the model offline.

---

## 💡 My Critique & Proposed Enhancement
*This section details my independent analysis of the paper's limitations and my proposed architectural improvement.*

### The Limitation: Static Profiling Risks
BranchNet relies on **offline training** based on profiling data. It assumes that "hard" branches are static properties of the code. However, if runtime input varies significantly from the training profile (**Dataset Shift**), the offline model may perform worse than a standard dynamic predictor.

### 🚀 Proposal: Dynamic Confidence-Based Hybrid Selector
Instead of blindly trusting the offline CNN for H2P branches, I propose a hardware safety net:

1.  **Confidence Score:** Extend the CNN hardware to output a confidence score derived from the final sigmoid activation.
2.  **Runtime Monitor:** A lightweight hardware structure monitors BranchNet's dynamic accuracy.
3.  **Fallback Mechanism:** If BranchNet's confidence is low or its recent accuracy drops, the CPU dynamically falls back to the standard TAGE-SC-L predictor.

**Impact:** This ensures the processor benefits from Deep Learning when the execution matches the profile, but remains robust during unseen phases.

---

## 📊 Results (from Paper)
*   **MPKI Reduction:** 9.6% reduction in Mispredictions Per Kilo Instructions over a 64KB TAGE-SC-L.
*   **IPC Gain:** Geometric mean improvement of 1.3% (up to 7.9% on specific workloads).
*   **Storage Efficiency:** Higher accuracy per bit of storage compared to simply doubling TAGE size.
