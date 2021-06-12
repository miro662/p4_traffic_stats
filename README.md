# Obtaining traffic stats using P4
Authors: Mirosław Błażej, Łukasz Kowalski

## 0) Prerequirements
### 0.1) Set-up

Make sure that [Docker](https://docs.docker.com/engine/install/) is installed on your system. Then, clone this repository (with submodules) and move into it:
```bash
$ git clone --recursive https://github.com/miro662/p4_traffic_stats
$ cd p4_traffic_stats
```

### 0.2) p4app
This laboratory uses [p4app](https://github.com/p4lang/p4app) to provision necessary virtual infrastructure in a Docker container (such as Mininet and BMV2). Thanks to this, you do not need to install anything but Docker on your machine. p4app is included as submodule to this repository.

p4app expects app in a form of a directory which name is finished with `.p4app`. It should contain `p4app.json` file, containing informations about P4 program to run on switch and desired topology.

TASK 0: Investigate configuration of `laboratory.p4app` (it is present in main folder). Which version of P4 will be used (P4_14/P4_16)? How topology created by p4app will look like?

In order to run app using p4, use `p4app` script:

```bash
$ ./p4app/p4app run laboratory.p4app
```

After downloading Docker containers, it will run mininet shell. Test if you can ping between hosts:

```
mininet> pingall
```

In order to perform some additional operations in p4app, we'll need ID of Docker container where p4app runs. We'll get it and store it in an enviroment variable. In another terminal window, run:
```bash
$ P4APP_CID=`docker ps | grep p4app | awk '{print $1}'`
```

## 1) Counters
