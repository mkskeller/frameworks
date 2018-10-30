## MP-SPDZ

MP-SPDZ source-code is available at https://github.com/n1analytics/MP-SPDZ.

## Docker setup

Create a Docker image. This will take a few minutes. You only have to do this
once.
```
$ docker build -t mp-spdz .
```

Spin up a Docker container from the image.
```
$ docker run -it --rm mp-spdz
```

## Architecture

While MP-SPDZ allows benchmarking for a number of schemes, only two implementations provide a security guarantee throughout: semi-honest two-party Yao's garbled circuits and covertly secure full-threshold SPDZ-2.

## Yao's garbled circuits

First, compile the example source. We provide three examples (mult3, innerprod, xtabs). The `-B` argument determines the bit length of integers.
```
$ cd MP-SPDZ
$ ./compile.py -B 32 <ex>
```

Generate input for each party.
You'll need to fill in `Player-Data/Input-P<x>-0` with ASCII data for each party
`<x>`.
For example, to run mult3 with inputs 14, 12, and 8, we'd do the following:
```
$ cd MP-SPDZ
$ mkdir Player-Data
$ echo 14 8 > Player-Data/Input-P0-0
$ echo 12 > Player-Data/Input-P1-0
```

Execute the two parties. The output transcript will include the
computed solution.
``` 
$ ./yao-player.x -p 0 <ex> & ./yao-player -p 1 <ex>
```

For example, the output from the following will include `Mult3 prod = 1344`. 
```
$ ./yao-player.x -p 0 mult3 & ./yao-player.x -p 1 mult3
```

## SPDZ-2

### Generating pre-processing data

SPDZ has two parts:
- A pre-processing phase which generates shared randomness between the different parties. 
    This does not depend on the function to be computed and once pre-processing data is generated
    it can be used by many different parties until it is used up. 
    Pre-processing is VERY RESOURCE INTENSIVE. **It needs about 1GB RAM per party.**
- An online phase which uses pre-processing data to evaluate any function. This is quite fast.

Once in the docker, run the pre-processing phase for, say, 3 parties by executing the following:
```
$ cd MP-SPDZ
$ ./spdz2-offline.x -p 0 -N 3 -m &
$ ./spdz2-offline.x -p 1 -N 3 -m &
$ ./spdz2-offline.x -p 2 -N 3 -m
```

This will run for a while to generate the pre-processing data. The generated data is enough for our examples. If you need more, run the pre-processing without `-m`. This will run infinitely; you need to send SIGHUP (Ctrl-C) to stop the programs gracefully once they have generated sufficient data.

You should be able to see the pre-processing data files in MP-SPDZ/Player-Data.

See ./spdz2-offline.cpp for other cmd line options, such as how to specify IP addresses to run on the parties on different machines. The protocol is convertly secure; spdz2-offline.x also has a cmd line option for specifying the covert security parameter.

## Running examples

First, compile the example source. We provide three examples (mult3, innerprod,
xtabs).
```
$ cd MP-SPDZ
$ ./compile.py -p 128 <ex>
```

Generate input for each party and transform it into the SPDZ binary format.
You'll need to fill in `<ex>.P<x>` with correctly-formatted data for each party
`<x>`.
```
$ cd MP-SPDZ
$ mkdir Programs/InputData
$ touch Programs/InputData/<ex>.P<x> 
$ ./gen_input_fp.x -N 3 -i ./Programs/InputData/<ex>.P<x> -o ./Player-Data/Private-Input-<x>
```

For example, to run mult3 with inputs 14, 12, and 8, we'd do the following:
```
$ cd MP-SPDZ
$ mkdir Programs/InputData
$ printf "1\n14\n8\n" > Programs/InputData/mult3.P0 
$ printf "1\n12\n" > Programs/InputData/mult3.P1 
$ printf "0\n" > Programs/InputData/mult3.P1 
$ ./gen_input_fp.x -N 3 -i ./Programs/InputData/mult3.P0 -o ./Player-Data/Private-Input-0
$ ./gen_input_fp.x -N 3 -i ./Programs/InputData/mult3.P1 -o ./Player-Data/Private-Input-1
$ ./gen_input_fp.x -N 3 -i ./Programs/InputData/mult3.P2 -o ./Player-Data/Private-Input-2
```

Execute the server and each player. The output transcript will include the
computed solution.
``` 
$ ./Server.x <# players> <port #> 
$ ./Player-Online.x <player ID> <ex>
```

For example, the output from the following will include `Mult3 prod = 1344`. 
```
$ ./Server.x 3 5000 &
$ ./Player-Online.x 0 mult3 & ./Player-Online.x 1 mult3 & ./Player-Online.x 2 mult3
```

## Modifying examples

Other examples exist in Programs/Source (some of these come with SPDZ, others we wrote). These can be compiled and run in the same way as shown above, though note that the number of parties is different in different examples. You can learn the SPDZ syntax by reading and modifying these examples, as well as by reading MP-SPDZ/tutorial.md.
