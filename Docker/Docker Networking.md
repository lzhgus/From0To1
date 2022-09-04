# Networking overview 
## Network drivers[](https://docs.docker.com/network/#network-drivers)
---

Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality:

-   `bridge`: The default network driver. If you don’t specify a driver, this is the type of network you are creating. **Bridge networks are usually used when your applications run in standalone containers that need to communicate.** See [bridge networks](https://docs.docker.com/network/bridge/).
    
-   `host`: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly. See [use the host network](https://docs.docker.com/network/host/).
    
-   `overlay`: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers. See [overlay networks](https://docs.docker.com/network/overlay/).
    
-   `ipvlan`: IPvlan networks give users total control over both IPv4 and IPv6 addressing. The VLAN driver builds on top of that in giving operators complete control of layer 2 VLAN tagging and even IPvlan L3 routing for users interested in underlay network integration. See [IPvlan networks](https://docs.docker.com/network/ipvlan/).
    
-   `macvlan`: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. Using the `macvlan` driver is sometimes the best choice when dealing with legacy applications that expect to be directly connected to the physical network, rather than routed through the Docker host’s network stack. See [Macvlan networks](https://docs.docker.com/network/macvlan/).
    
-   `none`: For this container, disable all networking. Usually used in conjunction with a custom network driver. `none` is not available for swarm services. See [disable container networking](https://docs.docker.com/network/none/).
    
-   [Network plugins](https://docs.docker.com/engine/extend/plugins_services/): You can install and use third-party network plugins with Docker. These plugins are available from [Docker Hub](https://hub.docker.com/search?category=network&q=&type=plugin) or from third-party vendors. See the vendor’s documentation for installing and using a given network plugin.