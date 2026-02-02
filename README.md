# Multi-Threaded Newton Fractal Generator
A C11 implementation of the Newton-Raphson method for fractal generation.

This program generates two visualizations of Newton’s method on the complex plane:
1.  **Attractors:** Colored by which root the point converges to.
2.  **Convergence:** Shaded by how many iterations it took to get there.

The project uses a concurrency model for task management. It uses a producer-consumer architecture to ensure that the CPU stays busy computing while the disk stays busy writing, without either bottlenecking the other.



## How it Works (Under the Hood)

### 1. Manual Thread Management (`threads.h`)
Instead of using a simple parallel loop, I implemented a **task-queue system** using C11 threads. A benefit of this is that it doesn't break parallelization or perform unnecessary computation when different initial values require a different number of iteration of the newton method.  
* **Worker Threads:** A user-defined number of threads fetch "blocks" of the complex plane to compute.
* **The Writer Thread:** A dedicated thread handles all disk I/O. It waits for workers to mark a block as `READY` and writes them to the file in the correct order.
* **Synchronization:** Uses `mtx_t` (mutexes) and `cnd_t` (condition variables) to manage access to a shared pool of `BlockBuffers`.

### 2. I/O Optimizations
Writing a massive ASCII-based PPM file is notoriously slow because `fprintf` is expensive. To solve this, I used two specific optimizations:
* **String Pre-computation:** All possible RGB color strings are pre-calculated. And formatted as identical length strings. 
* **Buffer Direct-Copy:** Instead of formatting strings on the fly, I use `memcpy` to move the precomputed color strings into a large memory buffer, then use a single `fwrite` call to dump the entire block to disk. 

### 3. Fixed-Precision Representation
To handle large image sizes without blowing out the memory, the program processes data in discrete blocks. Each point's result is stored in a compact custom type before being handed off to the writer.

