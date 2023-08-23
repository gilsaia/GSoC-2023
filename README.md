# Optimizing MLIR’s Presburger library
This document is intended to serve as a summary of the work that I did during my participation in GSOC 2023.
## Introduction
The MLIR Presburger Library is a tool for polyhedral compilation and analysis that provides mathematical abstractions for sets of integer tuples defined by a system of affine inequality constraints. These sets can be manipulated using standard set operations, and the resulting sets may have additional constraints. However, when performing many set operations in sequence, the size of the constraint system may grow significantly, leading to reduced performance. To address this issue, there are several potential ways to simplify the constraint system, but this involves performing additional computations. The challenge is to find a balance between simplifying the constraint system enough to improve performance, but not so much that the additional computational cost of the simplification outweighs the performance gain. This project aims to find the optimal balance by developing and implementing simplification heuristics that strike the right balance between computational cost and performance improvement.
## Goals
- Understand Polyhedral Compilation.
- Understand the basic usage of ISL and FPL.
- Getting to know the contribution workflow for fpl.
- Implement a test Benchmark to measure the performance gap between different operations on ISL vs. FPL.
- Use the data from the Benchmark test to guide FPL optimization.
## Implementation
### Benchmark
The implementation of Benchmark consists of the following main parts:
- Test Data
- Test Code
- Analysis Code

For the test data, Benchmark uses the [data](https://github.com/Superty/presburger-benchmarks) here, and the parse process involves modifying the Parser in MLIR and saving it in the form of a matrix for subsequent Benchmarks.

Afterwards, Google Benchmark was used to build a MLIRPresburgerBenchmark, which contains the basic construction process, builds isl_map/PresburgerRelation, and conducts tests for different ops, counts their runtime, input size, and result size, and saves the corresponding detailed results for subsequent analysis.

For the obtained test results, they are visualized as images by a simple python script, which makes it easy to visually compare the improvement of FPL performance with different optimizations.

All of the above code and data is included [here](https://github.com/gilsaia/llvm-project-test-fpl/pull/1) (the PR is opened just to make it easier to see the changes), please note that this part of the code will not be very well commented as it will not be merged into upstream.
### Optimize
- Optimization of Intersect operations
  - Check for apparently empty or apparently universe.
  - https://reviews.llvm.org/D154771
- Optimization of Union & Subtract
  - Add a quick set equality check that applies to union and subtract.
  - https://reviews.llvm.org/D156241
- Add simplify function
  -  Add standard Gaussian elimination operations, as well as the ability to try to eliminate inequalities.
  -  https://reviews.llvm.org/D156885

Some of the results of the test are as follows:

For the case of calling Simplify before each operation, the overall result is shown in the following figure.
![simplify_with_time](https://github.com/gilsaia/GSoC-2023/assets/38588948/11cf164b-1726-4516-8231-9030a7d9a145)

For the case of calling Simplify on the result after Subtract, the overall results are as follows.
![simplify_post_subtract](https://github.com/gilsaia/GSoC-2023/assets/38588948/e0db8456-d03d-434a-bad6-5839cadc90f8)

Some of the more obvious detailed size comparisons are as follows.
![Empty_size](https://github.com/gilsaia/GSoC-2023/assets/38588948/be6d1639-cdde-4629-bfff-d6349617e4b9)
![Union_size](https://github.com/gilsaia/GSoC-2023/assets/38588948/fa2d7505-d84b-47a0-9d02-8fb40180d47f)
![Complement_size](https://github.com/gilsaia/GSoC-2023/assets/38588948/3315ea93-8ed1-421a-86ac-af46e89431fc)

## Future Work
Future work in the near future is to further refine the simplify function above and merge it into upstream, followed by the optimization of the findSymbolicIntegerLexMin function mentioned by mentor.

In the long run, I've got a good understanding of MLIR and FPL related functions, and I'll try to continue to contribute to FPL and try to optimize more functions.
## Acknowledgement
