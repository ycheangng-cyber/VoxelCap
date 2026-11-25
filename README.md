# VoxelCap

VoxelCap distributes electrostatic field solver and four voxelized device models for capacitance extraction. The solver reads an EWF geometry and a material table. In capacitance mode (`C`), it writes a `.capMatrix` text file with per-net totals and pairwise coupling capacitances in picofarads (pF). Each dataset includes a reference matrix for comparison.

## Repository layout

| Path | Contents |
| --- | --- |
| `bin/` | The main executable `leptonfd`, the companion `lepton` executable, test programs, utilities, and compiled libraries. |
| `dataset/pftn/` | The main example. `Step4.25.ewf` contains the geometry, `matlib.txt` defines the materials, and `nets.txt` maps coordinates to net names. The directory also includes `Step4.25.ewf.capMatrix`, a layout image (`layout.jpg`), and a port table (`porttable.txt`). |
| `dataset/RC/Test7/` | A small two-conductor example with `Step.ewf`, `materials.txt`, and the reference `Step.ewf.capMatrix`. |
| `dataset/RC/Test9/`, `dataset/RC/Test10/` | Two further cases with the same geometry, material, and reference filenames as Test7. Each also supplies its own `nets.txt`. |

## Linux runtime

Use an **x86-64 GNU/Linux** environment. The bundled `bin/leptonfd` is a dynamically linked, position-independent ELF64 executable with debug information. Its interpreter is `/lib64/ld-linux-x86-64.so.2`.

| Binary detail | Value |
| --- | --- |
| Build ID (SHA-1) | `1d785227481ef0d71c3578c6f08c6f6df0b4e8e1` |
| Build compiler | GCC 7.5.0 (Ubuntu `7.5.0-3ubuntu1~18.04`) |
| ABI | Linux ABI tag 3.2.0. The highest glibc and libstdc++ symbol versions required directly by the executable are `GLIBC_2.15` and `GLIBCXX_3.4.21`. |
| Direct shared libraries | `libhdf5_cpp.so.320`, `libzstd.so.1`, `libz.so.1`, `libstdc++.so.6`, `libm.so.6`, `libgomp.so.1`, `libgcc_s.so.1`, `libpthread.so.0`, `libc.so.6` |


## Reproduce the PFTN result

Start at the repository root. The commands copy the three solver inputs into a fresh directory and print its location. Keep the same shell open for the comparison step.

```sh
repo=$(pwd -P)
case_dir="$repo/dataset/pftn"
run_dir=$(mktemp -d)
cp "$case_dir/Step4.25.ewf" "$case_dir/matlib.txt" "$case_dir/nets.txt" "$run_dir/"
cd "$run_dir"
pwd

"$repo/bin/leptonfd" C matlib.txt Step4.25.ewf \
  -FVM -M -G -renametable nets.txt -tol1e-6 \
  -block-rhs 8 -nt 32 -nystrom-coarse-cholesky -nystrom-memory-mb 2048000
```

This invocation uses the finite-volume solver, requests a solver tolerance of `1e-6`, batches eight right-hand sides, and requests 32 threads. The Nyström coarse Cholesky option has a memory budget of `2048000` MB. That value is a solver setting. Actual peak memory depends on the case.
