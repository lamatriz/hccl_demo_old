# HCCL Demo
HCCL demo is a program that demonstrates HCCL usage and supports communication via Gaudi<br />
based Scale out or host NIC Scale out. Host NIC Scale out is achieved using OFI.<br />

The following list supported collective communication:
1. All_reduce
2. All_gather
3. All2All
4. Reduce_scatter
5. Broadcast
6. Reduce

Send/Recv is the supported point to point communication.

Supported from v1.3.0 and above.

## Contents
1. C++ project which includes all tests and a makefile
2. Python wrapper which builds and runs the tests on multiple processes according to the number of devices

## Licensing
Copyright (c) 2022 Habana Labs, Ltd.<br />
SPDX-License-Identifier: Apache-2.0

## Build
The Python wrapper builds and cleans the project (for cleaning please use "-clean").<br />
Alternatively, the project can be built using the following command:<br />
```
make
```
For building the project with MPI:<br />
```
MP1=1 make
```
By default, the demo is built with affinity configuration.<br />
When switching between MPI and non MPI modes, please remember to run with "-clean".

## Python wrapper arguments
    --nranks           - int, Number of ranks participating in the demo
    --ranks_per_node   - int, Number of ranks participating in the demo for current node
    --node_id          - int, ID of the running host. Each host should have unique id between 0-num_nodes
    --test             - str, Which hccl test to run (for example: broadcast/all_reduce) (default: broadcast)
    --size             - str, Data size in units of G,M,K,B or no unit (default: 33554432 Bytes)
    --loop             - int, Number of iterations (default: 10)
    --test_root        - int, Index of root rank for broadcast and reduce tests
    --csv_path         - str, Path to a file for results output
    -mpi               - Use MPI for managing execution
    -clean             - Clear old executable and compile a new one
    -list              - Display a list of available tests
    -help              - Display detailed help for HCCL demo in a form of docstring
    -ignore_mpi_errors - Ignore generic MPI errors
    -no_color          - Disable the usage of colors in console output

## Environment variables
    HCCL_COMM_ID     - IP of node_id=0 host and an available port, in the format <IP:PORT>
    HCCL_OVER_OFI    - 1 to use OFI between servers, 0 (default) to use Gaudi scaleout nics

    Please notice that the flag HCCL_OVER_OFI is optional (since autodetection is supported) and should not be used when Gaudi scaleout nics connected.

## Run

When using any operating system that have Linux kernel version between 5.9.x and 5.16.x. Currently this is applicable to Ubuntu20 and Amazon Linux AMIs:

    echo 0 > /proc/sys/kernel/numa_balancing

Run the execution command

    HCCL_COMM_ID=<IP:PORT> ./run_hcl_demo.py [options]

## Results
Results are printed to the display<br />
Results can also be printed to output file by using --csv_path <path_to_file>

## Examples - without MPI
### Running HCCL on 1 server (8 Gaudi devices)

Configuration: One server with 8 ranks, 32 MB size, all_reduce collective, 1000 iterations

    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 32m --test all_reduce --loop 1000 --ranks_per_node 8

