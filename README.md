BalenaCLI Masterclass
=====================

**Masterclass Type:** Core
**Maximum Expected Time To Complete:** 60 minutes

# Introduction

The balena Command Line Interface (balenaCLI) utility consists of a number
of commands that allow a user to develop, deploy and manage balena applications
and devices, as well as manage device configurations (including environment
variables) and balenaOS images.

Almost everything that can be achieved via the balenaCloud dashboard can also be
achieved via the balenaCLI.

In this masterclass, you will learn how to:

* Login to your account
* Push application code to a balena Application
* Deploy locally built code to a balena Application
* SSH into a balena device
* Push and build applications on a device on the local network for fast
    development and prototyping
* Use private Docker registries for base images and services
* Create secret files and build arguments for building service images

If you have any questions about this masterclass as you proceed through it,
or would like clarifications on any of the topics raised here, please do
raise an issue as on the repository this file is container in, or contact
us on the [balena forums](https://forums.balena.io/) where we'll be
delighted to answer your questions.

The location of the repository that contains this masterclass and all associated
code is
[https://github.com/balena-io-projects/balena-cli-masterclass](https://github.com/balena-io-projects/balena-cli-masterclass).

# Hardware and Software Requirements

It is assumed that the reader has access to the following:

* A locally cloned copy of this repository
	[Balena CLI Masterclass](https://github.com/balena-io-projects/balena-cli-masterclass).
	Either:
	* `git clone https://github.com/balena-io-projects/balena-cli-masterclass.git`
	* Download ZIP file (from 'Clone or download'->'Download ZIP') and then
		unzip it to a suitable directory
* A balena supported device, such as a [balenaFin 1.1](https://store.balena.io/collections/developer-kit/products/balenafin-v1-1-0-developer-kit),
	[Raspberry Pi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)
	or [Intel NUC](https://www.intel.co.uk/content/www/uk/en/products/boards-kits/nuc.html). If you don't have a device, you can emulate an Intel NUC by
	installing VirtualBox and following [this guide](https://www.balena.io/blog/no-hardware-use-virtualbox/)
* A suitable text editor for developing code on your development platform (eg.
    [Visual Code](https://code.visualstudio.com/))
* A suitable shell environment for command execution (such as `bash`)
* A [balenaCloud](https://www.balena.io/) account
* A local installation of [Docker](https://docs.docker.com/v17.09/engine/installation/)
	as well as a familiarity with [Dockerfiles](https://docs.docker.com/engine/reference/builder/)
* Should you wish to install via `npm`, a [NodeJS](https://nodejs.org/en/)
	installation, including [NPM](https://www.npmjs.com/get-npm) is required.
	The use of [`nvm`](https://github.com/nvm-sh/nvm)
	is recommended, which allows you to alter the version of Node/NPM being used
	per-user, and also removes the need to install global dependencies using
	`sudo`

# Exercises

All of the following exercises assume that you are running the balenaCLI from
a suitable Unix based shell. The exercises include commands which can be run
in such a shell, and are represented by a line prefixed with `$`. Information
returned from execution of a command may be appended under the line to show
what might be returned. For example:

```
$ balena version
11.9.3
```

## 1. Installation and Authentication

### 1.1 Installation
First we need to install balenaCLI. The easiest way to achieve this is to use
the installers for your OS from the
[balena CLI releases page](https://github.com/balena-io/balena-cli/releases).
Choose the installer for your OS, download it and follow the instructions.
Note that there is not currently an installer for Linux, but you
can download the standalone binary and then move it to a relevant location.

The alternative way to install it is via `npm` on a system running NodeJS.
Open a terminal on your development machine and run the following command:
```
$ npm --global install balena-cli
```
This will install the balenaCLI globally and allow you to run it in a terminal
via `balena <command>`. Note that, depending on how you've installed NodeJS
and NPM, you may need to prefix this command with `sudo`. Also, if you get an
error such as `EACCES: permission denied`, add param `--unsafe-perm` right
after `--global`

### 1.2 Authentication

To use balenaCLI, you need to login to a balenaCloud account. If you don't
have one, you can use the dashboard
[here](https://dashboard.balena-cloud.com/signup) or sign up with the
login command by selecting `I don't have a balena account!`. Either way,
login via the terminal:
```
$ balena login
 _            _
| |__   __ _ | |  ____  _ __    __ _
| '_ \ / _` || | / __ \| '_ \  / _` |
| |_) | (_) || ||  ___/| | | || (_) |
|_.__/ \__,_||_| \____/|_| |_| \__,_|


Logging in to balena-cloud.com
? How would you like to login? (Use arrow keys)
❯ Web authorization (recommended)
  Credentials
  Authentication token
  I don't have a balena account!
```

You will be asked how you wish to authenticate with balenaCloud. The
easiest method is that of 'Web authorization' which will bring up a browser
window (and ask you to first login to balenaCloud if you have not) and ask you
to confirm you wish to login.

Other authentication methods include using your username and password
credentials or authentication token. Authentication tokens come in two types,
API tokens and JSON Web Token (JWT) session tokens. Whilst API tokens do not
expire, JWT session tokens do after 7 days.

Once logged in, a [JWT](https://jwt.io/) session token will be saved in your
home directory (`~/.balena/token`). Be aware that the lifetime of a balena
JWT is limited to seven days, after which time reauthentication will be
required.

## 2. Creating an Application and Provisioning a Device

### 2.1 Creating an Application

Applications can be created via the dashboard or via the balenaCLI. We're going
to create a new application via balenaCLI called `cliApp`. Run the following
command:
```
$ balena app create cliApp
```
This will ask you which device type you wish to create the application for.
You can scroll up and down this list using the arrow keys. For now, exit
the command by hitting `Ctrl-C`, as there's another, non-interactive way to
do this which we'll use instead. Type:
```
$ balena devices supported
```
to see a list of all supported device types by balenaCloud. For the rest of
this masterclass we're going to assume you're using a balenaFin, but you can
just as easily use any supported balena device.
We'll pass the balenaFin device type (`fincm3`) to the application creation
command directly:
```
$ balena app create cliApp --type fincm3
Application created: cliApp (fincm3, id 1234567)
```
As can be seen, this will return the name of the application, its type
(`fincm3`) and its unique ID. If you're using a different device
type, pass the appropriate device type to the `app create` command instead.

Non-interactive commands are useful when you need to script actions via
balenaCLI for a shellscript (although balena also includes HTTPS endpoints and
SDKs which can be used for this purpose).

You can list the applications currently owned by (or shared with) your account
by typing:
```
$ balena apps
ID      APP NAME         DEVICE TYPE      ONLINE DEVICES DEVICE COUNT
1234567 cliApp           fincm3           0              0
```

### 2.2 Provisioning a Device

You can now provision your balenaFin by downloading a provisioning image from
the balenaCloud dashboard. Be sure to download a development image, as we'll
be utilising its features later.

Once the provisioning image is downloaded, connect your balenaFin to your development machine and run
[Etcher](https://www.balena.io/etcher/)
to
[flash it](https://www.balena.io/fin/1.1/docs/getting-started/#Flashing-the-OS).

Once the image has been flashed to the balenaFin it will register itself and
connect to the balenaCloud VPN, showing up in the dashboard and being viewable
using balenaCLI:
```
$ balena devices
ID      UUID    DEVICE NAME      DEVICE TYPE  APPLICATION NAME STATUS IS ONLINE SUPERVISOR VERSION OS VERSION           DASHBOARD URL
7654321 1234567 restless-glade   fincm3       cliApp                  true                                              https://dashboard.balena-cloud.com/devices/12345678901234567890123456789012/summary
```

You can get detailed information on a device by using its Universally Unique
Identifier (UUID), for example:
```
$ balena device 1234567
== RESTLESS GLADE
ID:                 7654321
DEVICE TYPE:        fincm3
STATUS:             idle
IS ONLINE:          true
IP ADDRESS:         192.168.1.173
APPLICATION NAME:   cliApp
UUID:               12345678901234567890123456789012
SUPERVISOR VERSION: 9.15.7
IS WEB ACCESSIBLE:  false
OS VERSION:         balenaOS 2.38.0+rev1
DASHBOARD URL:      https://dashboard.balena-cloud.com/devices/12345678901234567890123456789012/summary
```

UUIDs can either be used in their shortened version (as above) or in their
long version (for example, the `DASHBOARD URL` field in the output above
shows the entire UUID for the device).

Be aware that there are ways to download, configure and provision an application
image via balenaCLI, but as some extra work is required to create a provisioning
image (to allow greater flexibility) we'll go into that in the advanced
masterclass.

## 3. Pushing Code to a Device

Once an application has been created, we want to be able to push code to it.
There are a couple of way to do this, but the most common is that of using
`balena push`.

### 3.1 `balena push`

As this masterclass contains a Dockerfile and simple NodeJS source, we can
use a balenaCLI command to 'push' the code from the masterclass repository
to the balena builders.

Ensure that in your terminal, you have changed directory to the root of the
repository for this
[masterclass](https://github.com/balena-io-projects/balena-cli-masterclass)
and then type:
```
$ balena push cliApp
[Info]     Starting build for cliApp, user heds
[Info]     Dashboard link: https://dashboard.balena-cloud.com/apps/1234567/devices
[Info]     Building on arm03
[Info]     Pulling previous images for caching purposes...
[Success]  Successfully pulled cache images
[main]     Step 1/6 : FROM balenalib/fincm3-node:8
[main]      ---> 392c3f6339f7
[main]     Step 2/6 : WORKDIR /usr/src/app
[main]      ---> 7d7f394b0c9c
[main]     Step 3/6 : COPY package.json package-lock.json ./
[main]      ---> 9548fa0048b4
[main]     Removing intermediate container f79901843a6f
[main]     Step 4/6 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[main]      ---> Running in a99982fff18b
[main]     added 50 packages from 37 contributors and audited 126 packages in 2.439s
[main]     found 0 vulnerabilities
[main]     npm
[main]      WARN using --force I sure hope you know what you are doing.
[main]
[main]      ---> 495bbe5f5018
[main]     Removing intermediate container a99982fff18b
[main]     Step 5/6 : COPY src/ ./src/
[main]      ---> c51c598e9c35
[main]     Removing intermediate container 0f10b22606b5
[main]     Step 6/6 : CMD npm start
[main]      ---> Running in 5b0dfd22b549
[main]      ---> e2683253db67
[main]     Removing intermediate container 5b0dfd22b549
[main]     Successfully built e2683253db67
[Info]     Uploading images
[Success]  Successfully uploaded images
[Success]  Release successfully created!
[Info]     Release: 123446b90543220bd0c7f98b4b55a989 (id: 1234567)
[Info]     ┌─────────┬────────────┬────────────┐
[Info]     │ Service │ Image Size │ Build Time │
[Info]     ├─────────┼────────────┼────────────┤
[Info]     │ main    │ 208.43 MB  │ 7 seconds  │
[Info]     └─────────┴────────────┴────────────┘
[Info]     Build finished in 15 seconds
			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'
```

This will package up and then push the code from the local directory to the
balena builders, which examines any `Dockerfile`, `Dockerfile.template`,
`docker-compose.yml` or `package.json` file to determine how to build the code
and then create a Docker image that will be stored in the balenaCloud private
Docker repository.

The `push` command actually accepts a wide range of arguments, but this is the
basic (and most widely used) form of the command. You can see those arguments
and optional switches in the balena CLI documentation
[here](https://www.balena.io/docs/reference/cli/#push-applicationordevice).

As a balenaFin has been provisioned against the `cliApp` application, you'll
soon see that it starts to download the code we've just pushed (by observing
the dashboard). Eventually, the application image will be pulled and the
Supervisor on the device will start the service. You can look at the logs by
using the following balenaCLI command:
```
$ balena logs 1234567
[Logs]    [9/3/2019, 11:18:50 AM] Downloading image 'registry2.balena-cloud.com/v2/1234568b5d909bf95c2bbe6c1df150f3@sha256:75f4f5bfb6e610f2b7fe8d61e8a77979a4be3de344cad09fba4df5bba8da73a3'
[Logs]    [9/3/2019, 11:18:51 AM] Service exited 'main sha256:f9813f5cc0ee22ca166c93c2c72e2fb6f37dff6cb4add332e3f57e418de6b339'
[Logs]    [9/3/2019, 11:18:58 AM] Downloaded image 'registry2.balena-cloud.com/v2/1234568b5d909bf95c2bbe6c1df150f3@sha256:75f4f5bfb6e610f2b7fe8d61e8a77979a4be3de344cad09fba4df5bba8da73a3'
[Logs]    [9/3/2019, 11:18:58 AM] Killing service 'main sha256:f9813f5cc0ee22ca166c93c2c72e2fb6f37dff6cb4add332e3f57e418de6b339'
[Logs]    [9/3/2019, 11:18:58 AM] Killed service 'main sha256:f9813f5cc0ee22ca166c93c2c72e2fb6f37dff6cb4add332e3f57e418de6b339'
[Logs]    [9/3/2019, 11:18:58 AM] Deleting image 'registry2.balena-cloud.com/v2/12344fad6b184fb68e1f94f8fced204d@sha256:35428c9df601789509f2100a8e6a22c2a2f0257bb49308053cb598be82e67ce4'
[Logs]    [9/3/2019, 11:18:58 AM] Deleted image 'registry2.balena-cloud.com/v2/12344fad6b184fb68e1f94f8fced204d@sha256:35428c9df601789509f2100a8e6a22c2a2f0257bb49308053cb598be82e67ce4'
[Logs]    [9/3/2019, 11:18:59 AM] Installing service 'main sha256:e2683253db679de87adca3b3fa80ac6aa16b62be4329524c394d3ce0d5f96f65'
[Logs]    [9/3/2019, 11:19:00 AM] Installed service 'main sha256:e2683253db679de87adca3b3fa80ac6aa16b62be4329524c394d3ce0d5f96f65'
[Logs]    [9/3/2019, 11:19:00 AM] Starting service 'main sha256:e2683253db679de87adca3b3fa80ac6aa16b62be4329524c394d3ce0d5f96f65'
[Logs]    [9/3/2019, 11:19:01 AM] Started service 'main sha256:e2683253db679de87adca3b3fa80ac6aa16b62be4329524c394d3ce0d5f96f65'
[Logs]    [9/3/2019, 11:19:02 AM] [main]
[Logs]    [9/3/2019, 11:19:02 AM] [main] > balena-cli-masterclass@1.0.0 start /usr/src/app
[Logs]    [9/3/2019, 11:19:02 AM] [main] > node src/helloworld.js; sleep infinity
[Logs]    [9/3/2019, 11:19:02 AM] [main]
[Logs]    [9/3/2019, 11:19:03 AM] [main] Hello world!
```

### 3.2 `git push`

Traditionally, balenaCloud has exposed a private git repository endpoint that
allows you to use `git push` to push git stored code to the balena builders.

This is now considered a legacy method for pushing code to an application, and
if possible you should use `balena push` as it makes for a consistent workflow
and methodology. However, in order to understand how this works, the following
instructions will demonstrate how to push code via git.

The dashboard will display the command required to run in a directory containing
an already initialised git repository. Copy this command, ensure in your
terminal you change to the local cloned directory for the the source repository
for this
[masterclass](https://github.com/balena-io-projects/balena-cli-masterclass)
and the execute the command. It should look something like this:
```
$ git remote add balena myuser@git.balena-cloud.com:myuser/cliapp.git
```
This adds the balena origin to the repository. You should now be able to push
the code to balena's private git repository:
```
$ git push balena master
Counting objects: 11, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (11/11), 12.04 KiB | 2.01 MiB/s, done.
Total 11 (delta 0), reused 3 (delta 0)

[Info]     Starting build for cliapp, user heds
[Info]     Dashboard link: https://dashboard.balena-cloud.com/apps/1234567/devices
[Info]     Building on arm01
[Info]     Pulling previous images for caching purposes...
[Success]  Successfully pulled cache images
[main]     Step 1/6 : FROM balenalib/fincm3-node:8
[main]      ---> 392c3f6339f7
[main]     Step 2/6 : WORKDIR /usr/src/app
[main]     Using cache
[main]      ---> 7d7f394b0c9c
[main]     Step 3/6 : COPY package.json package-lock.json ./
[main]      ---> d4e08e51860f
[main]     Removing intermediate container e6d0ddf83498
[main]     Step 4/6 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[main]      ---> Running in 72b59e4ecd47
[main]     added 50 packages from 37 contributors and audited 126 packages in 2.465s
[main]     found 0 vulnerabilities
[main]     npm
[main]      WARN
[main]      using --force
[main]      I sure hope you know what you are doing.
[main]
[main]      ---> e2f81bc3ff91
[main]     Removing intermediate container 72b59e4ecd47
[main]     Step 5/6 : COPY src/ ./src/
[main]      ---> 21b7d6b7f231
[main]     Removing intermediate container 0658f893dadd
[main]     Step 6/6 : CMD npm start
[main]      ---> Running in b2dd4ccc8285
[main]      ---> 83babbbb57b4
[main]     Removing intermediate container b2dd4ccc8285
[main]     Successfully built 83babbbb57b4
[Info]     Uploading images
[Success]  Successfully uploaded images
[Success]  Release successfully created!
[Info]     Release: 0576485d6a2049e066c424e1bab957a16e7cb396 (id: 1234567)
[Info]     ┌─────────┬────────────┬────────────┐
[Info]     │ Service │ Image Size │ Build Time │
[Info]     ├─────────┼────────────┼────────────┤
[Info]     │ main    │ 208.39 MB  │ 7 seconds  │
[Info]     └─────────┴────────────┴────────────┘
[Info]     Build finished in 15 seconds
			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'

To git.balena-cloud.com:heds/cliapp.git
 * [new branch]      initial-pr -> master
```

This will store the code from the local repository into the private copy
stored by balena.

Just like `balena push`, once the code has been built it will be downloaded
by the device and then run.

**Important Note:** There is absolutely no guarantee that the private repository
will be available for pulling code from, and as such it should never be used
as an application's source repository origin. It is intended to be a temporary
repository used only for pushing to the builders.

## 4. SSHing into a Device

Once a device has been provisioned, it can be accessed by SSHing into it via
the balenaCloud VPN. To do this, you'll need the UUID of the device you want to
SSH into (remember you can see all your devices by running `balena devices`):
```
$ balena ssh 1234567
=============================================================
    Welcome to balenaOS
=============================================================
root@827b231:~#
```
By default, SSH access is routed into the host balenaOS shell. However, you
can SSH into a service by specifying its name as part of the command:
```
$ balena ssh 1234567 main
root@827b231:/usr/src/app#
```
This also works in multicontainer applications, simply pass the name of the
appropriate service as defined in `docker-compose.yml` you'd like to access
the shell for.

When using device UUIDs, `balena ssh` uses the balena VPN to create a secure
tunnel to the device and then forward SSH traffic between it and your
development machine (for production devices, this is the only available method).

For devices running development images on your local network, you can also
use SSH by specifying the hostname or IP address of that device (development
images have SSH enabled by default). Using `balena ssh` in this way doesn't use
the balena VPN and instead makes a direct SSH connection to the device.
For example:
```
$ balena ssh 192.168.1.2
```
To find the hostname of a local development device, you can use `balena scan`:
```
$ sudo balena scan
Reporting scan results
-
  host:          827b231.local
  address:       192.168.1.173
  dockerInfo:
    Containers:        1
    ContainersRunning: 1
    ContainersPaused:  0
    ContainersStopped: 0
    Images:            2
    Driver:            aufs
    SystemTime:        2019-09-05T13:31:34.910619617Z
    KernelVersion:     4.14.98
    OperatingSystem:   balenaOS 2.38.0+rev1
    Architecture:      armv7l
  dockerVersion:
    Version:    18.09.6-dev
    ApiVersion: 1.39
```
In this instance `827b231.local` is the hostname, so the device can be SSHd
into using `balena ssh 827b231.local`. Note that by default, the hostname
of a device is always its short UUID, so if you already know the UUID for the
device, you can `balena ssh <uuid>.local` without having to perform a
`balena scan`.

## 5. Building and Deploying an Application without the Builder

### 5.1 Building an Image on a Development Machine

Whilst you can build applications using the balenaCloud builder, it's also
possible to build the application and generate the Docker images locally
on your development machine.

There are several reasons why you want might to do this. For example, should
your development machine exist on an airgapped network (with no Internet
connection), but the base images for a build as well as all the other
package requirements your build will need exist on the local network, this
allows builds for balena devices to still be carried out.

Another good example is if you have your own CI/CD pipeline with dedicated
machines that cache specific package/build data that you use frequently. In
these cases, a build on a local machine may be significantly quicker than
using balena generic builders.

Before we try building locally, it's worth a note on an extra switch that can be
used with `balena push`. `--emulated` tells balenaCLI that the target
architecture environment should be emulated, if it differs from that of the
native architecture on which balenaCLI is being run. For example, most
development machines tend to use an x64 architecture, whereas a large number of
devices are based around Armv6 or v7 (and more lately v8) architectures. To
correctly build images for Arm targets, an x64 builder must emulate the target
architecture whilst running the Docker commands. Because we're assuming the use
of a balenaFin here, we'll run all local builds using the `--emulated` switch.
Should you be building for an Intel NUC, or other AMD64 based device, you do not
need to pass this switch in the following examples.

To carry out a local build requires more information than a `balena push`,
because balenaCLI needs to know the CPU architecture and device type to produce
a Docker image that will work on the specified target. The easiest way to do
this is to specify an application, which will allow balenaCLI to determine this
information itself by querying the balenaCloud API. In the
`balena-cli-masterclass` repository, execute this command:
```
$ balena build --application cliApp --emulated
[Info]    Creating default composition with source: /Work/Support/MasterClasses/repos/balena-cli-masterclass
[Info]    Building for armv7hf/fincm3
[Info]    Emulation is enabled
[Build]   Built 1 service in 1:32
[Build]   main Image size: 208.45 MB
```
A call to `docker images` will show the newly built image:
```
$ docker images
REPOSITORY                     TAG                                        IMAGE ID            CREATED              SIZE
balena-cli-masterclass_main    latest                                     321025486d49        About a minute ago   219MB
```

As mentioned, there are instances where the ability to use balenaCloud is not
possible (for example an airgapped network), or is not desirable. In these
situations, `balena build` can be notified of the device type and architecture
to build on the command line:
```
$ balena build --arch armv7hf --deviceType fincm3 --emulated
[Info]    Creating default composition with source: /Work/Support/MasterClasses/repos/balena-cli-masterclass
[Info]    Building for armv7hf/fincm3
[Info]    Emulation is enabled
[Build]   Built 1 service in 1:41
[Build]   main Image size: 219.15 MB
[Success] Build succeeded!
```

There are a few caveats to building images locally, however. Emulated builds
will always be slower than native builds due to having to mimic a different
architecture. Coupled with other factors, such as potentially lower
network bandwidth than that enjoyed by the balenaCloud builders, this can
mean a far slower build than would occur than pushing to our native builders
(which use both dedicated 64bit AMD64 and Arm servers).

### 5.2 Deploying an Image from a Development Machine

An image from a development machine can be deployed as an application release
to balenaCloud from balenaCLI. This allows any pre-built image to be uploaded
directly to balena's registry without the requirement of the builder to
generate it first. Assuming you've followed exercise 5.1, run the following:
```
$ balena deploy cliApp balena-cli-masterclass_main:latest
[Info]    Creating default composition with image: balena-cli-masterclass_main:latest
[Info]    Everything is up to date (use --build to force a rebuild)
[Info]    Creating release...
[Info]    Pushing images to registry...
[Info]    Saving release...
[Success] Deploy succeeded!
[Success] Release: b9e5ec4b309f91281ecb592028dcea0c

			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'
```
This will create a new release (visible via the dashboard), and push the image
directly to the balena Docker registry. Your balenaFin should then download the
new release and run it. This is useful if you already have an image pre-built
and just need to upload it.

However, `balena deploy` also allows you to complete the build step as well
implicitly by not specifying an image to upload. Run the following command in
the `balena-cli-masterclass` repository:
```
$ balena deploy cliApp --build --emulated
[Info]    Creating default composition with source: /Work/Support/MasterClasses/repos/balena-cli-masterclass
[Info]    Building for armv7hf/fincm3
[Build]   Built 1 service in 0:58
[Build]   main Image size: 213.80 MB
[Info]    Creating release...
[Info]    Pushing images to registry...
[Info]    Saving release...
[Success] Deploy succeeded!
[Success] Release: 13f39923c2ddf95fa35a129d8efb5b53

			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'
```
This forces the `deploy` command to first build (or rebuild if the image already
exists) the project before pushing it to the Docker registries.

## 6. Using Local Mode to Develop Applications

So far, you've seen how to push code to the balena builders, or to build and
push images on a development machine. Whilst practical solutions for pre-tested
code, or for a CI pipeline, this is not a fast workflow for active development
of an application by an engineer as it involves rebuilding an image and then
delivering it to the target device.

To make active development of applications easier for an engineer, balena
devices provisioned with a development image include a device mechanism called
'Local Mode'. This can be activated easily from the dashboard. Go to your
device's dashboard page, select 'Actions' from the lefthand toolbar, and then
select 'Enable local mode'. Local mode does a couple of important things:

* Stops running the application currently associated with it, including stopping
	all running containers
* Exposes a Docker socket on the local network

Once activated, balenaCLI can push code directly to the local device instead
of going via the balena builders. Code is built on the device and then executed,
which can significantly speed up development when requiring frequent changes.
As mentioned previously, you can find local devices on your network in
development mode by using `balena scan`.

`balena push` includes optional switches which allow you to specify that you
want to push code to a local device using the results from `balena scan`.
To see this working in practice, carry out a `balena scan`, and then pass either
the host or IP address to `balena push` whilst in the `balena-cli-masterclass`
repository:
```
$ balena push 827b231.local
[Info]    Starting build on device 827b231.local
[Info]    Creating default composition with source: .
[Build]   [main] Step 1/8 : FROM balenalib/fincm3-node:8
[Build]   [main]  ---> 392c3f6339f7
[Build]   [main] Step 2/8 : WORKDIR /usr/src/app
[Build]   [main]  ---> Running in 446517b1afdb
[Build]   [main] Removing intermediate container 446517b1afdb
[Build]   [main]  ---> c27a2f22ba27
[Build]   [main] Step 3/8 : COPY package.json package-lock.json ./
[Build]   [main]  ---> fc79948ab18b
[Build]   [main] Step 4/8 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[Build]   [main]  ---> Running in 59c1b1cbe571
[Build]   [main] added 50 packages from 37 contributors and audited 126 packages in 9.178s
[Build]   [main] found 0 vulnerabilities
[Build]   [main] npm
[Build]   [main]
[Build]   [main] WARN
[Build]   [main]
[Build]   [main] using --force
[Build]   [main]  I sure hope you know what you are doing.
[Build]
[Build]   [main] Removing intermediate container 59c1b1cbe571
[Build]   [main]  ---> 0cdc6d1d7af9
[Build]   [main] Step 5/8 : COPY src/ ./src/
[Build]   [main]  ---> 23e41b46ee6f
[Build]   [main] Step 6/8 : CMD ["npm", "start"]
[Build]   [main]  ---> Running in c62943c5e22f
[Build]   [main] Removing intermediate container c62943c5e22f
[Build]   [main]  ---> 9434eb22cc67
[Build]   [main] Step 7/8 : LABEL io.resin.local.image=1
[Build]   [main]  ---> Running in 9023bae27f0f
[Build]   [main] Removing intermediate container 9023bae27f0f
[Build]   [main]  ---> 7e2603a523f2
[Build]   [main] Step 8/8 : LABEL io.resin.local.service=main
[Build]   [main]  ---> Running in 05feac72072e
[Build]   [main] Removing intermediate container 05feac72072e
[Build]   [main]  ---> 9ef9a4510175
[Build]   [main] Successfully built 9ef9a4510175
[Build]   [main] Successfully tagged local_image_main:latest

[Info]    Streaming device logs...
[Live]    Watching for file changes...
[Live]    Waiting for device state to settle...
[Logs]    [9/5/2019, 2:34:37 PM] Creating network 'default'
[Logs]    [9/5/2019, 2:34:37 PM] Creating volume 'resin-data'
[Logs]    [9/5/2019, 2:34:37 PM] Installing service 'main sha256:9ef9a45101757ee81aa26d5ca43713289b2e99401d1b13e32842523876fde664'
[Logs]    [9/5/2019, 2:34:38 PM] Installed service 'main sha256:9ef9a45101757ee81aa26d5ca43713289b2e99401d1b13e32842523876fde664'
[Logs]    [9/5/2019, 2:34:38 PM] Starting service 'main sha256:9ef9a45101757ee81aa26d5ca43713289b2e99401d1b13e32842523876fde664'
[Logs]    [9/5/2019, 2:34:39 PM] Started service 'main sha256:9ef9a45101757ee81aa26d5ca43713289b2e99401d1b13e32842523876fde664'
[Live]    Device state settled
[Logs]    [9/5/2019, 2:34:41 PM] [main]
[Logs]    [9/5/2019, 2:34:41 PM] [main] > balena-cli-masterclass@1.0.0 start /usr/src/app
[Logs]    [9/5/2019, 2:34:41 PM] [main] > node src/helloworld.js; sleep infinity
[Logs]    [9/5/2019, 2:34:41 PM] [main]
[Logs]    [9/5/2019, 2:34:41 PM] [main] Hello world!
```
Once the code has been built on the device, it immediately starts executing
and logs are output to the console. You can halt the connection to the local
device by using `Ctrl-C`. Note that after disconnection, the service containers
on the device will continue to run.

In a multicontainer environment, it may quickly become difficult for an engineer
to determine whether their code is working, especially if many services are
all outputting log information. In these cases, filtering log output via
service is possible, by using the `--service` switch:
```
$ balena push 827b231.local --service main
[Info]    Starting build on device 827b231.local
[Info]    Creating default composition with source: .
[Build]   [main] Step 1/8 : FROM balenalib/fincm3-node:8
[Build]   [main]  ---> 392c3f6339f7
[Build]   [main] Step 2/8 : WORKDIR /usr/src/app
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> c27a2f22ba27
[Build]   [main] Step 3/8 : COPY package.json package-lock.json ./
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> fc79948ab18b
[Build]   [main] Step 4/8 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> 0cdc6d1d7af9
[Build]   [main] Step 5/8 : COPY src/ ./src/
[Build]   [main]  ---> 5adcd43b12c6
[Build]   [main] Step 6/8 : CMD ["npm", "start"]
[Build]   [main]  ---> Running in a415b6e7f0af
[Build]   [main] Removing intermediate container a415b6e7f0af
[Build]   [main]  ---> 1a1ff3926d42
[Build]   [main] Step 7/8 : LABEL io.resin.local.image=1
[Build]   [main]  ---> Running in 7390bcd46425
[Build]   [main] Removing intermediate container 7390bcd46425
[Build]   [main]  ---> ebf9fc9c43ed
[Build]   [main] Step 8/8 : LABEL io.resin.local.service=main
[Build]   [main]  ---> Running in 52ab62708a46
[Build]   [main] Removing intermediate container 52ab62708a46
[Build]   [main]  ---> 60f2a99b07c8
[Build]   [main] Successfully built 60f2a99b07c8
[Build]   [main] Successfully tagged local_image_main:latest

[Live]    Waiting for device state to settle...
[Info]    Streaming device logs...
[Live]    Watching for file changes...
[Logs]    [9/5/2019, 2:46:54 PM] [main]
[Logs]    [9/5/2019, 2:46:54 PM] [main] > balena-cli-masterclass@1.0.0 start /usr/src/app
[Logs]    [9/5/2019, 2:46:54 PM] [main] > node src/helloworld.js; sleep infinity
[Logs]    [9/5/2019, 2:46:54 PM] [main]
[Logs]    [9/5/2019, 2:46:54 PM] [main] Hello world!
```
As you can see, none of the Supervisor logs were printed. Note that there is
also a `balena logs` command that is dedicated to just showing logs. This
command includes both the `--system` and `--service` switches to filter
output to just that of system messages or particular service messages (these
switches can be combined in a single `balena logs` call). This allows the
setup of multiple terminals to act as loggers whilst another is used to
carry out `balena push` executions. A few examples of logging are shown below:

* `balena logs 827b231.local --system --service main` - Will output all
	system messages and those from the `main` service
* `balena logs 827b231.local --service main` will only output messages from
	the `main` service
* `balena logs 827b231.local --service main --service secondary` will only
	output messages from the `main` and `secondary` services

Local Mode also has another huge benefit, known as Livepush.
[Livepush](https://github.com/balena-io-modules/livepush) makes intelligent
decisions on how, or even if, to rebuild an image when changes are made. It
does this by examining the source directory of an image being built on your
local development machine (via balenaCLI), and then deciding how to deal with
changes.

In some cases, Livepush rebuilds relevant parts of the image before starting the
new image as the service. As an example of this, ensure you've executed
`balena push` in Local Mode:
```
$ balena push 827b231.local --service main
```
Now modify `Dockerfile.template` in the `balena-cli-masterclass` repository in a
text editor, inserting a new line between the `COPY src/ ./src/` command and
`CMD ["npm", "start"]`:

```
...
COPY src/ ./src/

RUN echo "Rebuild the image"

CMD ["npm", "start"]
```
Finally, save the changes to the file in your text editor. The Supervisor
will immediately detect that the Dockerfile has changed and start a rebuild
of the service:
```
[Live]    Detected Dockerfile change, performing full rebuild of service main
[Build]   [main] Step 1/9 : FROM balenalib/fincm3-node:8
[Build]   [main]  ---> 392c3f6339f7
[Build]   [main] Step 2/9 : WORKDIR /usr/src/app
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> c27a2f22ba27
[Build]   [main] Step 3/9 : COPY package.json package-lock.json ./
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> fc79948ab18b
[Build]   [main] Step 4/9 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[Build]   [main]  ---> Using cache
[Build]   [main]  ---> 0cdc6d1d7af9
[Build]   [main] Step 5/9 : COPY src/ ./src/
[Build]   [main]  ---> fec02483c800
[Build]   [main] Step 6/9 : RUN echo "Rebuild the image"
[Build]   [main]  ---> Running in 0c1f2cbca19f
[Build]   [main] Rebuild the image
[Build]   [main] Removing intermediate container 0c1f2cbca19f
[Build]   [main]  ---> aec92835242c
[Build]   [main] Step 7/9 : CMD ["npm", "start"]
[Build]   [main]  ---> Running in 824e56df2920
[Build]   [main] Removing intermediate container 824e56df2920
[Build]   [main]  ---> c49d013039de
[Build]   [main] Step 8/9 : LABEL io.resin.local.image=1
[Build]   [main]  ---> Running in e9158ba83571
[Build]   [main] Removing intermediate container e9158ba83571
[Build]   [main]  ---> 2317085d9161
[Build]   [main] Step 9/9 : LABEL io.resin.local.service=main
[Build]   [main]  ---> Running in ebcaf1a3351d
[Build]   [main] Removing intermediate container ebcaf1a3351d
[Build]   [main]  ---> fc5e4459c406
[Build]   [main] Successfully built fc5e4459c406
[Build]   [main] Successfully tagged local_image_main:latest
[Logs]    [9/5/2019, 3:58:56 PM] [main]
[Logs]    [9/5/2019, 3:58:56 PM] [main] > balena-cli-masterclass@1.0.0 start /usr/src/app
[Logs]    [9/5/2019, 3:58:56 PM] [main] > node src/helloworld.js; sleep infinity
[Logs]    [9/5/2019, 3:58:56 PM] [main]
[Logs]    [9/5/2019, 3:58:56 PM] [main] Hello world!
```
Once rebuilt, it will restart the service. Notice that the `Rebuild the image`
echoed line is now in the build log.

Livepush goes way further than this, however. Only files that affect the
building of the service force a rebuild. For other files, for example source
files that run in-service, the Supervisor replaces the files in-situ in the
relevant container layer.
To show this, continue to run the `balena push` command and then alter the
`src/helloworld.js` in a text editor and change:
```
console.log('Hello world!');
```
to
```
console.log('Hello moon!');
```
On saving the file, you'll see the following output:
```
[Live]    Detected changes for container main, updating...
[Live]    [main] Restarting service...
[Logs]    [9/5/2019, 4:02:27 PM] [main]
[Logs]    [9/5/2019, 4:02:27 PM] [main] > balena-cli-masterclass@1.0.0 start /usr/src/app
[Logs]    [9/5/2019, 4:02:27 PM] [main] > node src/helloworld.js; sleep infinity
[Logs]    [9/5/2019, 4:02:27 PM] [main]
[Logs]    [9/5/2019, 4:02:28 PM] [main] Hello moon!
```
Instead of rebuilding the image, which takes time, the file is injected directly
into the container's file system and then it is restarted. This happens in a
few seconds and makes the process of developing much faster and more convenient.

Sometimes an engineer may not want to rebuild code 'on the fly'. For this reason
`balena push` in Local Mode also has a `--nolive` option which can be passed
to it. When using this switch, engineers need to re`push` when they want to
rebuild code.

Livepush also supports `balena logs`, and can be used in the same was as
described earlier.

## 7. Using Private Registries

As well as using public Docker registries it's possible to instruct builders,
either balena-based or using Local Mode, to pull images from private Docker
registries. This is achieved by using the `--registry-secrets` switch when
calling `balena push` passing a filename containing the secret information.
This information can be in either YAML or JSON. For example, a relevant
JSON object containing this information follows:
```
{
    "https://index.docker.io/v1/": {
		"username": "myUser",
		"password": "myPassword"
	}
}
```
If saved as a JSON file, for example `secrets.json`, it will then be used when
a base image or image for a service is pulled which requires credentials:
```
$ balena push --registry-secrets secrets.json
```
You can also save a file with secrets in JSON or YAML format in your home
directory, under `~/.balena/secrets.<yml|json>`, which will automatically be
used for the secrets if it exists and the `--registry-secrets` switch has not
been passed to `balena push`.

## 8. Building with Secrets and Variables

Building images occasionally requires the use of credentials (such as those
for private repositories), or environment variables that may change depending
on circumstances such as architecture (for example package versioning).

The following exercise sections show you how to use build-time secrets and
variable substitution.

### 8.1 Build Time Secrets

Sometimes it is necessary to build images using secret information, commonly
to login to source repositories or fetch data which is required for the building
of an image, but which should not exist *in* that image when run as a service
container.

Our builders allow you to do this by adding such secrets in files in a
`.balena` directory in the root of the build directory. This allows them to
be passed to builders, which will use (and then discard) them for generating
images.

We'll make a few changes to the example project to show this in operation.
First create a `.balena` directory in the root of the `balena-cli-masterclass`
directory, and then create an empty `balena.yml` file and create another
directory called `secrets` in the `.balena` directory. You should now have a
file tree that looks like this:
```
$ tree -a -I .git
.
├── .balena
│   ├── balena.yml
│   └── secrets
├── Dockerfile.template
├── README.md
├── package-lock.json
├── package.json
└── src
    └── helloworld.js
```

You can now add secrets to the build by adding a section to the `balena.yml`
file and then creating appropriate secret files in the `.balena/secrets`
directory. We'll add some now, open a text editor and fill the `balena.yml`
file with the following:
```
build-secrets:
  global:
    - source: my-build-secrets
      dest: my-secrets
```
Note that the source file should exist in the `.balena/secrets` directory, and
that it is mapped into the `my-secrets` file when the image is built.
Save the file, and create a new one called `.balena/secrets/my-build-secrets`
and copy the following into it:
```
This file has build-time secrets!
```
Finally, we'll add a line into our Dockerfile that uses the secrets file, which
are mapped into the `/run/secrets/` directory during build time:
```
COPY src/ ./src/

RUN cat /run/secrets/my-secrets

CMD ["npm", "start"]
```
Now, push the project to the builders:
```
$ balena push cliApp
[Info]     Starting build for cliApp, user heds
[Info]     Dashboard link: https://dashboard.balena-cloud.com/apps/1505952/devices
[Info]     Building on arm03
[Info]     Pulling previous images for caching purposes...
[Success]  Successfully pulled cache images
[main]     Step 1/7 : FROM balenalib/fincm3-node:8
[main]      ---> 392c3f6339f7
[main]     Step 2/7 : WORKDIR /usr/src/app
[main]      ---> f9c421b7aa77
[main]     Removing intermediate container df598a62ef2c
[main]     Step 3/7 : COPY package.json package-lock.json ./
[main]      ---> 6af34973ecb0
[main]     Removing intermediate container d7b84160d116
[main]     Step 4/7 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[main]      ---> Running in e0a52c6e7d2e
[main]     added 50 packages from 37 contributors and audited 126 packages in 2.53s
[main]     found 0 vulnerabilities
[main]     npm
[main]      WARN using --force
[main]      I sure hope you know what you are doing.
[main]
[main]      ---> 9ff0e005febd
[main]     Removing intermediate container e0a52c6e7d2e
[main]     Step 5/7 : COPY src/ ./src/
[main]      ---> 211ef0e15576
[main]     Removing intermediate container 75e2127552bf
[main]     Step 6/7 : RUN cat /run/secrets/my-secrets
[main]      ---> Running in 949098cba8b6
[main]     This file has build-time secrets!
[main]      ---> 6da25ec8f94f
[main]     Removing intermediate container 949098cba8b6
[main]     Step 7/7 : CMD npm start
[main]      ---> Running in 1ae564a36e44
[main]      ---> e590faac0013
[main]     Removing intermediate container 1ae564a36e44
[main]     Successfully built e590faac0013
[Info]     Uploading images
[Success]  Successfully uploaded images
[Info]     Built on arm03
[Success]  Release successfully created!
[Info]     Release: cedeff39dc537c9cd4c1df3b97ffcbc7 (id: 1055448)
[Info]     ┌─────────┬────────────┬────────────┐
[Info]     │ Service │ Image Size │ Build Time │
[Info]     ├─────────┼────────────┼────────────┤
[Info]     │ main    │ 208.47 MB  │ 11 seconds │
[Info]     └─────────┴────────────┴────────────┘
[Info]     Build finished in 46 seconds
			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'
```
As you can see, step 6 output:
```
[main]     Step 6/7 : RUN cat /run/secrets/my-secrets
[main]      ---> Running in 949098cba8b6
[main]     This file has build-time secrets!
```
which is the contents of the secrets file. This file could obviously contain
a raft of different functions, including a script that gets executed, text
for filling in files, etc. As well as defining globally accessible secrets
(which are shared to all services being built), there is also the option to
define secrets that are only accessible to particular services, or to map them
to different paths. This becomes useful in multicontainer build scenarios. This
can be achieved by appending a `services` section to `build-secrets` on the
`balena.yml` file. For example:
```
build-secrets:
  services:
    main:
      - source: main-only-secrets
        dest: my-main-secrets
```
This change would map the `.balena/secrets/main-only-secrets` file into the
`/run/secrets/my-main-secrets` runtime path at build-time but *only for the
`main` service*.

### 8.2 Build Time Variables

Another frequent build-time use is that of environment variables that may alter
between builds but still use the same flow of a Dockerfile. Allowing these to
be defined dynamically means that no changes to a Dockerfile are required as
long as the variables are referenced within them.

Much in a similar way to secrets files, these are defined in the
`.balena/balena.yml` file. Add two new build-time arguments to your
`balena.yml` file:
```
build-variables:
  global:
    - BUILD_TIME_ARG_1=This is the first arg
    - BUILD_TIME_ARG_2=This is the second arg
```
Now alter the Dockerfile to use them:
```
RUN cat /run/secrets/my-secrets

ARG BUILD_TIME_ARG_1
ARG BUILD_TIME_ARG_2

RUN echo "${BUILD_TIME_ARG_1}" && \
    echo "${BUILD_TIME_ARG_2}"

CMD ["npm", "start"]
```
Note that you need to ensure that both arguments are declared using the `ARG`
Docker command. Save the file and push to the builders:
```
$ balena push cliApp --nocache
[Info]     Starting build for cliApp, user heds
[Info]     Dashboard link: https://dashboard.balena-cloud.com/apps/1505952/devices
[Info]     Building on arm01
[Info]     Pulling previous images for caching purposes...
[Success]  Successfully pulled cache images
[main]     Step 1/10 : FROM balenalib/fincm3-node:8
[main]      ---> 392c3f6339f7
[main]     Step 2/10 : WORKDIR /usr/src/app
[main]      ---> 0d930a93f967
[main]     Removing intermediate container 37dcff73ed1b
[main]     Step 3/10 : COPY package.json package-lock.json ./
[main]      ---> 036c9574af90
[main]     Removing intermediate container 19b7f4d28593
[main]     Step 4/10 : RUN npm install --ci --production     && npm cache clean --force     && rm -f /tmp/*
[main]      ---> Running in fa65ea126c2d
[main]     added 50 packages from 37 contributors and audited 126 packages in 2.436s
[main]     found 0 vulnerabilities
[main]     npm
[main]      WARN using --force I sure hope you know what you are doing.
[main]
[main]      ---> 6369ab6a6802
[main]     Removing intermediate container fa65ea126c2d
[main]     Step 5/10 : COPY src/ ./src/
[main]      ---> a7103d76c564
[main]     Removing intermediate container ae6e3614de1d
[main]     Step 6/10 : RUN cat /run/secrets/my-secrets
[main]      ---> Running in 3bc6e36ce8d1
[main]     This file has build-time secrets!
[main]      ---> 6ee5058cc5f3
[main]     Removing intermediate container 3bc6e36ce8d1
[main]     Step 7/10 : ARG BUILD_TIME_ARG_1
[main]      ---> Running in a42ba6af90d6
[main]      ---> 06bd854b2e68
[main]     Removing intermediate container a42ba6af90d6
[main]     Step 8/10 : ARG BUILD_TIME_ARG_2
[main]      ---> Running in dcc256fdc0a6
[main]      ---> e283099d8adc
[main]     Removing intermediate container dcc256fdc0a6
[main]     Step 9/10 : RUN echo "${BUILD_TIME_ARG_1}" &&     echo "${BUILD_TIME_ARG_2}"
[main]      ---> Running in 1c572bc0e7c0
[main]     This is the first arg
[main]     This is the second arg
[main]      ---> 81c8da87d775
[main]     Removing intermediate container 1c572bc0e7c0
[main]     Step 10/10 : CMD npm start
[main]      ---> Running in 81dd184fc8ee
[main]      ---> d494a4090d2d
[main]     Removing intermediate container 81dd184fc8ee
[main]     Successfully built d494a4090d2d
[Info]     Uploading images
[Success]  Successfully uploaded images
[Info]     Built on arm01
[Success]  Release successfully created!
[Info]     Release: 7efbc95825641b6482742a54c8e74010 (id: 1056307)
[Info]     ┌─────────┬────────────┬────────────┐
[Info]     │ Service │ Image Size │ Build Time │
[Info]     ├─────────┼────────────┼────────────┤
[Info]     │ main    │ 208.48 MB  │ 9 seconds  │
[Info]     └─────────┴────────────┴────────────┘
[Info]     Build finished in 28 seconds
			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'
```
As can be seen, both build arguments were available in the log output:
```
[main]     This is the first arg
[main]     This is the second arg
```

In the same way that build secrets can be made service specific, so may secret
build arguments, by specifying them directly in the `.balena/balena.yml` file:
```
build-variables:
  services:
    main:
      - MAIN_ONLY_BUILD_ARG=This is only available when building 'main'
```

An important note for build variables is that, unlike secrets, they will be
defined and available as part of the final image. If you need variables to
be secret, they should be defined specifically as secrets.

# Conclusion

In this masterclass, you've learnt how to use the most commonly used balenaCLI
commands, as well as how to start development using it. You should now be
familiar and confident enough to:

* Create applications for specific device types
* Provision devices as well as SSH into balenaOS and any running service
	container
* Push code to applications, either via `balena push` or `git push`
* Locally build service images on a development machine, as well as deploying
	those images to balenaCloud
* Switch a development device into Local Mode, push code locally to a device
	to build an application and service images
* Use Livepush to dynamically alter application code on the fly and immediately
	see results on a device in Local Mode, as well as filter log output for
	specific services
* Use build secret files and arguments to generate images which
	should not include those secrets, as well as build variables

# References

None
