### Tutorial on SINA Alignment Tool
SINA is written in C++ and at the moment exist as Linux and Mac executables.  
This document is a short tutorial on installing and running the SINA Alignment tool. 

**Installing SINA from pre-compiled tarballs**  
*I cannot seem to install SINA using Bioconda on Windows using the Conda-Forge and Bioconda channels*

**Step 1.** Tar archives containing pre-compiled binaries and requisite libraries are available on the SINA releases page at Github:  
https://github.com/epruesse/SINA 

Download the Linux version and unpack the tar archive:
```
$ tar xf ~/Downloads/sina-1.7.1-linux.tar.gz
```
**Step 2.** Try running the SINA executable:
```
$ ~/Downloads/sina-1.7.1-linux/sina --help
```
**Step 3.** In running SINA alignment, here are the command line options:  
for a complete list refer to: https://sina.readthedocs.io/en/latest/commandline.html
```
$ ~/sina-1.7.1-linux/sina -i <unaligned> -r <reference> -o <aligned> #SOP

$ ~/sina-1.7.1-linux/sina [ -h | --help | --help-all | --version | --has-cli-vers ]
```
>your **unaligned** file is your query sequence from you own data,  
This is usually in fasta format

>Note that the **reference** file is usually downloaded from the SILVA website: https://www.arb-silva.de/download/arb-files/  
indicated as the recommended reference database.  
>It is based on the SILVA Ref dataset with a 99% criterion applied to remove redundant sequences.  
>This file must be in ARB format.

To convert a reference alignment from FASTA to ARB format in SINA, run:

```
$ ~/sina-1.7.1-linux/sina -i reference.fasta --prealigned -o reference.arb
```
