Zcash annotations
=================
This branch describes an experiment in visualizing SNARK proof systems, like those used in Zcash.

SNARK proof systems are encoded as "Rank 1 Constraint Systems." The proof system in Zcash has roughly 2M variables and 2M constraints. Each constraint relates some subset of variables to each other. The variables represent elements of a finite field (like modular arithmetic with a large prime number). Each constraint is of the form `(Ax)(Bx)=(Cx)`, where `A`,`B`,`C` are coefficient vectors, and `x` is the vector of variables.

This framework can express arbitrary computations. For example, to express that variable `X` is either 0 or 1, we would add the constraint `(1-X)(X)=0`.

The Zcash Sprout circuit is defined in the `zcash/circuit/` directory. It makes use of the libsnark `gadgetlib1` circuit composition framework, which is a modular way of building complex circuits using C++ functions and loops.

The gadget library includes a framework for adding annotations to variables and constraints (they are ignored unless `DEBUG` is defined). These annotations are useful for understanding the structure of the circuit. Note that unlike the example gadgets provided in the libsnark, the new gadgets defined in Zcash do not supply annotations, but this branch fixes that. :)
Using these annotations, we can export the circuit to a file, which we can then input into visualization tools and other libraries.

In this experiment, we just repurpose the "GenerateParams" utility to dump the circuit in a format resembling the output of the `du` (disk usage) utility. By feeding this into a treemap library, we get the following visualization of the Sprout circuit:

![zcash sprout circuit treemap](https://raw.githubusercontent.com/amiller/zcash/circuit-annotations/sprout-treemap.png)


Instructions
============
1. Build the zcash dependences
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

5. Generate (takes ~5 minutes on my machine, `sprout.html` is ~150M)
```sh
cat ../sprout.du | node --max-old-space-size=3000 build/cli.js --title 'Sprout R1CS (Vars+Constraints)' > ../sprout.html
```