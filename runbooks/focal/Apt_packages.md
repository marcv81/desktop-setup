# Apt packages

## Remove sources

Remove the sources from `/etc/apt/sources.list.d/`.

List and remove any unnecessary keys. The key ID is the last 2 elements of the fingerprint (as of Ubuntu 18.04).

    $ apt-key list
    /etc/apt/trusted.gpg
    --------------------
    pub   rsa4096 2017-02-22 [SCEA]
          9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
    sub   rsa4096 2017-02-22 [S]

    $ sudo apt-key del 0EBFCD88

## Purge removed packages

List the removed packages to purge.

    dpkg -l | awk '/^rc/{print $2}'

Purge the removed packages.

    dpkg -l | awk '/^rc/{print $2}' | sudo xargs apt-get purge -y

## Explain

`aptitude` and `apt-rdepends` can attempt to explain why a package is installed.

    apt-rdepends --reverse --state-follow=Installed --state-show=Installed --follow=Depends,Recommends <package_name>
    aptitude why --show-summary <package_name>
