---
title: Archive &amp; Upgrade Python Packages Automatically
slug: upgrade-all-pip-python-packages-store-an-archive

publish_timestamp: Oct. 4, 2017
url: https://www.codingforentrepreneurs.com/blog/upgrade-all-pip-python-packages-store-an-archive/

---


<div class='alert alert-warning'>This guide is no longer maintained. Please consider <a href='https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python'>pipenv</a> instead.</div>

The code below is a script to update your virtualenv's python-install packages. It also creates a requirements history for your packages for future reference if you need it. 

Learn how to build it by watching the video [below](#watch) or [on youtube](https://youtu.be/nkAhOKoQ6h4).

### Using the [CFE CLI](https://www.codingforentrepreneurs.com/blog/cfe-cli/)
```
(venv) $ pip install cfe --upgrade
(venv) $ cfe --upgrade  
```
The CFE CLI `--upgrade` flag will automatically upgrade your outdated python packages, update your requirements.txt, and create an archive of your past package requirements. 


### The Code
```
import datetime
import os
import pip
from subprocess import call

# joincfe.com/blog/

def get_outdated():
    list_command = pip.commands.list.ListCommand()
    options, args = list_command.parse_args([])
    packages = pip.utils.get_installed_distributions()
    return list_command.get_outdated(packages, options)


def upgrade_oudated(all_pkgs=False):
    packages_upgraded = []
    if all_pkgs:
        to_upgrade = get_packages(upgrade=True)
    else:
        to_upgrade = get_outdated()
    for dist in to_upgrade:
        call("pip install --upgrade {pack_name}".format(pack_name=dist.project_name), shell=True)
        packages_upgraded.append(dist.project_name)

    if len(packages_upgraded) > 0:
        print("Upgraded the folllowing pacakges:\n")
        for pkg in packages_upgraded:
            print(" - " + str(pkg))
        print("\n")

# upgrade_oudated()


def set_archive_filepath(requirements_path, filepath=None, item=0):
    if filepath is None:
        base_filepath = os.path.join(requirements_path, "archive.txt")
    else:
        if item >= 0:
            item += 1
        today = str(datetime.date.today())
        base_filepath = os.path.join(requirements_path, "archive__%s__%i.txt" %(today, item))
    if os.path.exists(base_filepath):
        return set_archive_filepath(requirements_path, filepath=base_filepath, item=item)
    return base_filepath



def save_archive(archive_list):
    requirements_path = os.path.join(os.getcwd(), "requirements")
    if not os.path.exists(requirements_path):
        os.mkdir(requirements_path)
    #filepath = os.path.join(requirements_path, "requirements-archive.txt")
    filepath = set_archive_filepath(requirements_path)
    with open(filepath, "w+") as archive:
        for rq in sorted(archive_list, key=str.lower):
            archive.write(str(rq) + "\n")
    print("Archive done")

def make_archive():
    print("Making archive...")
    current = get_packages()
    save_archive(current)



def get_packages(upgrade=False):
    print("Getting packages...")
    current = []
    for dist in pip.get_installed_distributions():
        if upgrade:
            current.append(dist)
        else:
            current.append(str(dist.as_requirement()))
    return current



def get_requirements_location(next_to=None):
    if next_to is not None:
        location = None
        for root, dirs, files in os.walk(os.getcwd()):
            for file in files:
                if str(file) == str(next_to):
                    location = os.path.join(root, "requirements.txt")
                    return location
    return None


def save_requirements(next_to='manage.py'):
    packages  = get_packages()
    print("Saving requirements.txt")
    location = get_requirements_location(next_to=next_to) # os.path.join(os.getcwd(), "requirements.txt")
    if location is None:
        location = os.path.join(os.getcwd(), "requirements.txt")
    with open(location, "w") as f:
        for pkg in sorted(packages, key=str.lower):
            f.write(str(pkg) + "\n")




make_archive()
upgrade_oudated(all_pkgs=False)
save_requirements()

```

### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/nkAhOKoQ6h4?rel=0" frameborder="0" allowfullscreen></iframe>
