﻿ # `Vectorize VecMatMult` Sample

The `Vectorize VecMatMult` demonstrates how to use the auto-vectorizer to improve the performance
of the sample application. You will compare the performance of a serial version and a version compiled with the auto-vectorizer.

| Area                     | Description
|:---                      |:---
| What you will learn      | How to use automatic vectorization with the Intel® Fortran Compiler
| Time to complete         | 15 minutes

## Purpose

The Intel® Fortran Compiler has an auto-vectorizer that detects operations in the application that can be done in parallel and converts sequential operations to parallel operations by using the Single Instruction Multiple Data (SIMD) instruction set.

For the Intel® Fortran Compiler, vectorization is the unrolling of a loop combined with the generation of packed SIMD instructions. Because the packed instructions operate on more than one data element simultaneously, the loop can execute more efficiently. It is sometimes referred to as auto-vectorization to emphasize that the compiler automatically identifies and optimizes suitable loops on its own.

Vectorization may call library routines that can result in additional performance gain on Intel microprocessors when compared to non-Intel microprocessors. The vectorization can also be affected by specific options, such as `-m` or `-x`.

Vectorization is enabled with the compiler at optimization levels of `-O2` (default level) and higher for both Intel® microprocessors and non-Intel® microprocessors. Many loops are vectorized automatically, in cases where this doesn't happen, you may be able to vectorize loops by making simple code modifications. 

This sample leads you through the following steps.
1. Establish a performance baseline.
2. Generate a vectorization report.
3. Improve performance by aligning data.
4. Improve performance with interprocedural optimization.

