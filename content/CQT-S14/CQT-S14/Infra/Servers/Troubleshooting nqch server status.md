
[Infra](../../Infra.md) > [Servers](../Servers.md)

# Troubleshooting nqch server status

- [Investigate problems with sinq20 partition](#troubleshootingnqchserverstatus-investigateproblemswithsinq20partition)
  - [Check what is running in sinq20 partition](#troubleshootingnqchserverstatus-checkwhatisrunninginsinq20partition)
  - [Check detailed node info (why nodes are drained)](#troubleshootingnqchserverstatus-checkdetailednodeinfowhynodesaredrained)
  - [Check if there are other partitions you can use](#troubleshootingnqchserverstatus-checkifthereareotherpartitionsyoucanuse)
  - [Check history of your past and current Slurm jobs](#troubleshootingnqchserverstatus-checkhistoryofyourpastandcurrentslurmjobs)

# Investigate problems with sinq20 partition

The following are some commands to try when your slurm jobs are not successfully submitted, you see hanging jobs or any other type of problem that initially is not clear where it originates.

#### **Check what is running in sinq20 partition**

```java
squeue -p sinq20

# Eg. output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
47600    sinq20 bihourly nqch-dep  R      34:53      1 nqch-login
      
```

#### **Check detailed node info (why nodes are drained)**

```java
sinfo -p sinq20 -o "%n %t %C %E" -N

# Eg. output
HOSTNAMES STATE CPUS(A/I/O/T) REASON
nqch-login alloc 1/0/0/1 none

```

#### **Check if there are other partitions you can use**

```java
sinfo

# Eg. output
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
sim*         up   infinite      2   idle nqch-node[1-2]
sim-heavy    up   infinite      2   idle nqch-qpu[1-2]
gpu          up 1-00:00:00      2   idle nqch-gpu[1,3]
sinq20       up 1-00:00:00      1   idle nqch-login
```

#### **Check history of your past and current Slurm jobs**

```java
sacct

# Eg. output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
47601              bash     sinq20     eortiz          0 CANCELLED+      0:0 
```
