# Installation

We will be using Ubuntu 22.04 LTS in this guide.

### 1. Add the ntop repository into ubuntu

Inside a sudo shell:

```
apt-get install software-properties-common wget 
add-apt-repository universe
wget https://packages.ntop.org/apt-stable/22.04/all/apt-ntop-stable.deb 
apt install ./apt-ntop-stable.deb 
```

### 2. Install ntop

```
apt-get clean all 
apt-get update 
apt-get install ntopng
```

## References

* https://www.ntop.org/guides/ntopng/what_is_ntopng.html#installing-on-linux
* https://packages.ntop.org/apt-stable/