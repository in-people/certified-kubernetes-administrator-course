# Pod Networking

  - Take me to [Lecture](https://kodekloud.com/topic/pod-networking/)

In this section, we will take a look at **Pod Networking**


![net-2615](../../images/net2615.PNG)



- To add bridge network on each node

> node01
```
$ ip link add v-net-0 type bridge
```
> node02
```
$ ip link add v-net-0 type bridge
```

> node03
```
$ ip link add v-net-0 type bridge
```

![net-2616](../../images/net2616.PNG)


- Currently it's down, turn it up.

> node01
```
$ ip link set dev v-net-0 up
```

> node02
```
$ ip link set dev v-net-0 up
```

> node03
```
$ ip link set dev v-net-0 up
```

- Set the IP Addr for the bridge interface

> node01
```
$ ip addr add 10.244.1.1/24 dev v-net-0
```

> node02
```
$ ip addr add 10.244.2.1/24 dev v-net-0
```

> node03
```
$ ip addr add 10.244.3.1/24 dev v-net-0
```

![net-11](../../images/net11.PNG)

![net-2617](../../images/net2617.PNG)

- Check the reachability 

```
$ ping 10.244.2.2
Connect: Network is unreachable
```

- Add route in the routing table
```
$ ip route add 10.244.2.2 via 192.168.1.12
```

> node01
```
$ ip route add 10.244.2.2 via 192.168.1.12

$ ip route add 10.244.3.2 via 192.168.1.13
```

> node02
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.3.2 via 192.168.1.13

```

> node03
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.2.2 via 192.168.1.12
```

- Add a single large network 

![net-12](../../images/net12.PNG)

![net-2618](../../images/net2618.PNG)

## Container Network Interface


![net-13](../../images/net13.PNG)


![net-2619](../../images/net2619.PNG)

![net-2620](../../images/net2620.PNG)






#### References Docs

- https://kubernetes.io/docs/concepts/workloads/pods/
