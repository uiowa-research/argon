# Array Jobs

For embarrassingly parallel workloads (e.g., repeating a simulation over many data files or parameter seeds), submit a single **Array Job (`-t`)** instead of multiple independent scripts:

```bash
qsub -t 1-100:1 myscript.job
```

### Syntax Format: `-t n[-m[:s]]`
* `n`: Starting index.
* `m`: Ending index.
* `s`: Step size (optional; defaults to `1`).

### Using `$SGE_TASK_ID`
Each task in the array receives an environment variable, `$SGE_TASK_ID`, which corresponds to its task index:

```bash
#!/bin/bash
#$ -cwd
#$ -N array_experiment
#$ -t 1-50

INPUT_FILE="inputs/sample_${SGE_TASK_ID}.dat"
OUTPUT_FILE="results/out_${SGE_TASK_ID}.txt"

./run_analysis --input "$INPUT_FILE" --output "$OUTPUT_FILE"
```

---

More information on Array Jobs can be found in the [Argon wiki page](https://uiowa.atlassian.net/wiki/spaces/hpcdocs/pages/76513479/Array+Jobs).