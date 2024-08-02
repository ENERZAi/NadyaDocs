# Command Line Options

## General Options

- `--version`

  Print version of `ndc`.

- `--help, -h`

  Print command line options.

- `--input, -i <file>`

  Input file (for single file compilation).

- `--output, -o <file>`

  Output file or dir.

- `--target, -t <file>`

  target file (for target compilation).

- `--workspace, -w <file>`

  workspace directory.

- `--function <function name>`

  specify function name for target compilation

- `--template-arguments, -a <values>`

  Template arguments for `--function`

- `--lowering-level=<level>`

  Lowering level ( `compiletime`, `runtime`, `nadya-dialect`, `llvm-dialect`, `llvmir`, `object`, `final` -> default)
  This option
  is only for `--lowering-level=final`

- `--ll=<uint>`

  Alias for lowering
  level. (`0=compiletime`, `1=runtime`, `2=nadya-dialect`, `3=llvm-dialect`, `4=llvmir`, `5=object`, `6=final`)

- `--rtlib=<copiler runtime library>`

  Choose compiler runtime library. (`libgcc` : gcc runtime library, `compiler-rt` : clang runtime library)

- `--object`

  Object type (`shared` : shared library, `static` : static library, `executable`(default) : executable file). This
  options
  needs `--lowering-level=final` or `--ll=6`, if not, this option will be ignored.

- `-shared` `-static`

  Alias for `--object=shared` and `--object=static`


- `--print-log=<value>`

  Print internal task and execution of `ndc`. If `value` were `stderr/stdout`, print log to stderr/stdout, or not,
  print to `<value>` file. If the `<value>` were not appropriate, `ndc` treat this value to `stdout`

- `--verbose, -v`

  Alias for `--print-log=stderr`.

- `-l<library>`

  External libraries for link.

- `-L<dir>`

  Add directories which libraries are exists.

- `-g, --O1, --O2, --O3`

  Optimization level

- `-fuse-ld=<linker>`

  Choose linker (lld, bfd, ...)

- `-fopenmp`, `-fno-openmp`

  Enable parallel compilation and link with `libomp`. This option requires `libomp.so` in library paths. You can add
  paths by using option `-L<dir>`.

- `-fopt-vectorize-size=<uint>`

  Make `nadya-opt` vectorize to size `<uint>`. This priority is higher than `--cpu` or x86 instruction options
  like `-mavx512f`, `-mavx`, etc... .

- `fopt-stack-threshold=<uint>`

  Make `nadya-opt` change stack threshold size to `<uint>`

- `fopt-full-pass="<pass1> <pass2> ..."`

  Ignore default `nadya-opt` pass and replace to given pass.

- `-fpic`, `-fPIC`

  Choose reallocation model

- `-fopci-path=<path to opci>`, `-fnadya-opt-path=<path to nadya-opt>`, `-fmlir-translate-path=<path to mlir-translate>`, `-fclang-path=<path to clang>`

  Choose tools explicitly

## Direct Options

- `WClang,<arg1>,<arg2>,...` : Pass options to clang internally. `ndc` doesn't check any problem in the arguments.
- `WLinker,<arg1>,<arg2>,...` : Pass options to linker internally. `ndc` doesn't check any problem in the arguments.

## Target option

### Options for X86

- `--triple=<value>` : target triple for cross compilation
- `--sysroot=<path>` : system root for cross compilation
- `-mcpu=<cpu>`

  Target cpu. This option will emit error when `--triple` architecture is not `x86` or be ignored when instruction sets
  are defined by user. This option will not be delivered to `clang`, `ndc` analyze it and use it to vectorize or
  choose instruction sets. However, the program which is compiled by `-mcpu` option can't be executed in other device.

    - available cpus
        - `alderlake`, `bdver1`, `bdver2`, `bdver3`, `bdver4`, `icelake_client`, `icelake_server`, `meteorlake`, `raptorlake`, `tigerlake`,
          `znver1`, `znver2`, `znver3`, `znver4`

- `-march=<value>`

  This option is passed to `clang`'s `march=<value>` option directly.

