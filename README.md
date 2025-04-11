# Getting started: MPCDF Raven HPC

This tutorial is written for the Max Planck Computing and Data Facility (MPCDF) Raven HPC users from the MPI of Animal Behavior. It focuses on using R on Raven and provides specific instructions on how to run jobs that require popular spatio-temporal R packages. This tutorial has been put together and is maintained by the [Animal-Environment Interactions research group](https://www.ab.mpg.de/safi).

The Raven user guide can be found [here](https://docs.mpcdf.mpg.de/doc/computing/raven-user-guide.html#login)

## Overview of the workflow

The Raven HPC system is a cluster of interconnected computers that is designed to process large amounts of data and perform complex computations. Raven's workflow involves submitting a job to the system, which is then processed in a queue.

A job is a set of instructions that tells the system what computations to perform. It is submitted using a batch system, which allows the user to specify the resources needed to run the job, such as the number of processors, the amount of memory, and the amount of time needed to complete the job. When the job is completed, the results are stored in the file system for retrieval or later analysis.

To make the whole process more enjoyable for yourself, have a look at these tutorials on [directory management in Unix](https://www.tutorialspoint.com/unix/unix-directories.htm) and [basics of shell scripting](https://www.tutorialspoint.com/unix/unix-what-is-shell.htm) if these topics are new to you.

## Step 1: Create an account

1.  Go to <https://selfservice.mpcdf.mpg.de/register/antrag.php?lang=en>
2.  Select MPI of Animal Behavior
3.  Pick one person to approve your application

## Step 2: Set up Two-factor authentication (2FA)

1.  Install a 2FA app on your mobile phone (e.g. Aegis Authenticator, andOTP, FreeOTP, etc.)
2.  When using the app for the first time, read the QR code available here to connect to the cluster: <https://selfservice.mpcdf.mpg.de/index.php?r=security>

Find more info [here](https://docs.mpcdf.mpg.de/faq/2fa.html)

## Step 3: Log in

**If on the MPI network** (or using a VPN to connect to one. this is required for the Bücklestraße or uni Konstanz), open the terminal and type:

```sh
ssh username@raven.mpcdf.mpg.de
```

Enter your MPCDF password and then the token generated by the 2FA app on your phone.

**If not on the MPI network**, open the terminal and type:

```sh
ssh username@gate.mpcdf.mpg.de
```

Enter your MPCDF password and then the token generated by the 2FA app on your phone.

Then, ssh into Raven:

```sh
ssh username@raven.mpcdf.mpg.de
```

Enter your MPCDF password and then the token generated by the 2FA app on your phone.

FF: after this another terminal page will open, with your username@raven01 or @raven02 as identifier. This is the communication channel to the cluster, pay attention whether using this or the local terminal for the following lines of code.

### Note on file systems

**File System `/u`**

This is your home directory `$HOME`, i.e. your default working directory when logging in to Raven and when using R. The complete path to this directory is `/raven/u/username/`. The default disk quota in `/u` is 2.5 TB.

**File System `/ptmp`**

This directory is designed for batch jobs. The compete path is `/raven/ptmp/username/`. Files in this directory are not backed up and will be removed if not accessed for more than 12 weeks. There is no size limit on this directory, so that the user can manage their data according to their needs. Users are required to remove all files that are not currently used. A best practice is to move your output files to your `/u` directory and then to your local machine as soon as your batch job is complete. This is done in the example files accompanying this tutorial.

## Step 4: Transfer your files

Raven does not have access to the files on your local machine. You need to copy the files that you need for your job (e.g. input files, scripts, etc.) to your `/u` directory on Raven.

From your **local terminal**, use the shell function copy `cp` or secure copy `scp` to move your files to Raven. After each copying attempt, you will be prompted to enter your MPCDF password and the 2FA token.

```sh
scp path_to_file_on_local_machine username@raven.mpcdf.mpg.de:/raven/u/username/
```

## Step 5: Load the required software packages and test your code

MPCDF uses environment modules to provide software packages and enable using different software versions. No modules are automatically loaded, so load the modules that require using the command `module load package_name/version`. For example, load the R module as follows:

```sh
module load R/4.4
```

FF: When checking the R version after this code, you may notice that your local R environment is 4.4.3, while in Raven is loaded as 4.4.1; you can ignore this difference, Raven is always a bit less updated than the CRAN last release.


Some other useful module commands:

```sh
module avail # see all available modules
find-module R # to locate modules and its dependencies
module avail R # see all available versions
module load R # loads the default version of R
module unload R # unload R
module purge # unload all currently loaded modules
module show R # see details of the module
module list # shows all currently loaded modules
```

Before going ahead and starting R, open a screen. A screen will allow you to go back to your last instance in case your connection to Raven is interrupted, your terminal window closes, etc. Here are some useful screen related commands:

```sh
screen -S my_screen_name # open a new screen
screen -r my_screen_name # open already existing (detached) screen 
screen -list # see a list of created screens 
screen -d -r my_screen_name # open a screen that is still attached
screen -S my_screen_name -X quit  # kill a screen or use ctrl+AK
screen ???? # detach or use ctrl+AD
```

Now, create a new screen and open R to test your code: `sh  screen -S new_R_screen  R`

Once in R, you can test your code to make sure that you can install the necessary libraries, read in your file, and overall make sure that your code (or a small version of it) works on the cluster. You will then have more confidence in your code before submitting a batch job using the slurm batch system.

``` r
install.packages("tidyverse")
install.packages("move")
```

DO NOT RUN long scripts or your entire job here! The node that you long into is only for editing and managing your data. If you run memory- or time-consuming jobs, you will get an email from the MPCDF asking you to stop the job.

**NOTE**: if you have issues installing R libraries, like e.g. `sf`, `units`, `ctmm` or others, one solution is to build a container and work within a docker. Instructions on how to do that are here ["Using_apptainer.md"](Using_apptainer.md).

FF: I would recommend to install ALL packages in the apptainer and just load the libraries into the R script submitted to the cluster. In the apptainer tutorial, remember to install as many packages as you know you will use, in one go. This will save you time next time you may need an additional package, because to get to the apptainer you need to do all the passages again.
FF: about the apptainer, when installing and loading packages, you may have some errors or warnings: ignore them, everything is alright.

## Step 6: Prepare your slurm file

### Note on SLURM

SLURM (Simple Linux Utility for Resource Management) is a job scheduler and resource manager used by the Raven HPC system. It is a software system that helps manage the allocation of computing resources (such as processors, memory, and storage) on a cluster of computers, so that jobs can be run efficiently and effectively.

Based on the resources that you need, your job will be either **exclusive**, where all resources on the nodes are allocated to the job, or **shared**, where several jobs share the resources of one node. In this case it is necessary that the number of CPUs and the amount of memory are specified for each job. Overview of the available per-job resources on Raven is as follows. See the [Raven user guide](https://docs.mpcdf.mpg.de/doc/computing/raven-user-guide.html#login) for more information.

```         

    Job type          Max. CPUs            Number of GPUs   Max. Memory      Number     Max. Run
                      per Node               per node        per Node       of Nodes      Time
   =============================================================================================
    shared    cpu     36 / 72  in HT mode                     120 GB          < 1       24:00:00
    ............................................................................................
                      18 / 36  in HT mode        1            125 GB          < 1       24:00:00
    shared    gpu     36 / 72  in HT mode        2            250 GB          < 1       24:00:00
                      54 / 108 in HT mode        3            375 GB          < 1       24:00:00
   ---------------------------------------------------------------------------------------------
                                                              240 GB         1-360      24:00:00
    exclusive cpu     72 / 144 in HT mode                     500 GB         1-64       24:00:00
                                                             2048 GB         1-4        24:00:00
    ............................................................................................
    exclusive gpu     72 / 144 in HT mode        4            500 GB         1-80       24:00:00
    exclusive gpu bw  72 / 144 in HT mode        4            500 GB         1-16       24:00:00
   ---------------------------------------------------------------------------------------------
```
A job submit will automatically choose the right partition and job parameters from the resource specification.


### The SLURM script

The SLURM file is a shell program that contains instructions for the cluster and the job that is to be run. The header includes the instructions. The lines starting with `#SBATCH` are SLURM directives. Here a detailed explanation of each of the elements contained in the scripts (also see [example scripts](/example_scripts/)): 

- specifies the shell to be used to run the script
```sh
#!/bin/bash -l 
```
- Create files with the standard output and error. It is recommended to save these files within a specific folder (e.g. here called "messages"), specially if you are running many jobs, or jobs as a array, many of these files will be created. Remember to delete these files when not needed any more to not use up unnecessary space.
The `job.out.%j` files contain the information on the job duration, how much memory it needed and the CPU utilization.
The `job.err.%j` files contain everything that is being printed in the R console, therefore also the errors and is very useful fo debugging the code.
`%j` will be the job ID. Use `_%A_%a` in array jobs, it will append the jobID_arrayIndex
```sh
#SBATCH -o ./messages/job.out_%j
#SBATCH -e ./messages/job.err_%j

## if the job is an array:
#SBATCH -o ./messages/job.out_%A_%a
#SBATCH -e ./messages/job.err_%A_%a
```
- Initial working directory. You can specify a directory in which the R and slrm file can be found and the folder where the output and error (from above) will be stored.
```sh
#SBATCH -D ./your_directory/
```
- When using apptainer, the initail working directory has to contain the .sif file, so most probably the home directory.
```sh
#SBATCH -D ./
```
- Give your job a name. This name will appear in the job.out and job.err files and in the list of jobs running or queuing when checking with e.g. `squeue -u <user_name>` (see below)
```sh
#SBATCH -J your_job_name
```
- Setting number of nodes, cpus and memory. Depending on the type of job you run, a different combination of options is needed. The standard node has 72 CPUs and 120GB memory (see table above). If you ask for (or need) one entire node, the job will be queued until one entire node is available and that can take time. Requiring an entire node can be done setting the memory to 120GB or by setting the CPUs per task to 72. To speed up the process, one should put some thought in what is really needed and not just asking for a high number.

Description of options:    
**`--nodes`**: is the number of nodes, if it is and array job, you can set to several nodes, but only if you request at least half of the node, i.e. 36 CPUs per task. Assumption: if you request all memory or CPUs of the nodes the waiting time will probaly increase and slow down the process.      
**`--ntasks-per-node`**: assumption: this value cannot be greater than 72. In the case of an array job that has as input very heavy files, it might be worth while to state how many jobs should be done on a single node. When iterations of the same job run on different nodes, all files are copied to each node and that can take time. In all other cases, it is probably faster if the system distributes the jobs as space is made available.    
**`--cpus-per-task`**: a single job will always run on a single core. If one is doing paralelization within the R script, here the number of CPUs should be specified. This number has to be the same as the one set in the R script via e.g. `doMC::registerDoMC()` or `doParallel::registerDoParallel()`.     
**`--array`**: this is the number of times the R code will be executed. The maximum is 300 at a time. The numbers stated here will be the value "i" in the R script (see example file). If you have more than 300, you will have to submit the next job when the previous has finished, stating in the second one e.g. 301-600, and so on.    
**`--mem`**: (in MB) by limiting the memory of each job, the nodes can be shared for multiple jobs, of the same user or of different users. Set the memory according to what you think you will need, do tests, and check in the "job.out_jobId" file created above the memory needed for a test run (+~20%). Remember that if you assign the entire memory of a node (i.e. 120Gb) but are only using 1 CPU, this will block the entire node. If you need that much memory, it is fine, but if not you’ll be only using 1/20 of the power. Try to optimize the memory you need, so you can use multiple CPUs per node.    


**A. single job - R code is run once**: given that a single job always will run on one CPU, and therefore on 1 node

```sh
#SBATCH --nodes=1 ## another value does not make sense
#SBATCH --ntasks-per-node=1 ## another value does not make sense
#SBATCH --cpus-per-task=1 ## another value does not make sense
#SBATCH --mem=8000 ## what ever is required by your job 
```
**B. parallelization type 1 - array job: R script runs multiple times**, each time with different input data. No parallelization in the R code.

```sh
#SBATCH --nodes=1 ## another value does not make sense
#SBATCH --ntasks-per-node=1 ## can be multiple tasks per node, but can also be omited and system will optimally distribute the jobs
#SBATCH --cpus-per-task=1 ## another value does not make sense
#SBATCH --array=1-300 # this is the number of times your R code will be executed, the maximum is 300
#SBATCH --mem=8000 ## what ever is required by your job. This is the memory per each instance (each time the R script is run)
```

**C. parallelization type 2: R script runs once, but there is paralelization in the R code**

```sh
#SBATCH --nodes=1 ## another value does not make sense
#SBATCH --ntasks-per-node=1 ## another value does not make sense
#SBATCH --cpus-per-task=12 ##this number of CPUs has to be the same as the one stated in the R script
#SBATCH --mem=8000 ## what ever is required by your job.
```
**D. array job with with parallelization within the R code**: combination of B and C

```sh
#SBATCH --nodes=15 ## can be multiple nodes, but only when at least 50% of the cpus are requested, i.e. when "--cpus-per-task" 36 or more, if not job submission will give error
#SBATCH --ntasks-per-node=10 ## can be multiple tasks per node, but can also be omitted and system will optimally distribute the jobs
#SBATCH --cpus-per-task=36 ##this number of CPUs has to be the same as the one stated in the R script
#SBATCH --array=1-300 ## this is the number of times your R code will be executed, the maximum is 300
#SBATCH --mem=8000 ## what ever is required by your job. This is the memory per each instance (each time the R script is run)
```

- Setting the max duration of each instance (max. is 24 hours). A single job (with internal parallelization or not) can run max. 24h. With array jobs, each instance can run max. 24. Use the information of the job.out_jobId file of the test run to estimate the necessary time. Do not use 24h by default, try be state the time that you estimate that you actually need +20%

```sh
#SBATCH --time=00:05:00  
```
- Set an email alert. If `mail-type=all` than you will get an email when your job starts, is completed or failed. This is very useful than alternatively having to constantly login an check if the job is completed.
```sh
#SBATCH --mail-type=all 
#SBATCH --mail-user=your_email_address@ab.mpg.de
```

The script then continues with loading the required modules and running the R script. Load the same modules that you used in Step 5 when testing your code.

```sh
# Load compiler and modules:
module purge 
module load R/4.2

# Run the program:
R CMD BATCH your_R_script.R 2>&1 errorlog
```

The `2>&1 errorlog` command will write all messages that R produces while running your program (including warning and error messages) to an `errorlog` file. This is very helpful for debugging your code.

Save your slurm script as: your_slrm_script.slrm

If you are using a apptainer (see detailed instructions here ["Using_apptainer.md"](Using_apptainer.md)), the two sections above are slightly different:

```sh
# Load compiler and MPI modules. Probably other module needs to loaded as all is happening within the container
module purge
module load apptainer/1.2.2

# Run the program if your data is in the home directoy:
srun apptainer exec geospatial_latest_updated.sif Rscript myproject/myscripts/my_R_script.r

# Run the program if your data is on ptmp, it has to be mounted:
srun apptainer exec --bind /ptmp/<your_username> geospatial_latest_updated.sif Rscript myproject/myscripts/my_R_script.r 

# Run the program if job is submitted as an array:
srun apptainer exec geospatial_latest_updated.sif Rscript myproject/myscripts/my_R_script.r $SLURM_ARRAY_TASK_ID
```

FF: 
To create, access and edit the file: username@raven01:~> nano oneJob_apptainer.slrm
You can copy the example file from Anne "oneJob_apptainer.slrm", as you built an apptainer and want to submit only one job for now. 
You don't have to change much of it.
Remember to add the name of the script you want to run into the cluster, which you have already submitted with scp (check right directory).
Remember to change your email address to receive notifications.
Remember to change memory limit and time instructions. 
mem=8000 means 8Gb, you can go up to mem=120000, but try to stay below it.
time=00:05:00 means 5 minutes, you can go up to time=24:00:00, but try to stay below it because if not needed all this time, your batch job will be higher in priority and faster to execute.
After editing: ctrl O to save the changes, press ENTER to save the name of the file, ctrl X to exit the editing page.

## Step 7: Submit your job

Make sure that you have your slrm script, R script, and any input files on Raven. It's easier if they are all in one directory, which you can specify in your slurm script as your working directory. 

FF: you can check the files in your directories with username@raven01:~> ls -l; this will return the complete list of files loaded and their location, as well as their size and date of uploading.

The maximum number of jobs a user can run or have queuing simultaneously are 300, independently if they are coming from an array job or multiple separate jobs.

```sh

sbatch your_slrm_script.slrm
```

FF: this command needs to run into your Raven terminal page,  username@raven01:~> sbatch oneJob_apptainer.slrm
after which you'll see this message: Submitted batch job 16681835
You will also receive a mail telling when your job was submitted to the cluster, and how long the time in queue was.

Other useful commands:

```sh
squeue -u <user_name> # Check the status of your job(s)
scontrol show job <job_id> # Details of job status
scancel <job_id> # Cancel a job
sinfo # List the available batch queues (partitions).
```

FF: squeue is very useful to check the status, but if the Raven terminal stops working or gets stuck here, don't panic, the job is still running in the cluster and it will de-stuck itself.

FF: *Debugging errors*
If you receive a mail with a failed job, but your not sure about the error (for example, fail code 01), you can check in the Raven terminal:
username@raven01:~> ls ./myproject/messages/
here you will get a list of your jobs and related errors (job.out and job.err), then you need to open the specific case as:
username@raven01:~> cat ./myproject/messages/job.err_16681835
here you will get the panel of the specific job and a clarification of the type of error your job incurred into. Often is a problem of the above settigns in the slrm file: too little memory (srun: error: ravc4029: task 0: Out Of Memory) devoted or not enough time range.

## Step 8: Transfer your output files

Just like copying the files from the local system to Raven, to copy your files from Raven to your local machine can be done using `cp` or secure copy `scp` from your **local terminal**. After each copying attempt, you will be prompted to enter your MPCDF password and the 2FA token. You can copy all contents of a directory, and any subdirectories using the recursive `-r` flag.

```sh
scp username@raven.mpcdf.mpg.de:/raven/u/username/file_to_copy path_to_target_directory_on_local_machine
scp -r username@raven.mpcdf.mpg.de:/raven/u/username/dir_to_copy path_to_target_directory_on_local_machine
```

## Using the cluster after the first installation:

FF: *To keep in mind*
Every time you change the cluster R script (in case of errors here), you need to reload it from the local terminal. 
When changed the slrm file, it is already in Raven so just need to save the changes.
If working in the terminal is too abstract, you can download FileZilla as platform to communicate with the Cluster. 

About the R script to load into Raven: remember to
- load the libraries at the beginning of the script
- set the working directories to Raven and not your local machine
- save the results of your analyses as Rdata, pdf, png, csv etc to the raven specified directories
-- the script needs to be completely detatched from your local machine to be able to work well

FF: *Steps to follow each time*
For Linux.
Local terminal: sudo openfortivpn
Open a new terminal page: ssh username@raven.mpcdf.mpg.de - after adding your password and OTP, this page becomes the connection to raven as: username@raven01
Here you can check what is already in your deposit: username@raven01:~> ls -l
If needed, you can load more files from your local machine: open a new page into your local terminal(not the same one of the vpn), and load new files with scp path_to_file_on_local_machine username@raven.mpcdf.mpg.de:/raven/u/username/ (see above)
If you need to access the slrm file to change it (memory, time, new script): 
- only to visualize it: username@raven01:~> cat oneJob_apptainer.slrm
- to access and edit the file: username@raven01:~> nano oneJob_apptainer.slrm
- after editing: ctrl O to save the changes, press ENTER to save the name of the file, ctrl X to exit the editing page
After everything is ready, you can open a screen and submit the batch job: username@raven01:~> sbatch oneJob_apptainer.slrm -- Submitted batch job 16681835
If job failed, debug as above: username@raven01:~> ls ./myproject/messages/ -- username@raven01:~> cat ./myproject/messages/job.err_16681835
After correcting the mistake, re-submit the batch job. If you had to change the R code, remember to load it again with scp from your local terminal.
When job completed, download results into local machine from the local terminal: scp username@raven.mpcdf.mpg.de:/raven/u/username/file_to_copy path_to_target_directory_on_local_machine

