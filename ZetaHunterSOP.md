### Tutorial on ZetaHunter 

ZetaHunter is a command line script written in Ruby  

To run ZetaHunter via Ruby in Windows you need a Linux environment  

*if you already have a working Linux environment skip to Step 6*  
*if you already have Ruby installed you can skip to Step 11*

installation of a Linux environment can be done with WSL (Windows Subsystem for Linux) use of WSL2 is ideal

>Requirements for WSL2:  
>For x64 systems: Version 1903 or higher, with Build 18362 or higher.  
>To check your version and build number, select Windows logo key + R, type winver, select OK. 

**Step 1** - Enable Windows Subsystem for Linux. Open PowerShell as Administrator and run:
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

**Step 2** - Enable Virtual Machine Platform feature by entering the following in PowerShell as Administrator:
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Restart your machine to complete the WSL install and update to WSL 2  

**Step 3** - Download the Linux kernel update package with the following link:
 https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi  
run the update package to install and move on to the next step

**Step 4** - Set WSL 2 as your default version  
open PowerShell as Administrator and run the following command:
```
wsl --set-default-version 2
```
**Step 5** - Install your Linux distribution: Ubuntu 20.04 LTS on Windows Store  
**Optional Step** - You can check the WSL version assigned to each of the Linux distributions you have installed by opening the PowerShell command line and entering the command:
```
wsl --list --verbose
```
**Now we proceed to installing Ruby and ZetaHunter**  
**Step 6** - Install curl by entering the following on your Linux terminal: 
```
 $ sudo apt-get curl
```

**Step 7** - Install Ruby Version Manager (RVM) by first installing GPG keys used to verify the installation package
```
$ gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```
**Step 8** - Install Pre-requisites. You need software-properties-common in order to add PPA repositories. Run the following on your Linux terminal:
```
$ sudo apt-get install software-properties-common
```
**Step 9** - Add the PPA and install the RVM package
```
$ sudo apt-add-repository -y ppa:rael-gc/rvm
$ sudo apt-get update
$ sudo apt-get install rvm
```
Reboot

**Step 10** - Install Ruby. For superuser permissions you can use sudorvm
```
$ rvm install ruby
```

To check your ruby version run:
```
$ rvm current
```

### Install ZetaHunter and its dependencies
**Step 11** - Install Bundler, which is used to manage dependencies for Ruby projects, by running the following on your linux terminal

```
$ gem install bundler
```
**Step 12** - Download the latest release from the release tab in https://github.com/mooreryan/ZetaHunter/ and place your tar file to your desired directory.  
Extract your tar file:
```
$ tar xzf ZetaHunter-1.0.11.tar.gz
```
and navigate to your ZetaHunter folder:
```
$ cd ~/ZetaHunter-1.0.11
```
>**IMPORTANT:**   
before running bundle install in the next step, you may need to change permissions and ownership of your Gemfile.lock and ZetaHunter folder. To do this you may need to restart your Ubuntu 20.04 LTS, Run as Administrator, navigate to your ZetaHunter directory and run the following:
```
chmod ugo+rwx Gemfile.lock
chmod ugo+rwx ~/ZetaHunter-1.0.11
```
If you get an error, you might also need to change file and folder ownership by running:
```
chown <name of user> Gemfile.lock
chown <name of user> ~/ZetaHunter-1.0.11
```
You can check file and folder permissions and ownership by running:
```
ls -l
```

**Step 13** Install the Ruby dependencies for ZetaHunter. Run the following while in your ZetaHunter folder
```
$ bundle install
```
**Step 14** Try running ZetaHunter. You can also use sudorvm for superuser permissions. The latter might help as ZetaHunter uses /tmp folders
```
$ ruby zeta_hunter.rb -h
$ sudorvm ruby zeta_hunter.rb -h
```
**Step 15** In  using ZetaHunter, here are the command line options:
```
 Options:
  -i, --inaln=<s+>                             Input alignment(s)
  -o, --outdir=<s>                             Directory for output
  -t, --threads=<i>                            Number of processors to use (default: 2)
  -d, --db-otu-info=<s>                        Database OTU info file name (default: /Users/moorer/projects/ZetaHunter/assets/db_otu_info.txt)
  -m, --mask=<s>                               Fasta file with the mask (default: /Users/moorer/projects/ZetaHunter/assets/mask.fa.gz)
  -b, --db-seqs=<s>                            Fasta file with aligned DB seqs (default: /Users/moorer/projects/ZetaHunter/assets/db_seqs.fa.gz)
  -r, --mothur=<s>                             The mothur executable (default: /Users/moorer/projects/ZetaHunter/bin/mac/mothur)
  -s, --sortmerna=<s>                          The SortMeRNA executable (default: /Users/moorer/projects/ZetaHunter/bin/mac/sortmerna)
  -n, --indexdb-rna=<s>                        The SortMeRNA idnexdb_rna executable (default: /Users/moorer/projects/ZetaHunter/bin/mac/indexdb_rna)
  -c, --cluster-method=<s>                     Either furthest, average, or nearest (default: average)
  -u, --otu-percent=<i>                        OTU similarity percentage (default: 97)
  -k, --check-chimeras, --no-check-chimeras    Flag to check chimeras (default: true)
  -a, --base=<s>                               Base name for output files (default: ZH_2018_07_15_13_29)
  -e, --debug                                  Debug mode, don't delete tmp files or clean up the working dir (the out dir will be empty)
  -v, --version                                Print version and exit
  -h, --help                                   Show this message
```
>**NOTE:** one of the inputs required by ZetaHunter are SINA aligned sequences  
>Check this tutorial for installing and running the SINA alignment tool:  
>insert github link here

as an example, you can run ZetaHunter with just the following file:

- aligned.fasta ; which is your SINA aligned sequence that required a SILVA arb database and your fasta file

using the following code:   

```
rvmsudo ruby zeta_hunter.rb -i aligned.fasta -o ~/ZetaHunter/zetaoutput --no-check-chimeras -r ~/ZetaHunter/bin/linux/mothur -s ~/ZetaHunter/bin/linux/sortmerna -n ~/ZetaHunter/bin/linux/indexdb_rna
```
*NOTE that I skipped the chimera checking step since mothur keeps on getting errors*

