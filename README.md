📌 Hybrid MPI + OpenMP Adaptive Thresholding

This project implements a hybrid parallel version of adaptive image thresholding using MPI for distributed-memory parallelism and OpenMP for shared-memory multithreading.
The goal is to accelerate adaptive thresholding while maintaining correct segmentation accuracy across large input images.

🚀 Overview

Adaptive thresholding requires computing a local threshold for every pixel based on a surrounding window.
A naïve implementation is computationally expensive for large images.

This project uses:

Integral Images for O(1) window-sum computation

MPI to distribute rows of the image across multiple cluster nodes

OpenMP to parallelize per-pixel work within each node

Hybrid MPI+OpenMP execution for multi-node, multi-core acceleration



🖼️ Algorithm Summary
Adaptive Thresholding Formula

For each pixel p:

mean = sum(window) / area
thresh = mean - C
p_out = 255 if p > thresh else 0


Where:

window_size is user-defined (e.g., 15×15)

C is a constant adjustment (typically 3–10)

sum(window) uses an integral image for O(1) lookup

💻 Hybrid Parallel Execution
MPI (Inter-node parallelism)

Image rows are divided among MPI ranks.

Each rank receives the full input image + integral image.

Each rank computes thresholding on its row segment.

Results are collected via MPI_Gatherv on rank 0.

OpenMP (Intra-node parallelism)

Inside each MPI rank:

#pragma omp parallel for schedule(static)
for (int r = start_row; r < end_row; r++) {
    // Local pixel thresholding
}


Total concurrency = MPI ranks × OpenMP threads per rank.

🧪 Performance Experiments

Experiments were run on the Picocluster (ARM Cortex-A72 nodes, 4 cores/node).
Performance was compared across:

Mode	Description
Serial	No parallelism
OpenMP	Single MPI rank, 4 threads
MPI	4 MPI ranks, 1 thread each
Hybrid	4 MPI ranks × 4 threads = 16 cores

Large 4K grayscale images were used to test scaling.

📈 Example Timing Summary
Mode	MPI	OMP	Total Cores	Time (s)
serial_1core	1	1	1	0.4598
omp_1x4	1	4	4	0.1318
mpi_4x1	4	1	4	1.0099
hybrid_4x4	4	4	16	0.9174
Key Observations

OpenMP delivered the best speedup due to shared-memory efficiency.

MPI and hybrid runs were slower for moderate image sizes because of:

large MPI broadcast/gather overhead (~60 MB transfers)

low-power ARM cores

communication dominating computation time

▶️ How to Run
Convert an input image to PGM
python3 convert_to_pgm.py input.jpg input.pgm

Compile hybrid program
mpic++ -Ofast -fopenmp \
    main_hybrid.cpp threshold_hybrid.cpp threshold_common.cpp image_io.cpp \
    -o adaptive_hybrid

Run with MPI
export OMP_NUM_THREADS=4
mpirun -np 4 ./adaptive_hybrid input.pgm output.pgm 15 5 timing.csv

🏷️ PBS Script for Picocluster

Submit using the helper:

run_script_hyb pbs_Script.sh


The script handles:

input conversion

running serial, OpenMP, MPI, and Hybrid tests

collecting performance logs

📜 Requirements

MPI implementation (OpenMPI or MPICH)

OpenMP-capable compiler (GCC)

Python 3 + Pillow (for image conversion)

PGM image input format

📚 References

Viola, P., & Jones, M. (2001). Rapid object detection using boosted cascades of simple features.

Integral image technique

MPI + OpenMP hybrid programming guides

👩‍💻 Author

Sanjoy Dev
University of Alabama in Huntsville
