# guidelines_workflow_project

My notes / personal guidelines for a project workflow. This is typically the kind of workflow that can be relevant for a research project, where there are many dependencies, the code has to be runnable "in all future" following code release, etc. This is NOT the kind of workflow that should be used for running things operationally - as the following will naturally lead to having end of life, outdated software.

## Using a singularity container / image to make fully reproducible

To make things fully reproducible over time, for example when releasing a scientific paper with code, a form of static container is necessary. Having a robust conda install or apt installation script or similar is, in my experience, not enough. For example, some of the dependencies in specific version may disapper from the web in the future (this has happened to me, some versions of well known packages were needed in one of my conda env, and after several years, these versions were removed from the internet). By contrast, as long as the container specification does not change in a backwards incompatible way, it should remain possible to run a given container / image far in the future.

Moreover, using a container makes running the code independent of the host (as long as the host can run singularity in the following). This contrasts to, for example, using conda or pip (or even more apt), in which case, some key dependencies may "leak" between the host and the development environment, things may become host OS dependent, etc.

In the following, I assume that a singularity container / image is used to containerize the application. Of course, inside the container, conda, apt, or other, may be used to install software - this is perfectly fine, as all the dependencies are present / archived inside the container. So, if the container needs to be re-run in many years, all the software from conda and apt will still be part of it and available - even if the specific software versions have disappeared from the internet in the meantime.

The most convenient way to "get things done" in a research context is to use the singularity container in a writable sandbox way, as this allows installing software inside it on the fly. Then, once all the needed dependencies are inside it (and possibly the code that needs to be reproducible in the future, too), this can be packed into a .tar.gz file for long term archiving.

### Create sandbox

To create a sandbox (here, based on Ubuntu 22.04, but another base image could be used):

```
~/Desktop/Current/SingularityPlayground> singularity build --fakeroot --sandbox singularity_sandbox docker://ubuntu:22.04
INFO:    Starting build...
2023/04/18 14:40:52  info unpack layer: sha256:2ab09b027e7f3a0c2e8bb1944ac46de38cebab7145f0bd6effebfe5492c818b6
INFO:    Creating sandbox directory...
INFO:    Build complete: singularity_sandbox
```

Note that:

- ```fakeroot``` is used to give sudo rights inside the container even if we do not have sudo rights by defaults outside of it
- this is building a ```sandbox```, i.e., the container will actually live in a new folder ```./singularity_sandbox/```, which will contain all the files etc necessary to run the container:

```
~/Desktop/Current/SingularityPlayground> ls -l singularity_sandbox/
total 60
lrwxrwxrwx  1 jrmet jrmet    7 mars   8 03:05 bin -> usr/bin
drwxr-xr-x  2 jrmet jrmet 4096 april 18  2022 boot
drwxr-xr-x  2 jrmet jrmet 4096 mars   8 03:08 dev
lrwxrwxrwx  1 jrmet jrmet   36 april 18 14:40 environment -> .singularity.d/env/90-environment.sh
drwxr-xr-x 32 jrmet jrmet 4096 mars   8 03:08 etc
drwxr-xr-x  2 jrmet jrmet 4096 april 18  2022 home
lrwxrwxrwx  1 jrmet jrmet    7 mars   8 03:05 lib -> usr/lib
lrwxrwxrwx  1 jrmet jrmet    9 mars   8 03:05 lib32 -> usr/lib32
lrwxrwxrwx  1 jrmet jrmet    9 mars   8 03:05 lib64 -> usr/lib64
lrwxrwxrwx  1 jrmet jrmet   10 mars   8 03:05 libx32 -> usr/libx32
drwxr-xr-x  2 jrmet jrmet 4096 mars   8 03:05 media
drwxr-xr-x  2 jrmet jrmet 4096 mars   8 03:05 mnt
drwxr-xr-x  2 jrmet jrmet 4096 mars   8 03:05 opt
drwxr-xr-x  2 jrmet jrmet 4096 april 18  2022 proc
drwx------  2 jrmet jrmet 4096 mars   8 03:08 root
drwxr-xr-x  5 jrmet jrmet 4096 mars   8 03:08 run
lrwxrwxrwx  1 jrmet jrmet    8 mars   8 03:05 sbin -> usr/sbin
lrwxrwxrwx  1 jrmet jrmet   24 april 18 14:40 singularity -> .singularity.d/runscript
drwxr-xr-x  2 jrmet jrmet 4096 mars   8 03:05 srv
drwxr-xr-x  2 jrmet jrmet 4096 april 18  2022 sys
drwxrwxrwt  2 jrmet jrmet 4096 mars   8 03:08 tmp
drwxr-xr-x 14 jrmet jrmet 4096 mars   8 03:05 usr
drwxr-xr-x 11 jrmet jrmet 4096 mars   8 03:08 var
```

### Work inside sandbox

To start working inside the sanbox:

```
~/Desktop/Current/SingularityPlayground> singularity shell --writable --fakeroot --no-home singularity_sandbox/
WARNING: Skipping mount /etc/localtime [binds]: /etc/localtime doesn't exist in container
Singularity> exit
exit
```

note that this is quite minimal, if you want software inside, you will need to install it:

```
~/Desktop/Current/SingularityPlayground> singularity shell --writable --fakeroot --no-home singularity_sandbox/
WARNING: Skipping mount /etc/localtime [binds]: /etc/localtime doesn't exist in container
Singularity> python3
bash: python3: command not found
Singularity> apt update        
...
Singularity> apt install python3
...
Python 3.10.6 (main, Mar 10 2023, 10:55:28) [GCC 11.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
Singularity> exit
exit
~/Desktop/Current/SingularityPlayground> 
```

