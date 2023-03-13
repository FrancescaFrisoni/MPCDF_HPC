# Getting started: MPCDF Raven

This tutorial is written for the MPCDF Raven users from the MPI of Animal Behavior. It focuses on using R on Raven and provides specific instructions on how to run jobs that require popular spatio-temporal R packages. This tutorial has been put together and is maintained by the [Animal-Environment Interactions research group](https://www.ab.mpg.de/safi).

The Raven user guide can be found [here](https://docs.mpcdf.mpg.de/doc/computing/raven-user-guide.html#login)

# Step 1: Create an account #

1. Go to https://selfservice.mpcdf.mpg.de/register/antrag.php?lang=en
2. Select MPI of Animal Behavior
3. Pick one person to approve your application


# Step 2: Set up Two-factor authentication (2FA)

1. Install a 2FA app on your mobile phone (e.g. Aegis Authenticator, andOTP, FreeOTP, etc.) 
2. When using the app for the first time, read the QR code available here to connect to the cluster:
https://selfservice.mpcdf.mpg.de/index.php?r=security

Find more info [here](https://docs.mpcdf.mpg.de/faq/2fa.html)

# Step 3: Log in #

**If on the MPI network**
(or using a VPN to connect to one. this is required for the Bücklestraße or uni Konstanz), open the terminal and type:
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

### Note on file systems ###

**File System ``` /u ```**

This is your home directory ``` $HOME ```, i.e. your default working directory when logging in to Raven and when using R. The complete path to this directory is ```/raven/u/username/ ```. The default disk quota in /u is 2.5 TB. 

**File System ``` /ptmp ```**

This directory is designed for batch jobs. The compete path is ```/raven/ptmp/username/ ```. Files in this directory are not backed up and will be removed if not accessed for more than 12 weeks. There is no size limit on this directory, so that the user can manage their data according to their needs. Users are required to remove all files that are not currently used. A best practice is to move your output files to your ``` /u ``` directory and then to your local machine as soon as your batch job is complete. This is done in the example files accompanying this tutorial.

# Step 4: Transfer your files #

Raven does not have access to the files on your local machine. You need to copy the files that you need for your job (e.g. input files, scripts, etc.) to your ``` /u `` directory on Raven. 

From your **local terminal**, use the shell function copy ``` cp ``` or secure copy ```scp``` to move your files to Raven. After each copying attempt, you will be prompted to enter your MPCDF password and the 2FA token.

```sh
scp path_to_file_on_local_machine username@raven.mpcdf.mpg.de:/raven/u/username/
```
