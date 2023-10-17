---
title: "RHCSA - Managing Software"
date: 2023-10-15T21:07:00+03:00
draft: false
hideToc: false
enableToc: true
enableTocContent: false
description: "A software management cheatsheet for RHCSA"
author: Wissam
authorEmoji: 💻
tags: 
- RHCSA
- Linux
- Red Hat
categories:
- Linux
series:
- RHCSA Notes
---


## `dnf` / `yum` command

- **YUM** used to be an acronym for **Yellowdog Updater Modified**, **DNF** is the revolution of **YUM** and it stands for **Dandified YUM**.
- in RHEL 9 the `yum` command is just a symbolic link to `dnf`. 
- To search for a package:

```shell
dnf search <package_name>
```

- To search what package provides a specific file use `whatprovides` or just `provides` option :

```shell
dnf whatprovides /usr/bin/ls
dnf provides */ssh
```

- to get more information about a package:

```shell
dnf info <package_name>
```

- To install a package (you need root privileges to install and remove packages):

```shell-session
dnf install <package_name>
```

- You can install an rpm package on your file system or from a URL:

```shell-session
dnf install ~/Downloads/filename.rpm
dnf instal https://blahblah.com/filename.rpm
```

- to remove a package:

```shell-session
dnf remove <package_name>
```

- To list all software packages that are available:

```shell
dnf list
```

- To list all software packages that are installed:

```shell
dnf list installed
```

- To list all software packages that have updates:

```shell
dnf list updates
```

- To show which version of a package is installed and which version is available as the most recent version in the repositories:

```shell
dnf list <package_name>
```

- To list all installed packages whose names begin with "bash":

```bash-session
$ dnf list installed "bash*"
Installed Packages
bash.x86_64                                                          5.2.15-1.fc37                                                @updates 
bash-completion.noarch                                               1:2.11-8.fc37                                                @anaconda
```

- To list all packages available for installation from all enabled repos:

```shell
dnf repoquery
```

- To list all packages available only from a specific repo:

```shell
dnf repoquery --repo <repo_name>
```

- Determine if a package is available for installation in enabled repos:

```bash
dnf repoquery <package_name>
```

- To update a package:


```shell
dnf update <package_name>
```

- To show a list of transactoins done by `dnf` :

```shell
dnf history
```