Intel® Advisor can assist with vectorization and show optimization report messages with your source code. See [Intel® Advisor](https://software.intel.com/content/www/us/en/develop/tools/advisor.html) for more information.

## Prerequisites

| Optimized for                     | Description
|:---                               |:---
| OS                                | Linux*
| Software                          | Intel® Fortran Compiler

>**Note**: The Intel® Fortran Compiler is part of the Intel® oneAPI HPC Toolkit (HPC Kit).

## Key Implementation Details
You will use the following Fortran source files in the sample.

| File              | Description
|:---               |:---
|`matvec.f90`       | Fortran source file with a matrix-times-vector algorithm.
|`driver.f90`       | Fortran source file with the main program calling matvec.

Read the [Intel® Fortran Compiler Developer Guide and Reference](https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-developer-guide-and-reference/top.html) for more information the features and options mentioned in this sample. 

## Set Environment Variables

When working with the command-line interface (CLI), you should configure the oneAPI toolkits using environment variables. Set up your CLI environment by sourcing the `setvars` script every time you open a new terminal window. This practice ensures that your compiler, libraries, and tools are ready for development.

## Build the `Fortran Vectorization` Sample

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script in the root of your oneAPI installation.
>
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: ` . ~/intel/oneapi/setvars.sh`
>
> For more information on configuring environment variables, see [Use the setvars Script with Linux*](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html).

### Use Visual Studio Code* (VS Code) (Optional)

You can use Visual Studio Code* (VS Code) extensions to set your environment,
create launch configurations, and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 1. Configure the oneAPI environment with the extension **Environment Configurator for Intel® oneAPI Toolkits**.
 2. Download a sample using the extension **Code Sample Browser for Intel® oneAPI Toolkits**.
 3. Open a terminal in VS Code (**Terminal > New Terminal**).
 4. Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see the 
[Using Visual Studio Code with Intel® oneAPI Toolkits User Guide](https://www.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

### On Linux*

#### Step 1. Establish a Performance Baseline

Create a performance baseline for the improvements that follow in this sample by compiling your sources from the `src` directory.
1. Compile the sources with the following commands.
   ```
   ifort -real-size 64 -O1 src/matvec.f90 src/driver.f90 -o MatVector
   ```
2. Run `MatVector`.
   ```
   ./MatVector
   ```
3. Record the execution time reported in the output. This is the baseline against which subsequent improvements will be measured.

#### Step 2. Generate a Vectorization Report

A vectorization report shows what loops in your code were vectorized and explains why other loops were not vectorized. To generate a vectorization report, use the **qopt-report-phase=vec** compiler options together with **qopt-report=1** or **qopt-report=2**.

Together with **qopt-report-phase=vec**, **qopt-report=1** generates a report with the loops in your code that were vectorized while **qopt-report-phase=vec** with **qopt-report=2** generates a report with both the loops in your code that were vectorized and the reason that other loops were not vectorized.

Because vectorization is turned off with the **O1** option, the compiler does not generate a vectorization report. Generate a vectorization report by compiling the project with the **O2**, **qopt-report-phase=vec**, **qopt-report=1** options.

1. Compile the sources with the following commands.
   ```
   ifort -real-size 64 -O2 -qopt-report=1 -qopt-report-phase=vec src/matvec.f90 src/driver.f90 -o MatVector
   ```
2. Run `MatVector` again.
   ```
   ./MatVector
   ```
3. Record the new execution time. 

   The reduction in time is mostly due to auto-vectorization of the inner loop at line 32 noted in the vectorization report **matvec.optrpt**.

   > **Note**: Your line and column numbers may be different.

   ```
    Begin optimization report for matvec_

      Report from: Vector optimizations [vec]


    LOOP BEGIN at matvec.f90(26,3)
      remark #25460: No loop optimizations reported

      LOOP BEGIN at matvec.f90(26,3)
        remark #15300: LOOP WAS VECTORIZED
      LOOP END

      LOOP BEGIN at matvec.f90(26,3)
      <Remainder loop for vectorization>
      LOOP END
    LOOP END

    LOOP BEGIN at matvec.f90(27,3)
      remark #25460: No loop optimizations reported

      LOOP BEGIN at matvec.f90(32,6)
      <Peeled loop for vectorization>
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
        remark #15300: LOOP WAS VECTORIZED
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
      <Alternate Alignment Vectorized Loop>
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
      <Remainder loop for vectorization>
      LOOP END
    LOOP END
   ```

   The combination of **qopt-report=2** with **qopt-report-phase=vec, loop** returns a list that includes loops that were not vectorized or multi-versioned, along with the reason that the compiler did not vectorize them or multi-version the loop.

4. Recompile your project with the **qopt-report=2** and **qopt-report-phase=vec,loop** options.
   ```
   ifort -real-size 64 -O2 -qopt-report-phase=vec -qopt-report=2 src/matvec.f90 src/driver.f90 -o MatVector
   ```

   The vectorization report **matvec.optrpt** indicates that the loop at line 33 in matvec.f90 did not vectorize because it is not the loop nest's innermost loop.

   > **Note**: Your line and column numbers may be different.

   ```
    LOOP BEGIN at matvec.f90(27,3)
      remark #15542: loop was not vectorized: inner loop was already vectorized

      LOOP BEGIN at matvec.f90(32,6)
       <Peeled loop for vectorization>
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
        remark #15300: LOOP WAS VECTORIZED
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
       <Alternate Alignment Vectorized Loop>
      LOOP END

      LOOP BEGIN at matvec.f90(32,6)
       <Remainder loop for vectorization>
         remark #15335: remainder loop was not vectorized: vectorization possible but seemed inefficient. Use vector always directive or -vec-threshold0 to override
      LOOP END
    LOOP END
   ```
   For more information on the **qopt-report** and **qopt-report-phase** compiler options, read the *Compiler Options* section of the [Intel® Fortran Compiler Developer Guide and Reference](https://software.intel.com/content/www/us/en/develop/documentation/fortran-compiler-developer-guide-and-reference/top.html).

#### Step 3. Improve Performance by Aligning Data

The vectorizer can generate faster code when operating on aligned data. In this activity, you will improve the vectorizer performance by aligning the arrays a, b, and c in **driver.f90** on a 16-byte boundary so the vectorizer can use aligned load instructions for all arrays rather than the slower unaligned load instructions and can avoid runtime tests of alignment.

Using the ALIGNED macro will insert an alignment directive for a, b, and c in the driver.f90 with the following syntax:

```fortran
!dir$ attributes align : 16 :: a,b,c
```
This code sample instructs the compiler to create arrays aligned on a 16-byte boundary, facilitating the use of SSE aligned load instructions.

The column height of the matrix needs to be padded out to be a multiple of 16 bytes, so that each column maintains the same 16-byte alignment. In practice, maintaining a constant alignment between columns is much more important than aligning the arrays' start.

To derive the maximum benefit from this alignment, we also need to tell the vectorizer it can safely assume that the arrays in `matvec.f90` are aligned by using the directive

```fortran
!dir$ vector aligned
```

>**Note**: If you use **!dir$ vector aligned**, you must be sure that all the arrays or subarrays in the loop are 16-byte aligned. Otherwise, you may get a runtime error. Aligning data may still give a performance benefit even if `!dir$ vector aligned` is not used. See the code under the ALIGNED macro in `matvec.f90`.

If your compilation targets the Intel® AVX-512 instruction set, you should try to align data on a 64-byte boundary. This may result in improved performance. In this case, `!dir$ vector aligned` advises the compiler that the data is 64-byte aligned.

1. Recompile the program after adding the ALIGNED macro to ensure consistently aligned data.
   ```
   ifort -real-size 64 -qopt-report=2 -qopt-report-phase=vec -D ALIGNED src/matvec.f90 src/driver.f90 -o MatVector
   ```

#### Step 4. Improve Performance with Interprocedural Optimization

The compiler may be able to perform additional optimizations if it can optimize across source line boundaries. These may include but are not limited to function inlining. Enable this optimization with the `-ipo` option.

1. Recompile the program using the `-ipo` option to enable interprocedural optimization.
   ```
   ifort -real-size 64 -qopt-report=2 -qopt-report-phase=vec -D ALIGNED -ipo src/matvec.f90 src/driver.f90 -o MatVector
   ```
   The vectorization messages now appear at the point of inlining in **driver.f90** (line 70) and this is found in the file **ipo_out.optrpt**.

   > **Note**: Your line and column numbers may be different.

   ```
    LOOP BEGIN at the driver.f90(73,16)
       remark #15541: loop was not vectorized: inner loop was already vectorized

       LOOP BEGIN at matvec.f90(32,3) inlined into the driver.f90(70,14)
          remark #15398: loop was not vectorized: loop was transformed to memset or memcpy
       LOOP END

       LOOP BEGIN at matvec.f90(33,3) inlined into driver.f90(70,14)
          remark #15541: loop was not vectorized: inner loop was already vectorized

          LOOP BEGIN at matvec.f90(38,6) inlined into driver.f90(70,14)
             remark #15399: vectorization support: unroll factor set to 4
             remark #15300: LOOP WAS VECTORIZED
          LOOP END
       LOOP END
    LOOP END
   ```
2. Run the program, and record the execution time.

### Additional Exercises

The previous examples made use of double-precision arrays. You could build same examples with single precision arrays by changing the command-line option **-real-size 64** to **-real-size 32**. The non-vectorized versions of the loop execute only slightly faster than the double-precision version; however, the vectorized versions are substantially faster. This is because a packed SIMD instruction operating on a 32-byte vector register operates on eight single-precision data elements at once instead of four double-precision data elements.

> **Note**: In the example with data alignment, you will need to set ROWBUF=3 to ensure 16-byte alignment for each row of the matrix a. Otherwise, the directive **!dir$ vector aligned** will cause the program to fail.

## License

Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).