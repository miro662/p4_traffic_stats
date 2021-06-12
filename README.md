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

__TASK 0__: Investigate configuration of `laboratory.p4app` (it is present in the main folder). Which version of P4 will be used (P4_14/P4_16)? How the topology created by p4app will look like?

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
$ echo $P4APP_CID
```
Remember to refresh this envvar every time you re-launch p4app.

## 1) Counters
### 1.1) Defining a counter
__Counters__ are simple monitoring primitives available in P4. They are arrays of integers and can be used to count number of data (packets or bytes).

Counters are associated with `control` structures. In order to create one, you define it this way:
```p4
control ... {
    counter(counter_size, counter_type) counter_name;

    // actions, tables, other counters, ...
}
```
where:
* __counter_size__ is a size of an array
* __counter_type__ declares what should be counted by given counter:
    - `CounterType.packets` - packets
    - `CounterType.bytes` - bytes
    - `CounterType.packets_and_bytes` - both
* __counter_name__ is a name of a counter

Let's create a counter that will count total amount of packets sent through our switch. In file `laboratory.p4app\simple_router.p4`, in `ingress` control, add new counter:

```p4
...
control ingress(inout headers hdr, inout metadata meta, inout standard_metadata_t standard_metadata) {
    counter(1, CounterType.packets) packetsSent;
    ...
}
```

Restart `p4app`. There should be a warning about unused counter:
```
simple_router.p4(34): [--Wwarn=unused] warning: c: unused instance
    counter(64, CounterType.packets) c;
```
We'll use our counter later.

### 1.2) Accesing counter data
There is no way to access counter data directly from P4 data flow. However, we can access them from our switch in a different way. In case of software BMV2 switch, used in our laboratory, we can use `simple_switch_CLI`. It is included in p4app's Docker image.

In order to access `simple_switch_CLI`, run (from host):
```bash
$ docker exec -t -i $P4APP_CID simple_switch_CLI
```

Now, you can read counter values, using `counter_read {control_name}.{counter_name} {index}`:
```
RuntimeCmd: counter_read ingress.packetsSent 0
```

As we do not do anything with our counter yet, it will be equal to 0:
```
BmCounterValue(packets=0, bytes=0)
```

(Note: BMV2 always creates counter which counts both packets and bytes because of reasons. However, do not rely on this behaviour in your code - as real switches might work differently.)

### 1.3) Incrementing counter
In order to record currently transmitted packet in a counter, use `.count(idx)` method. For example, add it to `set_dmac` action:
```p4
    action set_dmac(bit<48> dmac) {
        hdr.ethernet.dstAddr = dmac;
        packetsSent.count(0);
    }
```
Restart p4app. Check counter in simple_switch_CLI (you'll have to run it again, as our previous container was terminated). Ping between h1 and h2:
```
mininet> h1 ping h2
```
and terminate it after a few (N) packets. Now, check counter in simple_switch_CLI again. How many packets were transmitted? What is the average size of an Ethernet frame containing ICMP Echo Request / Echo Reply?

Remember that packet is counted only if `count()` method is called during its processing. 

__TASK 1__: Create a counter `packetsDropped`, which will count amount of packets that were dropped during ingress processing.