- To show information about a specific transacton (get the ID number from the output of the previous command:

```shell
dnf history info <ID>
```

- To undo a transaction:

```shell-session
dnf history undo <ID>
```

- To list your repos:

```shell
dnf repolist
```

## Package Groups

- Groups are virtual collections of packages that make it easier to manage specific functionality. For example, if you want to install the packages needed to work with kvm virtual machines, you can install the "virtualization" package group instead of installing all the packages needed individually
- For an overview of all current groups: 

```shell
dnf groups list
```

- Some groups are not listed by default. To show all groups

```shell
dnf groups list hidden
```

- To show installed groups:

```shell
dnf group list installed
```

- To get information about the "Web Server" group:

```shell
dnf groups info "Web Server"
```

- To install a group:

```shell
sudo dnf group install <group_name>
```

- To update a group:

```shell
sudo dnf group update <group_name>
```

- To remove a group:

```shell
sudo dnf group remove <group_name>
```

- To display the number of installed and available package groups:

```shell
dnf group summary
```


## Modules, Streams and Profiles
- Modules are special package groups usually representing an application, a language runtime, or a set of tools
- Modules are available in one or multiple streams which usually represent a major version of a piece of software.
- A profile is a list of recommended packages that are installed together for a particular use case

- To list available modules and information about their streams and profiles.

```bash-session
$ dnf modules list
...
Name                Stream            Profiles                          Summary                                                            
avocado             latest            default [d], minimal              Framework with tools and libraries for Automated Testing           
avocado             82lts             default, minimal                  Framework with tools and libraries for Automated Testing           
avocado-vt          latest            default                           Avocado Virt Test Plugin                                           
avocado-vt          82lts             default                           Avocado Virt Test Plugin                                           
cri-o               1.20              default [d]                       Kubernetes Container Runtime Interface for OCI-based containers    
cri-o               1.21              default [d]                       Kubernetes Container Runtime Interface for OCI-based containers    
cri-o               1.22              default [d]                       Kubernetes Container Runtime Interface for OCI-based containers    
cri-o               1.24              default                           Kubernetes Container Runtime Interface for OCI-based containers    
ghc                 8.10              all, default [d], minimal, small  Haskell GHC 8.10                                                   
ghc                 9.2               all, default, minimal, small      Haskell GHC 9.2                                                    
mariadb             10.5              client, devel, galera, server [d] MariaDB: a very fast and robust SQL database server                
mariadb             10.6              client, devel, galera, server     MariaDB: a very fast and robust SQL database server                
mariadb             10.7              client, devel, galera, server     MariaDB: a very fast and robust SQL database server                
mariadb             10.8              client, devel, galera, server     MariaDB: a very fast and robust SQL database server                
mariadb             10.9              client, devel, galera, server     MariaDB: a very fast and robust SQL database server                
...
Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

- As you can see, different versions of `mariadb` are available via different streams and `10.5`  is the default stream and  `server`  is the default profile (Notice the `[d]`) 
- To show information about `mariadb` module in all module streams:

```shell
dnf module info mariadb
```

- To install a module:

```shell
sudo dnf module install <module-name>
```

- To update a module:

```shell
sudo dnf module update <module-name>
```

- To remove a module:

```shell
sudo dnf module remove <module-name>
```

- To show information about a specific stream:

```shell
dnf module info mariadb:10.5
```

- You can install a different stream in two ways:
  1. by enabling the module and then installing the package:
  ```shell
  sudo dnf enable mariadb:10.9
  sudo dnf install mariadb
  ```
  2. install the module directly:
  ```shell
  sudo dnf install mariadb:10.9
  ```

- To install a module specifying the stream and the profile :

```shell
sudo dnf module install NAME:STREAM/PROFILE
```

- Example:

```shell
sudo dnf module install mariadb:10.9/client
```

- To disable a module:

```shell
sudo dnf module disable <module-name>
```

- Not specifying a stream or a profile causes `dnf` to choose the default

- To reset the module so that neither stream is enabled or disabled:

```shell
sudo dnf module reset <module-name>
```

- To list installed modules:

```shell
dnf module list --installed
```

- To list enabled modules:

```shell
dnf module list --enabled
```

- To list disabled modules:

```shell
dnf module list --disabled
```


## `rpm` Command

- rpm file names usually look like this:
![rpmfilenames](rpm_filenames.svg#center)

- To list all software installed on the machine

```shell
rpm -qa
```

- to get a description of an installed package:

```shell
rpm -qi <package_name>
```

- To list all files that are in the package. 

```shell
rpm -ql <package_name>
```

- To show all documentation available for the package:

```shell
rpm -qd <package_name>
```

- To show all configuration files for the package:

```shell
rpm -qc <package_name>
```

- To find the package name that a specific file belongs to

```shell
rpm -qf /etc/passwd
```

- To query an rpm package file we use the `-p` option
- To query an rpm package file to see the scripts it contains before installing (and you should when installing packages from unknown sources)

```shell
rpm -qp --scripts package.rpm
```

- To show which parts of a specific package have been changed since installation:

```shell
sudo rpm -V <package_name>
```


- To verify all installed packages and shows which parts of the package have been changed since installation:

```shell
sudo rpm -Va
```

- To install a package:

```shell
sudo rpm -i package.rpm
```


- To install a package and print progress `-v` and hash marks `-h` as the package archive is unpacked:

```shell
sudo rpm -ivh package.rpm
```

- To upgrade a package:

```shell
sudo rpm -Uvh package.rpm
```

{{<notice info>}}
  If the package is not installed this will simply install the package
{{</notice>}}

- To remove (erase) a package:

```shell
sudo rpm -ev <package_name>
```