### Tar sandbox and moving it around

Once you are happy with the sandbox content (it contains all the dependencies needed, the code that needs to be made runnable in the future, etc), you can tar the sandbox into a file:

```
~/Desktop/Current/SingularityPlayground> sudo tar czf singularity_sandbox.tar.gz singularity_sandbox/
[sudo] password for jrmet: 
```

Note that the file you obtain may be quite large (not in this case though, my image is near empty):

```
~/Desktop/Current/SingularityPlayground> ls -lrth
total 68M
drwxr-xr-x 18 jrmet jrmet 4,0K april 18 14:40 singularity_sandbox
-rw-r--r--  1 root  root   68M april 18 14:48 singularity_sandbox.tar.gz
```

At this point, the sandbox archive may be moved around. A note: if, like me, you often end up doing things in slightly "ad hoc" ways, for example, sharing a sandbox .tar.gz with colleagues through GoogleDrive and Dropbox, or putting on github with git-lfs so the code and container live at the same location, you may hit weird issues with large container archives. An archive with a lot of dependencies inside it may easily reach 5+GB, and this seems to be a point at which data corruption, problems with uploads, etc, are common (are some services using a FAT32 storage in the background? ...). To solve this, a simple solution is to slice the archive into several files (as done in https://github.com/jerabaul29/docker_image_falling_fluid_film ). For this:

- slice the archive (note: in this example, I slice in 20MB slices, but of course, for a large image that is many GBs, you can use larger slices; 500MB to 1GB seems to work fine in my experience):

```
~/Desktop/Current/SingularityPlayground> split -b 20M singularity_sandbox.tar.gz singularity_sandbox.tar.gz__part.
~/Desktop/Current/SingularityPlayground> ls -lrth
total 135M
drwxr-xr-x 18 jrmet jrmet 4,0K april 18 14:40 singularity_sandbox
-rw-r--r--  1 root  root   68M april 18 14:48 singularity_sandbox.tar.gz
-rw-rw-r--  1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.aa
-rw-rw-r--  1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.ab
-rw-rw-r--  1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.ac
-rw-rw-r--  1 jrmet jrmet 7,5M april 18 15:07 singularity_sandbox.tar.gz__part.ad
```

- to allow future users to check integrity of all files, compute the sha256 checksums, to be distributed in a txt file together with the segments:

```
~/Desktop/Current/SingularityPlayground> sha256sum singularity_sandbox.tar.gz*
1933a043c7ce251008ef51f61ab1dab75980be6f170afd903e1e67929c85422d  singularity_sandbox.tar.gz
581a5522eae7e8d6a737355ae73615ea8e383f206f7aa53a4f90ad9e2b5f6a75  singularity_sandbox.tar.gz__part.aa
7db0cd720597e600262269c63ad7cbcdb9bc38b9bcbb1b5003c0e89446bab26a  singularity_sandbox.tar.gz__part.ab
3973a0a4e208499dddc40c3e66b55ac164f81c1e370ef379759335ae50bde169  singularity_sandbox.tar.gz__part.ac
12d2c016db5dab36f224b6b7e492e9bd6c8ba4ebc4412cb2a76996ee01c6e41e  singularity_sandbox.tar.gz__part.ad
```

You can now distribute the (reasonably heavy) slices on any system you want.

- to put them together, the user should check the integrity (sha256sum) of all slices, and them put them back together:

```
~/Desktop/Current/SingularityPlayground_fromslice> ls -lrth
total 68M
-rw-rw-r-- 1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.aa
-rw-rw-r-- 1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.ab
-rw-rw-r-- 1 jrmet jrmet  20M april 18 15:07 singularity_sandbox.tar.gz__part.ac
-rw-rw-r-- 1 jrmet jrmet 7,5M april 18 15:07 singularity_sandbox.tar.gz__part.ad
~/Desktop/Current/SingularityPlayground_fromslice> cat singularity_sandbox.tar.gz__part.?? > singularity_sandbox.tar.gz
~/Desktop/Current/SingularityPlayground_fromslice> sha256sum singularity_sandbox.tar.gz
1933a043c7ce251008ef51f61ab1dab75980be6f170afd903e1e67929c85422d  singularity_sandbox.tar.gz
~/Desktop/Current/SingularityPlayground_fromslice> tar xfp singularity_sandbox.tar.gz 
-rw-rw-r--  1 jrmet jrmet  68M april 18 15:11 singularity_sandbox.tar.gz
~/Desktop/Current/SingularityPlayground_fromslice> singularity shell --writable --fakeroot --no-home singularity_sandbox/
WARNING: Skipping mount /etc/localtime [binds]: /etc/localtime doesn't exist in container
Singularity> python3
Python 3.10.6 (main, Mar 10 2023, 10:55:28) [GCC 11.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
Singularity> exit
exit
```

As you can see, I could get back into my singularity container with the python3 I had installed inside.

## Installing software nside Singularity

Once inside the container, you can use any tool to install software (and you have admin rights by default, no need to sudo as visible above). Still, for convenience, I would recommend trying to install using (higher should be what to attempt first):

- conda
- pip
- apt
- custom packages

Remember that the base image is very barebone: you may need to install things as fundamental as pythong, vim, git, etc, yourself.
