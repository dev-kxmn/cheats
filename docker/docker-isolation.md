# Docker Isolation

To create a simple solution for development and production we can plan be
forehand the userids to be used:

* Id of host namespaced root user
* Id of the container user who handles data (including code)
* Id of the host developer

## Before Configuration

	----- on host -----   --- on container ---
	USERID   USERNAME       USERID   USERNAME
	-------------------   --------------------
	myuser   1000      -->    1000     myuser
	root        0      -->       0       root

We can see that the root on container is the same as the root in the host
that can be exploited and have security holes. But we can workaround this
configuring container and adapting our development area to be consistent.

	----- on host -----   --- on container ---
	USERID   USERNAME       USERID   USERNAME
	-------------------   --------------------
	subroot  100000    -->       0       root
	myuser   101000    -->    1000     myuser
	root     0         -->    (unnaccessible)

##How To

If you plan to use your current user as development, you will need to
logout and, or login as other user or goes to terminal dark as root and
run:

```
# Creating the used as root inside containers
groupadd -gid 100000 subroot
useradd -uid 100000 --gid subroot --basedir / --home-dir /home/subroot

# Change your user to be used 
usermod -o -u 101000 $YOUR_DEV_USER
groupmod -o g 101000 $USER_DEV_USER

# Change host configuration subuid and subgid
echo 'subroot:100000:65536' > /etc/subuid
echo 'subroot:100000:65536' > /etc/subgid

# Change the docker start to consider our user
/etc/init.d/docker stop
cat <<'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --userns-remap=subroot
EOF

# Reload systemv init scripts
/etc/init.d/systemctl daemon-reload

# Restart docker
/etc/init.d/docker start
```

## Next steps

* `sudo docker images` will bring a empty list. This is now, the images
  will build in the folder `/var/lib/docker/100000.100000` which is owned
  by the userid/groupid 100000 (our subroot)
  So rebuild your images.

* Make necessary changings on your *dockerfile* or container script
  install to respect the owner (read some cases explained below)

* Change to 101000 the group and user of the folders that will be mounted
  on container at `docker run`

This done, your should change to 101000 the owner user and group of the
folders that will be mounted on docker container.

##Adapting images

### Alternative 1: change image userid

Each container has different needs... the basic is, if you run a webserver
(ex. Apache or Nginx) the username, on Debian, is www-data. So we get the
id number inside the container:

```
id www-data
# uid=33(www-data) gid=33(www-data) grupos=33(www-data)

id mysql
# uid=101(mysql) gid=101(mysql) grupos=101(mysql)
```

Then we adapt our dockerfile or install script to change the numeric id:

```
usermod -u 1000 www-data
groupmod -g 1000 www-data
```

### Alternative 2: change service configuration

This is the trickiest part as each service can have its own way of
configuration, and each Linux flavor has its own standards. The only
standard is that, inside dockerfile you will need to create the user with
id 1000 and the name you want... lets think... `data`

```
groupadd --gid 1000 data
useradd --uid 1000  --gid 1000 --base-dir / --home-dir /home/data data
```

See as tricker it could be: creating a user you will need to define the
home dir that could break the service and create confused hours trying to
figure out how to correct. The alternative 1 is the best way.
