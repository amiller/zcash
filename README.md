Zcash annotations
=================
This branch of Zcash is an experiment in visualizing SNARK proof systems.

SNARK proof systems are encoded as arithmetic circuits (more precisely, Rank 1 Constraint Systems). The proof system in Zcash has roughly 2M variables and 2M constraints. Each constraint relates some subset of variables to each other. The variables represent elements of a finite field (like modular arithmetic with a large prime number). Each constraint is of the form `(Ax)(Bx)=(Cx)`, where `A`,`B`,`C` are coefficient vectors, and `x` is the vector of variables. Arbitrary computations and verification routines can be expressed as a circuit. For example, to express that variable `X` is either 0 or 1, we would add the constraint `(1-X)(X)=0`.

The Zcash Sprout circuit is defined in the `zcash/circuit/` directory. It makes use of the libsnark `gadgetlib1` circuit composition framework, which is a modular way of building complex circuits using C++ functions and loops.
The gadget library includes a framework for adding annotations to variables and constraints (they are ignored unless `DEBUG` is defined). These annotations are useful for understanding the structure of the circuit. Note that unlike the example gadgets provided in the libsnark, the new gadgets defined in Zcash do not supply annotations, but this branch fixes that. :)
Using these annotations, we can export the circuit to a file, which we can then input into visualization tools and other libraries.

In this experiment, the goal is to generate a Tree Map representing the Zcash circuit. Here's a screenshot of the final result:
![zcash sprout circuit treemap](https://raw.githubusercontent.com/amiller/zcash/circuit-annotations/sprout-treemap.png)
Each panel in the tree map represents a subcircuit, where its size indicates the number of variables and constraints. We can immediately see that the bulk of the Zcash circuit is accounted for the 58 hash function evaluations needed to verify the Merkle tree branches associated with each transaction input.

To generate the screenshot above, we used the [webtreemap](https://github.com/evmar/webtreemap) library. We repurposed the "GenerateParams" utility to dump the Zcash circuit in a format resembling the output of the `du` (disk usage) utility, which the treemap library expects.

```sh
zcutil/build.sh
```

2. To rebuild just the generation tool 
```sh
make src/zcash/GenerateParams
```

3. Generate the `sprout.du` file for input to the treemap software (takes ~120 seconds on my machine, `./sprout.du` is 548M large)
```sh
src/zcash/GenerateParams ./sproutpk ./sproutvk ./sprout.du
```

4. Build the webtreemap tool
```sh
git submodule update --init --recursive
cd webtreemap
npm run build
```

5. Generate the tree map (takes ~5 minutes on my machine, `sprout.html` is ~150M). The `--max-old-space-size=3000` option increases node's heapsize to 3GB.
```sh
cat ../sprout.du | node --max-old-space-size=3000 build/cli.js --title 'Sprout R1CS (Vars+Constraints)' > ../sprout.html
```
