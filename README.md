# Union Filesystem Lab
```
# Create some directories:
mkdir -p ~/jotb/{image,c1_container,c1_work,container1}
```

```
# cd into the main directory:
cd ~/jotb
```

```
# Set the architecture:
arch="$(arch)"
arch="${arch/arm64/aarch64}"
arch="${arch/amd64/x86_64}"
```

```
# wget Alpine:
wget -qO- https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/"${arch}"/alpine-minirootfs-3.19.1-"${arch}".tar.gz \
     | tar xvz -C ~/jotb/image
```

```
# Create an overlay filesystem
sudo mount -t overlay \
  -o lowerdir=image,upperdir=c1_container,workdir=c1_work \
     overlay container1
```

```
# List files in different layers:
ls -l ~/jotb/image
ls -l ~/jotb/c1_container
ls -l ~/jotb/container1
```

```
# Create files in different layers:
echo "AAA" > ~/jotb/image/AAA
echo "BBB" > ~/jotb/c1_container/BBB
```

```
# Change a file in the container layer:
echo "aaa" > ~/jotb/c1_container/AAA
```

```
# Check the contents of file AAA in each layer:
cat ~/jotb/image/AAA
cat ~/jotb/c1_container/AAA
cat ~/jotb/container1/AAA
```

```
# Update PATH (just in case!)
PATH=$PATH:/bin
```

```
# "Unshare" namespaces and chroot to the "container":
unshare --fork --uts --pid --mount \
        --user --ipc --net \
        --map-root-user \
        chroot ~/jotb/container1 \
        /bin/sh
```

```
# Check the hostname:
hostname
```

```
# Change the hostname
hostname alpine
hostname
```

```
# Explore the "container"
cat /etc/os-release
```

```
uname -a
```

```
whoami
```

```
echo $$
```

```
pwd
```

```
# What files are here?
ls -l /
cat AAA
cat BBB
```

```
# Add a group and user:
addgroup -g 12345 jotb_g
adduser -H -S -u 54321 jotb_u -G jotb_g
```

# Containers are Magic!
```
# Prepare directories for a second container
mkdir -p ~/jotb/{c2_container,c2_work,container2}
cd ~/jotb
```

```
# Create a second overlay filesystem
sudo mount -t overlay \
  -o lowerdir=image,upperdir=c2_container,workdir=c2_work \
     overlay container2
```

```
# Set the environment (just in case!)
PATH=$PATH:/bin
```

```
# "Run" the "container"
unshare --fork --uts --pid --mount \
        --user --ipc --net \
        --map-root-user \
        chroot ~/jotb/container2 \
        /bin/sh
```

```
# What files are here?
ls -l /
cat AAA
cat BBB
```

```
# Manipulate files within the container:
echo "111" > /AAA
echo "222" > /BBB
```

```
# Check the contents of files from the host:
cat ~/jotb/image/AAA
cat ~/jotb/c2_container/AAA
cat ~/jotb/c2_container/BBB
```

```
# Exit:
exit
```

```
# Restart the container:
unshare --fork --uts --pid --mount \
        --user --ipc --net \
        --map-root-user \
        chroot ~/jotb/container2 \
        /bin/sh
```
# Container Properties
```
# Exit the container
exit
```

```
# "Drop" the container:
sudo umount container2
rm -fr ~/jotb/container2
rm -fr ~/jotb/c2_container
rm -fr ~/jotb/c2_work
```
# Building Images
```
# Create some directories:
mkdir -p ~/jotb/{image,c3_container,c3_work,container3}
```

```
# cd into the main directory:
cd ~/jotb
```

```
# Create an overlay filesystem
sudo mount -t overlay \
  -o lowerdir=container1,upperdir=c3_container,workdir=c3_work \
     overlay container3
```

```
# Update PATH (just in case!)
PATH=$PATH:/bin
```

```
# "Unshare" namespaces and chroot to the "container":
unshare --fork --uts --pid --mount \
        --user --ipc --net \
        --map-root-user \
        chroot ~/jotb/container3 \
        /bin/sh
```

```
# Cleanup
sudo umount container3
sudo umount container1
cd ~
rm -fr ~/jotb
```
