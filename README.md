# Union Filesystem Lab
```
# Create some directories:
mkdir -p ~/workshop/{image,con1_upper,con1_work,container1}
```

```
# cd into the main directory:
cd ~/workshop
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
     | tar xvz -C ~/workshop/image
```

```
# Create an overlay filesystem
sudo mount -t overlay \
  -o lowerdir=image,upperdir=con1_upper,workdir=con1_work \
     overlay container1
```

```
# List files in different layers:
ls -l ~/workshop/image
ls -l ~/workshop/con1_upper
ls -l ~/workshop/container1
```

```
# Create files in different layers:
echo "AAA" > ~/workshop/image/AAA
echo "BBB" > ~/workshop/con1_upper/BBB
```

```
# Change a file in the container layer:
echo "aaa" > ~/workshop/con1_upper/AAA
```

```
# Check the contents of file AAA in each layer:
cat ~/workshop/image/AAA
cat ~/workshop/con1_upper/AAA
cat ~/workshop/container1/AAA

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
        chroot ~/workshop/container1 \
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
addgroup -g 12345 workshop_g
adduser -H -S -u 54321 workshop_u -G workshop_g
```

# Containers are Magic!
```
# Prepare directories for a second container
mkdir -p ~/workshop/{con2_upper,con2_work,container2}
cd ~/workshop
```

```
# Create a second overlay filesystem
sudo mount -t overlay \
  -o lowerdir=image,upperdir=con2_upper,workdir=con2_work \
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
        chroot ~/workshop/container2 \
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
cat ~/workshop/image/AAA
cat ~/workshop/con2_upper/AAA
cat ~/workshop/con2_upper/BBB
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
        chroot ~/workshop/container2 \
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
rm -fr ~/workshop/container2
rm -fr ~/workshop/con2_upper
rm -fr ~/workshop/con2_work
```
# Building Images
```
# Create some directories:
mkdir -p ~/workshop/{image,con3_upper,con3_work,container3}
```

```
# cd into the main directory:
cd ~/workshop
```

```
# Create an overlay filesystem
sudo mount -t overlay \
  -o lowerdir=container1,upperdir=con3_upper,workdir=con3_work \
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
        chroot ~/workshop/container3 \
        /bin/sh
```

```
# Cleanup
sudo umount container3
sudo umount container1
cd ~
rm -fr ~/workshop
```