- Instruction sets for x86 cpu
    - 64bit vectorize
        - `-mmmx`, `-m3dnow`, `-m3dnowa`
    - 128bit vectorize
        - `-msse`, `-msse2`, `-msse3`, `-mssse3`, `-mfma`, `-mfma4`, `-mf16c`
    - 256bit vectorize
        - `-msse4.1`, `-msse4.2`, `-msse4a`, `-mavx`, `-mavx2`, `-mxop`, `-mavxifma`, `-mavxvnni`,  `-mavxvnniint8`, `-mavxneconvert`, `-mavxvnniint16`
    - 512bit vectorize
        - `-mavx512f`, `-mavx512pf`, `-mavx512er`, `-mavx512cd`, `-mavx512vl`, `-mavx512bw`, `-mavx512dq`, `-mavx512ifma`, `-mavx512vbmi`, `-mavx512vbmi2`, `-mavx512bitalg`, `-mavx512vpopcntdq`, `-mavx512vnni`, `-mavx512vp2intersect`, `-mavx512bf16`
    - other instructions
        - `-msha`, `-maes`, `-mpclmul`, `-mclflushopt`, `-mclwb`, `-mfsgsbase`, `-mptwrite`, `-mrdrnd`, `-mpconfig`, `-mwbnoinvd`, `-mprfchw`, `-mrdpid`, `-mprefetchwt1`, `-mrdseed`, `-msgx`, `-mlwp`, `-mpopcnt`, `-madx`, `-mbmi`, `-mbmi2`, `-mlzcnt`, `-mfxsr`, `-mxsave`, `-mxsaveopt`, `-mxsavec`, `-mxsaves`, `-mrtm`, `-mtbm`, `-mmwaitx`, `-mclzero`, `-mpku`, `-mgfni`, `-mvaes`, `-mwaitpkg`, `-mvpclmulqdq`, `-mmovdiri`, `-mmovdir64b`, `-mcldemote`, `-msahf`, `-mrdpru`, `-minvpcid`, `-mshstk`, `-mkl`, `-menqcmd`, `-muintr`, `-mserialize`, `-mtsxldtrk`, `-mamx-bf16`, `-mamx-tile`, `-mamx-int8`, `-mamx-fp16`, `-mamx-complex`, `-msha512`, `-msm3`, `-msm4`, `-mraoint`, `-mcmpccxadd`, `-mprefetchi`, `-mwidekl`

  Enabling instruction in nadya compilation. If these option are defined and target architecture is `x86`, `--cpu`
  options are ignored, so in this case, you need to describe instruction sets specifically.

  Some instruction sets are corresponded to vectorize size. You can check the size above of instruction options. But
  this is not guarantee that your code must be vectorized to the size, compiler just **helps**.

### Options for arm

- `--triple=<value>` : target triple for cross compilation
- `--sysroot=<path>` : system root for cross compilation
- `-mcpu=<cpu>` :

  Target cpu. This option will emit error when `--triple` architecture is not `arm`, `aarch64`. `ndc` analyze it and
  use it to vectorize or choose instruction sets. However, the program which is compiled by `-mcpu` option can't be
  executed in other device.

    - available cpus
      `cortex-a35`, `cortex-a53`, `cortex-a57`, `cortex-a72`, `cortex-a73`, `cortex-a55`, `cortex-a75`, `cortex-a76`, `cortex-a77`, `cortex-a78`, `cortex-x1`, `neoverse-n1`, `cortex-a34`, `cortex-a65`, `neoverse-v1`

- `-march=<arch(+feature1+feature2...)?>` :

  Specify explicitly arm instruction sets version and several features. `ndc` can handle architectures and features
  below. If `-march` were not matched to `-mcpu`, `ndc` will generate a error.

    - **armv4t**
    - **armv5t**
    - **armv6-m**
    - **armv6s-m**
    - **armv7-m**
    - **armv8-m.main**
    - **iwmmxt**
    - **iwmmxt2**
    - **armv5te, armv6, armv6j, armv6k, armv6kz, armv6t2, armv6z, armv6zk**

  `vfpv2`(Alias for fp), `fp`, `nofp`

    - **armv7**

  `vfpv3-d16` (Alias for fp), `fp`, `nofp`

    - **armv7-a**

  `mp`,`sec`,`vfpv3-d16` (Alias
  for fp),`fp`,`nofp`,`simd`,`nosimd`,`vfpv3`,`vfpv3-d16-fp16`,`vfpv3-fp16`,`vfpv4-d16`,`vfpv4`,`neon-fp16`,`neon-vfpv4`

    - **armv7ve**

  `vfpv4-d16` (Alias for
  fp), `fp`, `nofp`, `simd`, `nosimd`, `vfpv3-d16`, `vfpv3`, `vfpv3-d16-fp16`, `vfpv3-fp16`, `vfpv4-d16`, `vfpv4`, `neon`, `neon-fp16`

    - **armv7-r**

  `fp-sp`, `fp`, `nofp`, `idiv`, `noidiv`, `vfpv3xd-d16-fp16`, `vfpv3-d16-fp16`

    - **armv7e-m**

  `dsp`, `nodsp`, `fp-dp`, `nofp`

    - **armv8-m.base**

  `dsp`, `nodsp`, `fp`, `fp-dp`, `nofp`

    - **armv8-r**

  `crc`,`fp-sp`,`simd`,`crypto`,`nocrypto`,`nofp`

    - **armv8-a**

  `crc`, `simd`, `crypto`, `nocrypto`, `nofp`, `sb`, `predres`

    - **armv8.1-a**

  `simd`, `crypto`, `nocrypto`, `nofp`, `sb`, `predres`

    - **armv8.2-a**

  `fp16`, `fp16fml`, `simd`, `crypto`, `dotprod`, `nocrypto`, `nofp`, `sb`, `predres`

    - **armv8.3-a**

  `fp16`, `fp16fml`, `simd`, `crypto`, `dotprod`, `nocrypto`, `nofp`, `sb`, `predres`

    - **armv8.4-a**

  `fp16`, `simd`, `crypto`, `nocrypto`, `nofp`, `sb`, `predres`

    - **armv8.5-a**

  `fp16`, `simd`, `crypto`, `nocrypto`, `nofp`

  #### Options for android

    - `mandroid-clang=<path to clang>`

  For android target, `ndc` requires clang which is built for android cross compilation and usually offered in
  [**Android NDK** packages](https://developer.android.com/ndk?hl=ko). Also, you can use `-fclang-path=<path to clang>`
  instead of this option, but `ndc` can't guarantee that there are not problem.  