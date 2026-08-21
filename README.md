# Orbis: Guiding Symbolic Execution Techniques to Maximize Option-Related Branch Coverage

This repository presents the data and results for the paper "Orbis: Guiding Symbolic Execution Techniques to Maximize Option-Related Branch Coverage"

#
### Build Orbis
We recommend building Orbis quickly and easily using a Docker image. To check the build process, refer to the Dockerfile in this repository.
```bash
$ docker pull minjongkim99/orbis-ase26:v1.3
$ docker run -it --ulimit='stack=-1:-1' minjongkim99/orbis-ase26:v1.3 /bin/bash
```

If a program is tested without data in the '/orbis/data/option_dict' and '/orbis/data/opt_branches', Orbis will fail to operate correctly and display an error. In such cases, you can use the provided tracer to generate the data files. Please refer to the README file in the 'tracer' directory for detailed instructions.


### Run Orbis
Finally, you can run Orbis with the following code. (e.g. grep-3.4).

**Quick smoke test (6 Minutes)**
```bash
/orbis/benchmarks $ orbis -p grep -t 360 -d Orbis_TEST grep-3.4/obj-llvm/src/grep.bc grep-3.4/obj-gcov/src/grep
```

**Full Experiment (24 Hours)**
```bash
/orbis/benchmarks $ orbis -p grep -t 86400 -d Orbis_TEST grep-3.4/obj-llvm/src/grep.bc grep-3.4/obj-gcov/src/grep
```

Format : orbis -p <target_program> -t <time_budget> -d <output_dir> <path_to_bc_file(llvm)> <path_to_exec_file(gcov)>
+ -p : Target Program
+ -t : Time Budget (seconds)
+ -d : Output Directory

* If the directory specified by the \texttt{-d} option already exists, Orbis removes the existing directory and creates a new one.

Then, you will see logs as follows.
```bash
[INFO] ORBiS : Coverage will be recorded at "Orbis_TEST/coverage.csv" at every iteration.
[INFO] ORBiS : All configuration loaded. Start testing.
[INFO] ORBiS : Iteration: 1 Iteration budget: 120 Total budget: 360 Time elapsed: 141 Used argument:  Coverage: 1711
[INFO] ORBiS : Iteration: 2 Iteration budget: 120 Total budget: 360 Time elapsed: 283 Used argument: -G Coverage: 2481
```

When the time budget expires without error, you can see the following output.
```bash
[INFO] ORBiS : Iteration: 3 Iteration budget: 120 Total budget: 360 Time elapsed: 360 Used argument: -l Coverage: 2662 
[INFO] ORBiS : Testing done. Achieve 2662 coverage.
```
* For the last iteration, to ensure that the total time budget set by the user is not exceeded, if the iteration budget is bigger than (total budget - elapsed), the budget for that iteration is set to (total budget - elapsed). Therefore, the time budget for the last iteration may be smaller than the actual iteration budget setting.
* Since option configurations are constructed probabilistically, the configurations constructed in each iteration may differ across Orbis runs.
* Due to the probabilistic construction of option arguments, smoke-test coverage results may vary. However, the 24-hour full experiments produce stable results.

## Reporting Results
### Branch Coverage
If you want to get results about how many branches Orbis has covered, run the following command.
```bash
# Needs 'matplotlib' package --> pip3 install matplotlib
/orbis/benchmarks $ pip3 install matplotlib
/orbis/benchmarks $ python3 report_coverage.py --benchmark grep-3.4 Orbis_TEST 
```

And if you want to compare multiple results in a graph, just list the directory names as: 
```bash
/orbis/benchmarks $ python3 report_coverage.py --benchmark grep-3.4 Orbis_TEST1 Orbis_TEST2 ...
```


### Bug Finding
If you want to check information about what bugs Orbis has found, run the following command.
```bash
/orbis/benchmarks $ python3 report_bugs.py --benchmark grep-3.4 Orbis_TEST
```

If the command is executed successfully, you will get a bug report in a file named "bug_result.txt".
```bash
/orbis/benchmarks $ cat bug_result.txt
# Example from find-4.7.0
TestCase : /Orbis_TEST/iteration-3/test000005.ktest
Arguments : "./find" "-amin" "-+NAN" 
CRASHED signal 6
File: ../../find/parser.c
Line: 3143
```


## Usage
```
$ orbis --help
usage: orbis [-h] [--klee KLEE] [--klee-replay KLEE_REPLAY]
             [--gen-bout GEN_BOUT] [--gcov GCOV] [--init-budget INT] [--n-testcases FLOAT]
             [--init-args STR] [-d OUTPUT_DIR] [--src-depth SRC_DEPTH]
             [-t INT] [-p STR]
             [llvm_bc] [gcov_obj]
```


