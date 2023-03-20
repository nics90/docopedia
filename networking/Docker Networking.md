
In Docker, each container runs in its own isolated network namespace. This means that by default, each container has its own virtual network interface, IP address, and routing table.

There are several ways that containers can communicate with each other and with the outside world:

1.  Bridge network: The default network type in Docker is the bridge network. When a container is started with no network options specified, it is automatically connected to the default bridge network. Containers on the same bridge network can communicate with each other using their container names or IP addresses.

Here is an example diagram of a bridge network in Docker:

```
     +-------------+
     | Docker Host |
     +-------------+
            |
  +---------------------+
  | Default Bridge Network |
  +---------------------+
  |         |           |
+----------+--+     +--+----------+
| Container A |     | Container B |
+-------------+     +-------------+

```

In this diagram, two containers (A and B) are connected to the default bridge network on the Docker host. They can communicate with each other using their container names or IP addresses.

In Docker's bridge network, each container is given a unique IP address and hostname within the network. This allows containers to communicate with each other using their IP addresses or hostnames.

When a container is started on a bridge network, Docker assigns it an IP address from the subnet range of the network. For example, if the bridge network uses the subnet `172.17.0.0/16`, Docker might assign the container the IP address `172.17.0.2`. This IP address is unique within the network and allows other containers on the same network to communicate with it.

In addition to the IP address, each container is also given a hostname based on its container name. For example, if the container is named `web`, its hostname within the network would be `web`. This hostname can be used by other containers on the same network to communicate with the container.

For example, let's say we have two containers named `web` and `db` on a bridge network. The `web` container needs to connect to the `db` container to retrieve data. It can do this by using the `db` container's IP address or hostname. Here's an example of how this communication might work:

1.  The `web` container sends a request to the `db` container using its IP address: `curl http://172.17.0.3/data`.
    
2.  The `db` container receives the request and sends back the requested data.
    

Alternatively, the `web` container could use the `db` container's hostname to make the request: `curl http://db/data`. Docker's built-in DNS server will resolve the `db` hostname to the `db` container's IP address, allowing the communication to take place.

In summary, Docker's bridge network assigns each container a unique IP address and hostname within the network, which allows containers to communicate with each other using these identifiers.

2.  Host network: Containers can also be started with the `--net=host` option, which makes them share the network namespace with the Docker host. This means that they have access to the same network interfaces and IP addresses as the host, and can communicate with other containers or external systems using the host's network.

Earlier I mentioned that Docker's bridge network operates at Layer 2 and uses MAC addresses for communication. While this is true at the low-level implementation of the bridge network, from a user's perspective, communication between containers on the bridge network is typically done using IP addresses or hostnames, which operate at Layer 3 of the network stack.

To clarify, when a container sends a network packet to another container on the same bridge network, Docker's bridge network implementation uses the MAC address of the destination container to forward the packet to the correct destination. However, from the perspective of the user, they typically use IP addresses or hostnames to identify and communicate with other containers on the network.

So while the low-level implementation of the bridge network uses MAC addresses for communication, the user typically interacts with the network using IP addresses or hostnames.

Here is an example diagram of a container using the host network in Docker:

```
     +-------------+
     | Docker Host |
     +-------------+
            |
     +-------------+
     | Container A |
     +-------------+

```

In this diagram, Container A is using the host network, so it shares the same network namespace as the Docker host. It can communicate with other containers or external systems using the host's network.

3.  Overlay network: In a multi-host Docker environment, you can create overlay networks that span multiple Docker hosts. This allows containers on different hosts to communicate with each other as if they were on the same network.

Here is an example diagram of an overlay network in Docker:

```
+-------------+                 +-------------+
| Docker Host |                 | Docker Host |
+-------------+                 +-------------+
       |                                 |
+-------------+                 +-------------+
| Container A |                 | Container B |
+-------------+                 +-------------+
       |                                 |
+-------------------------------------------+
            | Overlay Network |
+-------------------------------------------+
       |                                 |
+-------------+                 +-------------+
| Container C |                 | Container D |
+-------------+                 +-------------+


```

In this diagram, four containers are connected to an overlay network that spans two Docker hosts. Containers A and B are on one host, while Containers C and D are on another host. They can all communicate with each other using the overlay network, as if they were on the same local network.
