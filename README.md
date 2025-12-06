# Hybrid MPI + OpenMP Adaptive Thresholding

**Author:** MD. Shihab Uddin and Sanjoy Dev  
**Institution:** University of Alabama in Huntsville  
**Platform:** Picocluster (ARM Cortex-A72)

## 📌 Project Overview
This project implements a **hybrid parallel** version of adaptive image thresholding. It utilizes **MPI** (Message Passing Interface) for distributed-memory parallelism across cluster nodes and **OpenMP** for shared-memory multithreading within each node.

The goal is to accelerate adaptive thresholding while maintaining correct segmentation accuracy across large input images.

**Key Features:**
* **Integral Images:** Uses the integral image technique for $O(1)$ window-sum computation.
* **MPI:** Distributes rows of the image across multiple cluster nodes.
* **OpenMP:** Parallelizes per-pixel work within each node using threads.
* **Hybrid Execution:** Combines both models for multi-node, multi-core acceleration.

---

## 🖼️ Algorithm Summary

Adaptive thresholding computes a local threshold for every pixel based on a surrounding window. A naïve implementation is computationally expensive for large images, so this project optimizes the calculation.

### Formula
For each pixel $p$:

```math
mean = \frac{sum(window)}{area}
