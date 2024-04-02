# DNF
## what we do

In this post is show you who to create a package that you create with your commend in fedora.
For this we need to install python, rpm and createrepo.
First we install python (if you dont have).

```bash
$ sudo dnf install python
```

Create your script
If you dont have a script and want to learn who to create a package you can take the scrip below.

```python
#!/usr/bin/env python3
from datetime import datetime
import os
import shutil
import sys

today_date=datetime.now().strftime('%y.%m.%d')

sor=["/etc/hosts","/etc/resolv.conf"]
a=len(sys.argv)
for i in range(1,a):
    sor.append(sys.argv[i])

print(sor)

dest = os.path.expanduser("~/backup/")
if not os.path.exists(dest):
    os.makedirs(dest, exist_ok=True)
for file in sor:
    try:
        filename = os.path.basename(file)
        new_filename = f"{filename}-{today_date}"
        shutil.copy2(file, os.path.join(dest, new_filename))
    except Exception as e:
        fail= f'Failed to copy file {filename} due the {e}'
        print(fail)
        with open(os.path.join(dest, 'failure_log.txt'), 'a') as log_file:
            log_file.write(fail + '\n')
```

This scrip will backup you the files you choose with the date you do backup and create a directory called backup in your home directory.
Take the scrip and put it in a directory with version name i put him in directory called proup-0.0.1 

Install rpm.

```bash
$  sudo dnf install -y rpmdevtools rpmlint
$ rpmdev-setuptree
```

It will create for you a directory called rpmbuild and he look like this.

```
rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS
```

The BUILD directory  is where the temporary files are stored, moved around, etc.

The RPMS directory holds RPM packages built for different architectures and noarch if specified in .spec file or during the build.

The SOURCES directory, as the name implies, holds sources. This can be a simple script, a complex C project that needs to be compiled, a pre-compiled program, etc. Usually, the sources are compressed as .tar.gz or .tgz files.

The SPEC directory contains the .spec files. The .spec file defines how a package is built. More on that later.

The SRPMS directory holds the .src.rpm packages. A Source RPM package doesn't belong to an architecture or distribution. The actual .rpm package build is based on the .src.rpm package.

After we build the basic we going to tar the directory with the scrip to the rpmbuild.
Build the RPM

```bash
$ tar --create --file proup-0.0.1.tar.gz proup-0.0.1
$ mv proup-0.0.1.tar.gz rpmbuild/SOURCES
```

Are next step is to create a .spec file this file will show how to create him and where to put him.

```bash
$ cd rpmbuild
$ rpmdev-newspec proup
$ mv proup.spec SPECS
$ vim /SPECS/proup.spec
```

```bash
Name:           proup
Version:        0.0.1
Release:        1%{?dist}
Summary:        Backup script for system files
License:        GPLv3+
Source0:        %{name}-%{version}.tar.gz
Requires:       python3 bash
BuildArch:      noarch

%description
Backup files in the home user /backup and atumatic backup /etc/hosts and /etc/rsolev.conf

%prep
%setup -q

%install
mkdir -p $RPM_BUILD_ROOT/%{_bindir}
install -m 755 %{name}.py $RPM_BUILD_ROOT/%{_bindir}

%files
%{_bindir}/%{name}.py

%changelog
* Tue Feb 20 2024 Gogs <gogs@itay.local>
-
```

If you have you own script modify it for you if not you can use the this .spec file.
Now after you have the spce file build the package.

```bash
$ rpmbuild -bb ~/rpmbuild/SPECS/proup.spec
```

We finish with the rpmbuild directory go back to your home directory.
In here we need to use createrepo. so we install createrepo.

```bash
$ cd
$ sudo dnf install createrepo
```

Now need to create directory for the repo, ther you will sent the rpm package that we create and send him to ther.

```bash
$ mkdir myrepo
$ mv rpmbuild/RPMS/noarch/your-rpm-package.noarch.rpm myrepo/
$ cd myrepo/
$ createrepo .
```

The last thing for this step is to do for him a configuration file for make him work.

```bash
$ cd /etc/yum.reos.d/
$ sudo mkdir myrepo.repo
$ sudo vim myrepo.repo
```

This code is the basic for configuration file for the repo and we sent him to the directory you move the rpm package.

```bash
[myrepo]
name=myrepo
baseurl=file:///home/your-user/myrepo
enabled=1
gpgcheck=0
```

Now you can install your package.

```bash
$ sudo dnf install proup
```

For make it public in inside you comany do the next step, install nginx.

```bash
$ sudo dnf install nginx
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```

Go to /usr/share/nginx/html and create the directory repo.

```bash
$ cd /usr/share/nginx/html
$ sudo mkdir repo
$ sudo chmod -R nginx:nginx ./repo
$ mv ~/myrepo/rpmbuild/RPMS/noarch/your-rpm-package.noarch.rpm repo/
$ createrepo repo/
```

For make it work we need to add permeant http to the firewall.

```bash
$ sudo firewall-cmd --permeant --add-service=http
$ sudo firewall-cmd --relaod
```

Now add new file in /etc/nginx/conf.d

```bash
$ cd /etc/nginx/conf.d
$ sudo mkdir repo.conf
$ sudo vim repo.conf
```

Put that scrip in repo.conf.

```bash
server {
        listen 80;
        server_name your-ip;

        location /repo {
                alias /usr/share/nginx/html/repo;
                autoindex on;
        }
}
```

Last thing, go to onfiguration file for the repo and modify it to be correctly.

```bash
$ sudo vim /etc/yum.reos.d/myrepo.repo
```

Modify like that.

```bash
[myrepo]
name=myrepo
baseurl=http://your-user/repo
enabled=1
gpgcheck=0
```

Do restart to nginx and this is ready.

```bash
$ sudo systemctl restart nginx
```

## Summery

In this post we create rpm package in your own deigan commend and we've made it available to those on the same network who can download as well.