### Optional Arguments
| Option | Description |
|:------:|:------------|
| `-h, --help` | show help message and exit |
| `-d, --output-dir` | Directory where experiment results are saved |
| `--src-depth` | Depth from the obj-gcov directory to the directory where the gcov file was created |


### Executable Settings
| Option | Description |
|:------:|:------------|
| `--klee` | Path to "klee" executable |
| `--klee-replay` | Path to "klee-replay" executable |
| `--gen-bout` | Path to "gen-bout" executable |
| `--gcov` | Path to "gcov" executable |


### Hyperparameters
| Option | Description |
|:------:|:------------|
| `--init-budget` | Time budget for initial iteration |
| `--n-testcases` | Select the top n test cases with high coverage as candidate seeds |
| `--init-args` | Initial symbolic argument formats |

### Required Arguments
| Option | Description |
|:------:|:------------|
| `-t, --budget` | Total time budget of Orbis |
| `llvm_bc` | LLVM bitcode file for klee |
| `gcov_obj` | Executable with gcov support |

## Usage of Other Programs
### /benchmarks/report_bugs.py
```
/orbis/benchmarks$ python3 report_bugs.py --help
usage: report_bugs.py [-h] [--benchmark STR] [--table PATH] [DIRS ...]
```
| Option | Description |
|:------:|:------------|
| `-h, --help`  | Show this help message and exit |
| `--benchmark` | Name of benchmark & version |
| `--table`     | Path to save bug table graph |
| `DIRS`        | Name of directory to detect bugs |


### /benchmarks/report_coverage.py
```
/orbis/benchmarks$ python3 report_coverage.py --help
usage: report_coverage.py [-h] [--benchmark STR] [--graph PATH] [--budget TIME] [DIRS ...]
```
| Option | Description |
|:------:|:------------|
| `-h, --help`  | Show help message and exit |
| `--benchmark` | Name of benchmark & version |
| `--graph`     | Path to save coverage graph |
| `--budget`    | Time budget of the coverage graph |
| `DIRS`        | Names of directories to draw figure |


## Source Code Structure
Here are brief descriptions of the files. Some less-important files may be omitted.
```
.
├── benchmarks                    <Testing & reporting directory>
    ├── building_benchmarks.sh    Building target programs
    ├── report_coverage.py        Reporting branch coverage results
    └── report_bugs.py            Reporting bug-finding results
├── data                          <Saving data during experiments directory>
    ├── constraints               Directory of option-related path conditions for the program
    ├── opt_branches              Directory of option-related branches for the program
    └── option_dict               Directory of program options
├── changed                       <Changed files for each tool>
    ├── osdi08                    https://github.com/klee/klee.git
    ├── fse20                     https://github.com/kupl/HOMI_public.git
    ├── ccs21                     https://github.com/eth-sri/learch.git
    ├── icst21                    https://github.com/davidtr1037/klee-aaqc.git
    └── icse22                    https://github.com/skkusal/symtuner.git
├── tracer                        <Tool for getting option-related data>
└── orbis                         <Main source code directory>
    ├── bin.py                    Entry point of Orbis
    ├── construct.py              Constructing option arguments for each iteration
    ├── extract.py                Extracting options and option-related branches
    ├── guide.py                  Selecting efficient test-cases as seed 
    └── klee.py                   Interacting with symbolic executors (e.g., KLEE)
    
```


## Data Availability
If you want to access data about the experiments of Orbis, you can download it at the following URL:
[https://github.com/minjongkim99/orbis/releases/tag/experimental_result](https://github.com/minjongkim99/orbis/releases/tag/experimental_result)

Download the following file from the URL
+ orbis_experiments.zip

By clicking file or running the following codes on the terminal, you can download the data files.

```
$ wget https://github.com/minjongkim99/orbis/releases/download/experimental_result/orbis_experiments.zip
$ unzip orbis_experiments.zip
```

You can access the test-case directories for 6 programs: xorriso, sqlite, gcal, find, csplit, and ls.

Also, in each test directory, you will see the following files:
+ iteration-* : Iterations that used different option arguments and seed files.
   + info : Log file that expresses the KLEE command for the iteration.
   + test*.ktest : Generated test-cases for the target program. You can use the klee-replay to try each test-case.
+ coverage : Log file of elapsed time, accumulated coverage, coverage of each iteration, and used option arguments.