Output example:

    Allreduce hccl_rank=4 size=33554432 <float> Input Buffer [4 12 20 28 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=5 size=33554432 <float> Input Buffer [5 13 21 29 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=1 size=33554432 <float> Input Buffer [1 9 17 25 ... ] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=3 size=33554432 <float> Input Buffer [3 11 19 27 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=2 size=33554432 <float> Input Buffer [2 10 18 26 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=0 size=33554432 <float> Input Buffer [0 8 16 24 ... ] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=7 size=33554432 <float> Input Buffer [7 15 23 31 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=6 size=33554432 <float> Input Buffer [6 14 22 30 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    ###############################################################################
    [BENCHMARK] hcclAllReduce(src!=dst, count=8388608, dtype=fp32, iterations=1000)
    [BENCHMARK]     Bandwidth     : <Test results> MB/s
    ###############################################################################

Different options for running one server with 8 ranks and size of 32 MB:

    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 32m --test all_reduce
    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 32M --test all_reduce
    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 33554432 --test all_reduce
    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 33554432b --test all_reduce
    HCCL_COMM_ID=127.0.0.1:5555 python3 run_hccl_demo.py --nranks 8 --node_id 0 --size 33554432B --test all_reduce
### Running HCCL demo on 2 servers (16 Gaudi devices)

Configuration: Host NIC Scale out using OFI, 16 ranks, 32 MB size, all_reduce collective, 1000 iterations

First server command:

    HCCL_COMM_ID=10.111.12.234:5555 HCCL_OVER_OFI=1 python3 run_hccl_demo.py --test all_reduce --nranks 16 --loop 1000 --node_id 0 --size 32m --ranks_per_node 8

Second server command:

    HCCL_COMM_ID=10.111.12.234:5555 HCCL_OVER_OFI=1 python3 run_hccl_demo.py --test all_reduce --nranks 16 --loop 1000 --node_id 1 --size 32m --ranks_per_node 8

First server output:

    Allreduce hccl_rank=0 size=33554432 <float> Input Buffer [0 16 32 48 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=3 size=33554432 <float> Input Buffer [3 19 35 51 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=7 size=33554432 <float> Input Buffer [7 23 39 55 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=4 size=33554432 <float> Input Buffer [4 20 36 52 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=6 size=33554432 <float> Input Buffer [6 22 38 54 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=1 size=33554432 <float> Input Buffer [1 17 33 49 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=2 size=33554432 <float> Input Buffer [2 18 34 50 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=5 size=33554432 <float> Input Buffer [5 21 37 53 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    ###############################################################################
    [BENCHMARK] hcclAllReduce(src!=dst, count=8388608, dtype=fp32, iterations=1000)
    [BENCHMARK]     Bandwidth     : <Test results> MB/s
    ###############################################################################

Second server output:

    Allreduce hccl_rank=13 size=33554432 <float> Input Buffer [13 29 45 61 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=12 size=33554432 <float> Input Buffer [12 28 44 60 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=11 size=33554432 <float> Input Buffer [11 27 43 59 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=15 size=33554432 <float> Input Buffer [15 31 47 63 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=9 size=33554432 <float> Input Buffer  [9 25 41 57 ... ] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=8 size=33554432 <float> Input Buffer  [8 24 40 56 ... ] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=14 size=33554432 <float> Input Buffer [14 30 46 62 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=10 size=33554432 <float> Input Buffer [10 26 42 58 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.

## Examples - MPI mode
### Running HCCL on 1 server (8 Gaudi devices)

All available MPI options are supported.<br />
* For MPI different running options please refer to: https://www.open-mpi.org/faq/?category=running#mpirun

Configuration: One server with 8 ranks, 32 MB size, all_reduce collective, 1000 iterations

    python3 run_hccl_demo.py --size 32m --test all_reduce --loop 1000 -mpi -np 8

Output example:

    Allreduce hccl_rank=4 size=33554432 <float> Input Buffer [4 12 20 28 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=5 size=33554432 <float> Input Buffer [5 13 21 29 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=1 size=33554432 <float> Input Buffer [1 9 17 25 ... ] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=3 size=33554432 <float> Input Buffer [3 11 19 27 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=2 size=33554432 <float> Input Buffer [2 10 18 26 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=0 size=33554432 <float> Input Buffer [0 8 16 24 ... ] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=7 size=33554432 <float> Input Buffer [7 15 23 31 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    Allreduce hccl_rank=6 size=33554432 <float> Input Buffer [6 14 22 30 ...] reduced to Output Buffer [28 92 156 220 ...] which is fine.
    ###############################################################################
    [BENCHMARK] hcclAllReduce(src!=dst, count=8388608, dtype=fp32, iterations=1000)
    [BENCHMARK]     Bandwidth     : <Test results> MB/s
    ###############################################################################

### Running HCCL demo on 2 servers (16 Gaudi devices)

Configuration: Host NIC Scale out using OFI, 16 ranks, 32 MB size, all_reduce collective, 1000 iterations

First option using MPI hostfile:

    python3 run_hccl_demo.py --test all_reduce --loop 1000 --size 32m -mpi --hostfile hostfile.txt -x HCCL_OVER_OFI=1

* For MPI --hostfile option, please refer to: https://www.open-mpi.org/faq/?category=running#mpirun-hostfile

Second option using MPI host:

    python3 run_hccl_demo.py --test all_reduce --loop 1000 --size 32m -mpi --host 10.111.12.234,10.111.12.235 -x HCCL_OVER_OFI=1

* For MPI --host option, please refer to: https://www.open-mpi.org/faq/?category=running#mpirun-host

First server output:

    Allreduce hccl_rank=0 size=33554432 <float> Input Buffer [0 16 32 48 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=3 size=33554432 <float> Input Buffer [3 19 35 51 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=7 size=33554432 <float> Input Buffer [7 23 39 55 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=4 size=33554432 <float> Input Buffer [4 20 36 52 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=6 size=33554432 <float> Input Buffer [6 22 38 54 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=1 size=33554432 <float> Input Buffer [1 17 33 49 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=2 size=33554432 <float> Input Buffer [2 18 34 50 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=5 size=33554432 <float> Input Buffer [5 21 37 53 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    ###############################################################################
    [BENCHMARK] hcclAllReduce(src!=dst, count=8388608, dtype=fp32, iterations=1000)
    [BENCHMARK]     Bandwidth     : <Test results> MB/s
    ###############################################################################

Second server output:

    Allreduce hccl_rank=13 size=33554432 <float> Input Buffer [13 29 45 61 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=12 size=33554432 <float> Input Buffer [12 28 44 60 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=11 size=33554432 <float> Input Buffer [11 27 43 59 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=15 size=33554432 <float> Input Buffer [15 31 47 63 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=9 size=33554432 <float> Input Buffer  [9 25 41 57 ... ] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=8 size=33554432 <float> Input Buffer  [8 24 40 56 ... ] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=14 size=33554432 <float> Input Buffer [14 30 46 62 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.
    Allreduce hccl_rank=10 size=33554432 <float> Input Buffer [10 26 42 58 ...] reduced to Output Buffer [120 376 632 888 ...] which is fine.