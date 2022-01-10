# SliQSim - A BDD-based Quantum Circuit Simulator

## Introduction
`SliQSim` is a BDD-based quantum circuit simulator implemented in C/C++ on top of [CUDD](http://web.mit.edu/sage/export/tmp/y/usr/share/doc/polybori/cudd/cuddIntro.html) package. In `SliQSim`, a bit-slicing technique based on BDDs is used to represent quantum state vectors. For more details of the simulator, please refer to the [paper](https://arxiv.org/abs/2007.09304).

## Build
To build the simulator, one needs to first configure `CUDD`:
```commandline
cd cudd
./configure --enable-dddmp --enable-obj --enable-shared --enable-static 
cd ..
```
Next, build the binary file `SliQSim`:
```commandline
make
```
You can jump to the next section if you found the binary `SliQSim` . However, if an error message similar to the following shows up, we suggest to build again with the solution provided below. (Tested on WSL: Ubuntu-18.04 in 2021/10)
```
make[1]: Entering directory 'SliQSim/cudd'
CDPATH="${ZSH_VERSION+.}:" && cd . && /bin/bash SliQSim/cudd/build-aux/missing aclocal-1.16 -I m4
SliQSim/cudd/build-aux/missing: line 81: aclocal-1.16: command not found
WARNING: 'aclocal-1.16' is missing on your system.
         You should only need it if you modified 'acinclude.m4' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'aclocal' program is part of the GNU Automake package:
         <http://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <http://www.gnu.org/software/autoconf>
         <http://www.gnu.org/software/m4/>
         <http://www.perl.org/>
Makefile:983: recipe for target 'aclocal.m4' failed
make[1]: *** [aclocal.m4] Error 127
make[1]: Leaving directory 'SliQSim/cudd'
Makefile:11: recipe for target 'all' failed
make: *** [all] Error 2
```
Solution to the error message (the only difference is the second line of command added):
```commandline
cd cudd
autoreconf -f -i
./configure --enable-dddmp --enable-obj --enable-shared --enable-static 
cd ..
```

## Execution
The circuit format being simulated is `OpenQASM` used by IBM's [Qiskit](https://github.com/Qiskit/qiskit), and the gate set supported in this simulator now contains Pauli-X (x), Pauli-Y (y), Pauli-Z (z), Hadamard (h), Phase and its inverse (s and sdg), π/8 and its inverse (t and tdg), Rotation-X with phase π/2 (rx(pi/2)), Rotation-Y with phase π/2 (ry(pi/2)), Controlled-NOT (cx), Controlled-Z (cz), Toffoli (ccx and mcx), SWAP (swap), and Fredkin (cswap). One can find some example benchmarks in [examples](https://github.com/NTU-ALComLab/SliQSim/tree/master/examples) folder. 

For simulation types, we provide both weak and strong simulation options. The help message states the details:

```commandline
$ ./SliQSim --help
Options:
--help                produce help message
--sim_qasm arg        simulate qasm file string
--seed [=arg(=1)]     seed for random number generator
--print_info          print simulation statistics such as runtime,
                      memory, etc.
--type arg (=0)       the simulation type being executed.
                      0: weak simulation (default option) where the sampled 
                      outcome(s) will be provided after the simulation. The
                      number of outcomes being sampled can be set by argument
                      "shots" (1 by default).
                      1: strong simulation where the resulting state vector
                      will be shown after the simulation. Note that this
                      option should be avoided if the number of qubits is
                      large since it will result in extremely long runtime.
--shots arg (=1)      the number of outcomes being sampled, this argument is
                      only used when the simulation type is set to "weak".
--r arg (=32)         integer bit size.
--reorder arg (=1)    allow variable reordering or not.
                      0: disable reordering.
                      1: enable reordering (default option).
--alloc arg (=1)      allocate new BDDs when overflow is detected.
                      0: do not allocate new BDDs. This may lead to numerical
                      errors.
                      1: allocate new BDDs (default option).
```
For example, simulating [example/bell_state_measure.qasm](https://github.com/NTU-ALComLab/SliQSim/blob/master/examples/bell_state_measure.qasm), which is a 2-qubit bell state circuit with measurement gates at the end, with the weak simulation option can be executed by
```commandline
./SliQSim --sim_qasm examples/bell_state_measure.qasm --type 0 --shots 1024
```

Then the sampled results will be shown:
```commandline
{ "counts": { "11": 542, "00": 482 } }
```

If option `--print_info` is used, simulation statistics such as runtime and memory usage will also be provided: 
```commandline
  Runtime: 0.014433 seconds
  Peak memory usage: 12611584 bytes
  #Applied gates: 2
  Max #nodes: 13
  Precision of integers: 32
  Accuracy loss: 2.22045e-16
```

To demostrate the strong simulation, we use [example/bell_state.qasm](https://github.com/NTU-ALComLab/SliQSim/blob/master/examples/bell_state.qasm), which is the same circuit as in the weak simulation example except that the measurement gates are removed:
```commandline
./SliQSim --sim_qasm examples/bell_state.qasm --type 1
```

This will show the resulting state vector:
```commandline
{ "counts": { "00": 1 }, "statevector": ["0.707107", "0", "0", "0.707107"] }
```

One may also execute our simulator as a backend option of Qiskit through [SliQSim Qiskit Interface](https://github.com/NTU-ALComLab/SliQSim-Qiskit-Interface).


## Citation
Please cite the following paper if you use our simulator for your research:

<summary>
  <a href="https://arxiv.org/abs/2007.09304">Y. Tsai, J. R. Jiang, and C. Jhang, "Bit-Slicing the Hilbert Space: Scaling Up Accurate Quantum Circuit Simulation to a New Level," 2020, arXiv: 2007.09304</a>
</summary>

```bibtex
@misc{tsai2020bitslicing,
      title={Bit-Slicing the Hilbert Space: Scaling Up Accurate Quantum Circuit Simulation to a New Level}, 
      author={Yuan{-}Hung Tsai and Jie{-}Hong R. Jiang and Chiao{-}Shan Jhang},
      year={2020},
      note={arXiv: 2007.09304}
}
```

## Contact
If you have any questions or suggestions, feel free to [create an issue](https://github.com/NTU-ALComLab/SliQSim/issues), or contact us through matthewyhtsai@gmail.com.