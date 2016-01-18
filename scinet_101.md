# Scinet GPC basics

The GPC is the [General Purpose Cluster](https://wiki.scinet.utoronto.ca/wiki/index.php/GPC_Quickstart), where most of our work on SciNet HPC system will be done.

Scinet is a collection of compute nodes, each of which has multiple processors. This setup allows us to run jobs in a highly parallel fashion ; Scinets large disk quota (10Tb) means that multiple tests can be run at once without requiring cleanup between tests.

The system is **shared**, which means we submit a compute **job** to a **queue**. 


Links to more comprehensive resources:

* [scinet tutorial full doc](https://wiki.scinet.utoronto.ca/wiki/images/5/54/SciNet_Tutorial.pdf) -- download. section 2 (`Using the GPC`) is the section we want.
* [scinet courses](https://support.scinet.utoronto.ca/education/browse.php)-- sign up for a 1-hour "Intro to Scinet" workshop (hosted monthly)
* [scinet wiki](https://wiki.scinet.utoronto.ca/)  -- is GPC down?

## Overall flow

The overall flow for working on scinet and your compute is as follows:

* Code should always be synchronized between a system such as github and scinet. 

* Data should be imported/exported off scinet after each major set of tests. 
* The big working directory, `/scratch/` gets cleaned periodically. Every 3 months, you will get an email listing which files will be deleted. You will need to update the timestamp on these files, or preferrably delete/export them, to **avoid losing them forever**.

Suggested steps:

1. Prototype on small scale on your laptop
2. Sync code and data to scinet. (this should happen regularly. For code, atleast once a day).
3. Write small scale jobs. Submit to `debug` queue. 
4. When ready to scale up, submit to regular queue.
5. Run jobs till satisfied with outcome.
6. Export data to backup machines/laptop.

## Login and first job

First get an account from CCDB, get your PI to approve it by email. Here we will work with the username `spai`.

Login to the login node:

```
$ ssh -X spai@login.scinet.utoronto.ca
```

NOTE: you can define a shortcut command for this operation in your `.bashrc` file. That way you just type `$SCINET` at command line. Faster.

**Login node:** You never work on this node, you just use it as a temporary stop to a second node. Here we can pick any of `gpc01-gpc08` and choose `gpc03`.

```
scinet02-ib0-$ ssh gpc03
Last login: Mon Jan 18 09:14:14 2016 from scinet02-ib0
gpc-f103n084-ib0-
```

Now lets run a dummy job. You need to `load` a `module` to use most software (e.g. R, python, java). Here we use the command to load a lightweight code editor `nano`.

```
$ module load nano 
$ nano myFirstJob.sh
```

In the file, type:
```
#!/bin/bash
#PBS -l nodes=1:ppn=8,walltime=00:20:00

echo "Cluster test successful."
```

Notice the line starting with `#PBS`. The cluster recognizes these as commands to the job queuing system. That particular line says we are requesting `1` node for `20` minutes (HH:MM:SS). When done, save the file and exit to command-line. Now make it runnable and test that it works.

```
$ chmod u+x myFirstJob.sh
$ ./myFirstJob.sh
```

and submit the job to run on the debug queue.

```
$ qsub myFirstJob.sh -q debug
33589696.gpc-sched-ib0
```

Your job is submitted! That number `335...` is your job ID. Now see the status of your job:

```
gpc-f103n084-ib0-$ qstat
Job ID                    Name             User            Time Use S Queue
------------------------- ---------------- --------------- -------- - -----
33589696.gpc-sched-ib0     myFirstJob.sh    spai                   0 Q debug
```

The `S` column shows the status; our job is queued. When the job is complete, the status will show as being 'C'ompleted.

```
gpc-f103n084-ib0-$ qstat
Job ID                    Name             User            Time Use S Queue
------------------------- ---------------- --------------- -------- - -----
33589696.gpc-sched-ib0     myFirstJob.sh    spai            00:00:00 C debug
```

Every job automatically produces two files:

1. `<myJob>.o<jobID>` : output of job
2. `<myJob>.e<jobID>` : error messages

If the job completed successfully, #2 should be empty. In this test, all we did was print a message. So that message should be printed in the `.o` file.

```
gpc-f103n084-ib0-$ ls myFirstJob.sh*
myFirstJob.sh  myFirstJob.sh.e33589696  myFirstJob.sh.o33589696
```

Lets look at job output:
```
gpc-f103n084-ib0-$ cat myFirstJob.sh.o33589696 
----------------------------------------
Begin PBS Prologue Mon Jan 18 09:32:03 EST 2016 1453127523
Job ID:         33589696.gpc-sched-ib0
Username:       spai
Group:          gbader
Nodes:          gpc-f109n001-ib0
End PBS Prologue Mon Jan 18 09:32:04 EST 2016 1453127524
----------------------------------------
Cluster test successful.
----------------------------------------
Begin PBS Epilogue Mon Jan 18 09:32:10 EST 2016 1453127530
Job ID:         33589696.gpc-sched-ib0
Username:       spai
Group:          gbader
Job Name:       myFirstJob.sh
Session:        23915
Limits:         neednodes=1:ppn=8,nodes=1:ppn=8,walltime=00:10:00
Resources:      cput=00:00:00,mem=0kb,vmem=0kb,walltime=00:00:02
Queue:          debug
Account:
Nodes:  gpc-f109n001-ib0
Killing leftovers...

End PBS Epilogue Mon Jan 18 09:32:11 EST 2016 1453127531
----------------------------------------

```

The job output is prefixed and suffixed by job details.
And in the middle is our test output. 

Hurray! Now we can do more interesting things.


