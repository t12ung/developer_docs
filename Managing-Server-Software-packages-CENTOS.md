[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

<img src="https://upload.wikimedia.org/wikipedia/commons/b/bf/Centos-logo-light.svg" height=50 alt="CentOS Logo" />

# This Guide is for CentOS Linux Operating System

Packages can be installed using RPM or YUM. YUM is preferred and easier because it handles package dependencies.
A useful resource for information about available packages is https://centos.pkgs.org/

Before starting, we should verify what repositories are available to us.
```shell
# yum repolist
```

If not already available, we should add the following repositories for Enterprise RHEL release packages (Red Hat
Enterprise Linux):

* EPEL (Extra Packages for Enterprise Linux)
* [REMI-Safe](https://blog.remirepo.net/post/2015/06/02/New-remi-safe-repository-for-EL-7) (French guy named Remi -
contains packages that are now deprecated)
* ELRepo (Enterprise Linux Repository)

The REMI repository consists of multiple repos, and by default they are all disabled except for remi-safe. It's a good
idea to enable the remi-phpxx repo for the specific PHP version running on the server as they will typically have more
recent package updates compared to CentOS base repositories. To list all available installed repositories and
enable/disable them:
```shell
# yum repolist all
# yum-config-manager --<enable|disable> remi-phpxx
```

> It is recommended to try installing default "single" version packages if possible, as this will require less
> maintenance and additional installation steps. Default packages usually follow the naming convention of `php-xxx`.
> You can check whether a package installs to default locations or not using the `repoquery` tool (`yum-utils` package).
> See the Installation Notes below for more details.

We can _**search**_ for a package by string from all available repositories. If there are a lot of repositories enabled,
it may be more useful to restrict searches to only repositories with Enterprise packages (if the package we want is not
found, only then search all available repositories, omitting the optional parameters relating to repo).
```shell
yum search --disablerepo=* --enablerepo=epel <string>
```

If anything is found, a list is returned showing full package names with a 1-line description. The list may contain a
few different versions of what we're looking for so it's a good idea to use the info command get more information about
packages. The string parameter for this command tries to match by _**exact**_ package name, so it's more useful to
provide wildcards for multiple matches.
```shell
yum info --disablerepo=* --enablerepo=epel *<string>*
```

To install a package, and automatically try installing any dependency packages. It will ask show a confirmation,
detailing what package and dependencies will be installed:
```shell
yum install <package_name>
```

Some packages may require a specific version of a package (older one to currently installed) and fail to install the
original package you requested as a result. Check _**all**_ the error messages to see if this is the case - if so, we
can try to force installation and ignore the requirement for older package versions.


## Keeping Packages Up-to-date

By enabling the relevant `remi-phpxx` repo, you can easily keep your packages up to date simply by running the command
(`httpd` would normally need to be restarted for changes to be applied):
```shell
yum update
```

## Installation Notes

The standard installation locations are:

> `/etc/php.d/` (ini)
> `/usr/lib64/php/modules` (default so module)

Non-standard php packages are installed in different locations (ini and so files). To enable this extensions to load,
create symbolic links for the files in the appropriate default locations (`ln -s <source> <destination>`). Non-standard
installation paths

>`/opt/<vendor>/<package-name>`

To find out where the files of a package are installed to, run the command `repoquery -l <package-name>`.
