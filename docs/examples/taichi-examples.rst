.. meta::
  :description: Taichi Lang examples
  :keywords: Taichi Lang, JIT, parallel, programming, python, ROCm, example, sample, tutorial

.. _run-a-taichi-example:

********************************************************************
Run a Taichi Lang example
********************************************************************

Several examples are available in the `https://github.com/AMD-Ecosystem/taichi_examples <https://github.com/AMD-Ecosystem/taichi_examples>`__
repository, which includes a Dockerfile for building a container to install Taichi and a ``README.md`` file
with instructions for building and running the examples.

1. Clone the repository: 

.. code-block:: bash

   git clone https://github.com/AMD-Ecosystem/taichi_examples.git

2. If you have Taichi installed in your environment, you do not need to build an additional container. Simply install the example dependencies:

.. code-block:: bash

   pip3 install pillow
   pip3 install --no-cache-dir torch --index-url https://download.pytorch.org/whl/nightly/rocm6.3/

4. Run the examples scripts from the ``taichi_examples`` directory:

.. code-block:: bash

   cd taichi_examples
   ./run_demos.sh
   ./run_algorithm_graph_examples.sh

5. The output should show as follows for ``run_demos.sh``:

.. code-block:: bash

   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   2710
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   78498
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   Elapsed 0.07401013374328613 seconds
   Elapsed 0.07366204261779785 seconds
   Elapsed 0.07427334785461426 seconds
   Elapsed 0.07503175735473633 seconds
   Elapsed 0.0747842788696289 seconds
   Elapsed 0.0749671459197998 seconds
   Elapsed 0.07503128051757812 seconds
   Elapsed 0.07497262954711914 seconds
   =========================================================================
   Kernel Profiler(count, default) @ AMDGPU on AMD Instinct MI210
   =========================================================================
   [      %     total   count |      min       avg       max   ] Kernel name
   -------------------------------------------------------------------------
   [100.00%   0.002 s      8x |    0.256     0.262     0.271 ms] ti_pad_c80_0_kernel_0_range_for
   -------------------------------------------------------------------------
   [100.00%] Total execution time:   0.002 s   number of results: 1
   =========================================================================

6. The output should show as follows for ``run_algorithm_graph_examples.sh``:

.. code-block:: bash

   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   running in graph mode
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   0.0
   4.0
   0.0
   0.0
   4.0
   0.0
   0.0
   4.0
   0.0
   0.0
   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu


Prepared Taichi Lang examples using decorators
================================================================================

The following examples show you how to use decorators, and is organized by use case.

Count primes
--------------------------------------------------------------------------------

In this example, the function ``is_prime`` will be used in the kernel ``count_primes``. 
The code below is written as a Taichi program by decorating ``is_prime``
with the Taichi Lang decorator ``@ti.func`` and decorating ``count_primes`` with the Taichi
Lang decorator ``@ti.kernel``.

1. To run this example, copy the code below to a file named ``count_primes.py``:

.. code-block:: bash

   import taichi as ti
   ti.init(arch=ti.gpu)

   @ti.func
   def is_prime(n: int):
      result = True
      for k in range(2, int(n ** 0.5) + 1):
         if n % k == 0:
               result = False
               break
      return result

   @ti.kernel
   def count_primes(n: int) -> int:
      count = 0
      for k in range(2, n):
         if is_prime(k):
               count += 1

      return count

   print(count_primes(1000000))

2. Once this file has been created, execute the code in your Docker container with the following command:

.. code-block:: bash

   python3 count_primes.py

3. The output should be similar to the output below:

.. code-block:: bash

   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   78498

Longest common subsequence 
--------------------------------------------------------------------------------

This is an example of longest common subsequence kernel. You don't
need a helper function, so the only decorator needed is ``@ti.kernel`` to 
accelerate the kernel function ``compute_lcs``.

1. To run this example, copy the code below into a file named ``lcs.py``:

.. code-block:: bash

   import taichi as ti
   import numpy as np

   ti.init(arch=ti.gpu)

   benchmark = True

   N = 15000

   f = ti.field(dtype=ti.i32, shape=(N + 1, N + 1))

   if benchmark:
      a_numpy = np.random.randint(0, 100, N, dtype=np.int32)
      b_numpy = np.random.randint(0, 100, N, dtype=np.int32)
   else:
      a_numpy = np.array([0, 1, 0, 2, 4, 3, 1, 2, 1], dtype=np.int32)
      b_numpy = np.array([4, 0, 1, 4, 5, 3, 1, 2], dtype=np.int32)

   @ti.kernel
   def compute_lcs(a: ti.types.ndarray(), b: ti.types.ndarray()) -> ti.i32:
      len_a, len_b = a.shape[0], b.shape[0]

      ti.loop_config(serialize=True) # Disable auto-parallelism in Taichi
      for i in range(1, len_a + 1):
         for j in range(1, len_b + 1):
                  f[i, j] = ti.max(f[i - 1, j - 1] + (a[i - 1] == b[j - 1]),
                           ti.max(f[i - 1, j], f[i, j - 1]))

      return f[len_a, len_b]


   print(compute_lcs(a_numpy, b_numpy))

2. Once this file has been created, execute the code in your Docker container with the following command:

.. code-block:: bash

   python3 lcs.py

3. The output should be similar to the output below:

.. code-block:: bash

   [Taichi] version 1.8.0b1, llvm 15.0.0, commit eeb3354a, linux, python 3.10.12
   [Taichi] Starting on arch=amdgpu
   2706

Use cases and recommendations
================================================================================

* The `Accelerating Parallel Programming in Python with Taichi Lang on AMD GPUs 
  <https://rocm.blogs.amd.com/artificial-intelligence/taichi/README.html>`__
  blog highlights Taichi Lang as an open-source programming language designed for high-performance 
  numerical computation, particularly in domains such as real-time physical simulation, 
  artificial intelligence, computer vision, robotics, and visual effects. Taichi Lang
  is embedded in Python and uses just-in-time (JIT) compilation frameworks like 
  LLVM to optimize execution on GPUs and CPUs. The blog emphasizes the versatility 
  of Taichi Lang in enabling complex simulations and numerical algorithms, making 
  it ideal for developers working on compute-intensive tasks. Developers are 
  encouraged to follow recommended coding patterns and utilize Taichi Lang decorators 
  for performance optimization, with examples available in the `https://github.com/AMD-Ecosystem/taichi_examples 
  <https://github.com/AMD-Ecosystem/taichi_examples>`__ repository. Prebuilt Docker images 
  integrating ROCm, PyTorch, and Taichi are provided for simplified installation 
  and deployment, making it easier to leverage Taichi Lang for advanced computational workloads.

Refer to the `AMD ROCm blog <https://rocm.blogs.amd.com/>`__ to search for Taichi examples and best practices to optimize your workflows on AMD GPUs.
