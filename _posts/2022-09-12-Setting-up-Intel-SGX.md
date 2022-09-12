---
title: Setting up Intel SGX on Ubuntu
date: 2022-09-12 18:00:00 +0200
categories: [Sysadmin, Intel SGX]
tags: [intel, sgx, cli, bash, zsh]     # TAG names should always be lowercase
toc: true
---

[Intel's Software Guard Extension](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) is a CPU instruction set that allows you to run sensitive code into an encrypted part of memory called an _enclave_.
This enclave is completely isolated from the rest of running processes and cannot (in theory at least) be accessed by any other process that the one that created it, even by the OS itself.
If I find enough time, I may write a post about how SGX works from a developper's point of view.

Anyway: using SGX implies a few addons to be installed on your computer.

## First: is your computer SGX-capable?

There was a time where almost any Intel Core i7 CPU was SGX-capable, but in 2021 Intel shifted away from desktop-class CPU and reserved SGX to server-class CPUs. That made blu-ray lovers (and me btw) very angry.

So to know if your CPU can make use of SGX enclaves, you can use the great [test-sgx](https://github.com/ayeks/SGX-hardware) program. Basically, this program will retrieve some CPU information and read flags from it, telling you if your CPU supports SGX or not.

It might be necessary to enable it from the BIOS

In all cases: [follow the instructions provided in the repo's README](https://github.com/ayeks/SGX-hardware#test-sgx)

## If you only need the driver to run an SGX app

If you don't want to dev using SGX and only need to run an SGX-enabled app, then you only need the driver. [Scone](https://scontain.com/index.html?lang=en), which is a secure container environment based on SGX, created a nice script that make the installation of this driver as easy as this one-liner:

```bash
curl -fssl https://raw.githubusercontent.com/SconeDocs/SH/master/install_sgx_driver.sh | bash
```

As always, feel free to review the [installer's code](curl -fssl https://raw.githubusercontent.com/SconeDocs/SH/master/install_sgx_driver.sh) before executing it.

## If you want to dev with SGX: the full setup

Ok the driver ain't enough, and you want the full setup? Here is how to do it.

First, install dependencies:

```bash
apt update
apt upgrade -y
apt install -y curl lsb-release build-essential ocaml ocamlbuild automake autoconf libtool wget libssl-dev git cmake perl libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev debhelper cmake reprepro unzip
```

Nothing fancy here. Next, we're going to retrieve some OS values to be used later on:

```bash
# get os name (18.10, 20.04...)
version=$(lsb_release -sr)

# get os codename (trusty, focal...)
codename=$(lsb_release -sc)
```

Ok next, using these variables, we are going to create a few more. The first one is the latest Intel SGX SDK's version:

```bash
sdk_version=$(git ls-remote --refs --tags https://github.com/intel/linux-sgx.git | cut -d '/' -f 3 | sort --version-sort | egrep "^sgx_[0-9]\.[0-9]+$" | tail -1)
sdk_version_short=$(echo $sdk_version | cut -c 5-)
```

Then, from that we are going to create download URLs for the SDK's repo, the driver and the SDK's binary.

```bash
# create repo URL
repo_url=https://download.01.org/intel-sgx/sgx-linux/${sdk_version_short}/distro/ubuntu${version}-server/

# retrieve filenames from repo URL
driver_filename=$(curl -s $repo_url | grep -Po 'sgx_linux_x64_driver_[0-9a-z\._]+\.bin' | uniq | sort --reverse | head -1)
sdk_filename=$(curl -s $repo_url | grep -Po 'sgx_linux_x64_sdk_[0-9a-z\._]+\.bin' | uniq)
```

Now we're ready to go: let's create a temporary directory to download everything we need into:

```bash
tmp_dir=$(mktemp -d)
cd $tmp_dir
```

Let's clone the SDK's repo and compile/install SGX tools:

```bash
# clone SDK repo
git clone https://github.com/intel/linux-sgx.git

# compile SDK's toolset and copy it to system dir
cd linux-sgx && make preparation
cp external/toolset/ubuntu${version}/* /usr/local/bin

# back to temp dir
cd $tmp_dir
```

Next, let's download SDK and driver binaries to install them:

```bash
wget $repo_url$driver_filename
wget $repo_url$sdk_filename

chmod +x sgx_linux_x64_*

mkdir -p /opt/intel/

./sgx_linux_x64_sdk*.bin << 'EOF'
no
/opt/intel
EOF

./sgx_linux_x64_driver*.bin
```

Ok, almost there. Now we need to install a few more SGX services like AESM:

```bash
echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu ${codename} main" | tee /etc/apt/sources.list.d/intel-sgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add -

apt update
apt install -y libsgx-launch libsgx-urts libsgx-epid libsgx-quote-ex libsgx-dcap-ql libsgx-uae-service
```

Finally, cleanup the temp directory:

```bash
rm -rf $tmp_dir
```

Yay, we're done!

For convenience, here is the complete script you can save to a file and sudo-execute:

```bash
# install dependencies
apt update
apt upgrade -y
apt install -y curl lsb-release build-essential ocaml ocamlbuild automake autoconf libtool wget libssl-dev git cmake perl libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev debhelper cmake reprepro unzip

# get os name (18.10, 20.04...)
version=$(lsb_release -sr)

# get os codename (trusty, focal...)
codename=$(lsb_release -sc)

# retrieve latest version of SGX SDK
sdk_version=$(git ls-remote --refs --tags https://github.com/intel/linux-sgx.git | cut -d '/' -f 3 | sort --version-sort | egrep "^sgx_[0-9]\.[0-9]+$" | tail -1)
sdk_version_short=$(echo $sdk_version | cut -c 5-)

# create repo URL
repo_url=https://download.01.org/intel-sgx/sgx-linux/${sdk_version_short}/distro/ubuntu${version}-server/

# retrieve filenames from repo URL
driver_filename=$(curl -s $repo_url | grep -Po 'sgx_linux_x64_driver_[0-9a-z\._]+\.bin' | uniq | sort --reverse | head -1)
sdk_filename=$(curl -s $repo_url | grep -Po 'sgx_linux_x64_sdk_[0-9a-z\._]+\.bin' | uniq)

# create temp repo and move to it
tmp_dir=$(mktemp -d)
cd $tmp_dir

# clone SDK repo
git clone https://github.com/intel/linux-sgx.git

# compile SDK's toolset and copy it to system dir
cd linux-sgx && make preparation
cp external/toolset/ubuntu${version}/* /usr/local/bin

# back to temp dir
cd $tmp_dir

# download driver and SDK
wget $repo_url$driver_filename
wget $repo_url$sdk_filename

# mark both as executable
chmod +x sgx_linux_x64_*

# create directory for SDK
mkdir -p /opt/intel/

# on questions: "no" then "/opt/intel"
./sgx_linux_x64_sdk*.bin << 'EOF'
no
/opt/intel
EOF

# add SDK env vars to local shell environment
if [[ $SHELL = *zsh ]]; then
  echo "source /opt/intel/sgxsdk/environment" >> ~/.zshrc
else
  echo "source /opt/intel/sgxsdk/environment" >> ~/.bashrc
fi

# finally, install driver
./sgx_linux_x64_driver*.bin

# setup PSW repo
echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu ${codename} main" | tee /etc/apt/sources.list.d/intel-sgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add -

# install PSW/AESM daemon
apt update
apt install -y libsgx-launch libsgx-urts libsgx-epid libsgx-quote-ex libsgx-dcap-ql libsgx-uae-service

rm -rf $tmp_dir
```

## One more thing

Ok, having a bash script that automates everything is fine, but what about an ansible script? You guessed it, I've got you covered. [Here is a complete script](https://gitlab.com/X99/sgxsetup) that make all of the above and also sets up Gramine if you want to.

Refer to the project's README for more information.

Have fun!