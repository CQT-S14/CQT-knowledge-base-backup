
[Lab Projects](../../Lab%20Projects.md) > [Benchmarking Pipeline](../Benchmarking%20Pipeline.md)

# Troubleshooting Benchmarking Pipeline

# How to know something broke

Check if the gh-pages deployment happened the night before at around 22:00: 👉 <https://github.com/Scinawa/CQT-reporting/deployments>

If no deployment appeared, the pipeline failed somewhere. There are **3 possible breaking points**, in order of the pipeline flow:

1. **Job was never submitted (or submitted but crashed)** — the cron scheduler on `nqch-deploy` didn't fire at 21:22, or `benchmarking_pipeline.py` failed to call `sbatch`, or the Slurm job crashed before producing data.
2. **Experiments ran but data wasn't uploaded to AWS DB** — `scripts_executor.py` completed the experiments but failed at the upload step.
3. **Data was uploaded but the reporting side failed** — the `benchmarking` user on `cqt-ale-server` failed to poll, download, generate `report.pdf`, or push to `gh-pages`.

# Step-by-step diagnosis

## Run as `nqch-deploy` on `cqt-paul-server`

### Was the job submitted at all?

Check the most recent Slurm log files and their timestamps:

```java
ls -lt ~/logs/ | head -10
```

You should see a `slurm_sinq20_dev_<JOBID>.out` file timestamped around 21:22. If nothing recent is there, the job was never submitted.

Check what the files say:

```java
cat ~/logs/slurm_sinq20_dev_<JOBID>.err
cat ~/logs/slurm_sinq20_dev_<JOBID>.out
```

### Did the cron scheduler fire?

```java
tail -20 ~/forks-benchmarking-pipeline/CQT-experiments/logs/benchmarking_cron_scheduler.log
```

Look for `"Scheduled time reached"` at 21:22. If it's missing, the cron or the scheduler script is the problem.

### Did the experiments run and finish?

```java
tail -200 ~/forks-benchmarking-pipeline/CQT-experiments/logs/runscripts.log
```

This shows what `scripts_executor.py` did. Look for errors or incomplete runs.

### What did the Slurm job itself output?

```java
cat ~/logs/slurm_sinq20_dev_<JOBID>.out
cat ~/logs/slurm_sinq20_dev_<JOBID>.err
```

Replace `<JOBID>` with the actual job ID from `ls -lt ~/logs/`.

### Was the Slurm job status healthy?

```java
sacct -j <JOB_ID> --format=JobID,JobName,State,ExitCode,Elapsed
```

You want `State=COMPLETED` and `ExitCode=0:0`. Anything else means it failed.

## How to re-launch the pipeline manually

Once you've identified and fixed the problem, re-launch experiments to regenerate results and trigger everything downstream automatically.

### Option A: Direct submission (fastest)

```java
python3 ~/forks-benchmarking-pipeline/CQT-experiments/pipeline/benchmarking_pipeline.py
```

You should see `Submitted batch job <JOBID>`. Then monitor:

```java
squeue -u nqch-deploy
```

Verify the job didn't instantly fail:

```java
sacct -j <JOB_ID> --format=JobID,JobName,State,ExitCode,Elapsed
```

### Option B: Temporary schedule change (lets cron handle it)

1. Edit the scheduler script:

```java
nano ~/forks-benchmarking-pipeline/CQT-experiments/pipeline/benchmarking_cron_scheduler.sh
```

2. Change `SCHEDULED_TIME="21:22"` to a few minutes from now (e.g. `"14:35"`).
3. Wait for that time, then verify it fired:

```java
tail -5 ~/forks-benchmarking-pipeline/CQT-experiments/logs/benchmarking_cron_scheduler.log
squeue -u nqch-deploy
```

4. **Important:** Set `SCHEDULED_TIME` back to `"21:22"` after confirming.

## If the problem is on the AWS side (breaking point 3)

SSH into `cqt-ale-server` as user `benchmarking` and check if the polling script is still running:

```java
screen -ls
```

You should see `benchmarking (Detached)`. If not, the screen session died (e.g. server reboot). Restart it:

```java
screen -S benchmarking -L -Logfile /home/benchmarking/benchmarking.log bash -c '/home/benchmarking/run_benchmarking.sh'
```

Check recent polling activity:

```java
tail -f /home/benchmarking/benchmarking.log
```

(`Ctrl+C` to stop watching — this does NOT stop the script.)

# Quick reference: Log locations

| What | Path | Server |
| --- | --- | --- |
| Cron scheduler log | `~/forks-benchmarking-pipeline/CQT-experiments/logs/benchmarking_cron_scheduler.log` | cqt-paul-server |
| Experiment runner log | `~/forks-benchmarking-pipeline/CQT-experiments/logs/runscripts.log` | cqt-paul-server |
| Slurm job output | `~/logs/slurm_sinq20_dev_<JOBID>.out` | cqt-paul-server |
| Slurm job errors | `~/logs/slurm_sinq20_dev_<JOBID>.err` | cqt-paul-server |
| AWS polling log | `/home/benchmarking/benchmarking.log` | cqt-ale-server |
