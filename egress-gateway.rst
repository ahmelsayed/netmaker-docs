=====================================
Egress Gateway
=====================================

Introduction
===============

.. image:: images/egress1.png
   :width: 80%
   :alt: Gateway
   :align: center

Netmaker allows your clients to reach external networks via an Egress Gateway. The Egress Gateway is a netclient which has been deployed to a server or router with access to a given subnet.

In the Netmaker UI, that node is set as an "egress gateway." Range(s) are specified which this node has access to. Once created, all clients (and all new ext clients) in the network will be able to reach those ranges via the gateway.

Configuring an Egress Gateway
==================================

Configuring an Egress Gateway is very straight forward. As a prerequisite, you must know what you are trying to access remotely. For instance:

- a VPC
- a Kubernetes network
- a home network
- an office network
- a data center

After you have determined this, you must next deploy a netclient in a compatible location where the network is accessible. For instance, a Linux server or router in the office, or a Kubernetes worker node. This machine should be stable and relatively static (not expected to change its IP frequently or shut down unexpectedly).


Once you have determined the subnet, and deployed your netclient, you can go to your Netmaker UI and set the node as a gateway. click on the network on the sidebar and navigate to the egress section.

.. image:: images/egress7.png
   :width: 80%
   :alt: Gateway
   :align: center

At this point you will choose your selected host to use as an egress. You can choose if you would like to use NAT or not with the switch. You also have a choice of using this host as an internet gateway. more on that in a bit. You can put the selected CIDR for your egress range(s) in the field. click the add range button to add more egress ranges for the host. The interface is automatically chosen and will not be shown in this window. With everything filled out, click the create button.

.. image:: images/ui-6.png
   :width: 80%
   :alt: Gateway
   :align: center

Netmaker will set either iptables or nftables rules on the node depending on which one you have installed on your client. This will then implement these rules, allowing it to route traffic from the network to the specified range(s).


Use Cases
============

1) Remote Access
-------------------

A common scenario would be to combine this with an "Ingress Gateway" to create a simple method for accessing a home or office network. Such a setup would typically have only two nodes: the ingress and egress gateways. The Ingress Gateway should usually be globally accessible, which makes the Netmaker server itself a good candidate. This means you need only the Netmaker server as the Ingress, and one additional machine (in the private network you wish to reach), as the Egress.

.. image:: images/egress2.png
   :width: 80%
   :alt: Gateway
   :align: center

In some scenarios, a single node will act as both ingress and egress! For instance, you can enable acess to a VPC using your Netmaker server, deployed with a public IP. Traffic comes in over the public IP (encrypted of course) and then routes to the VPC subnet via the egress gateway.

.. image:: images/egress3.png
   :width: 50%
   :alt: Gateway
   :align: center

2)  / NAT Gateway
-----------------------

Most people think of a VPN as a remote server that keeps your internet traffic secure while you browse the web, or as a tool for accessing internet services in another country, using a VPN server based in that country.

These are not typical use cases for Netmaker, but can be easily enabled.

Navigate to the egress setup mentioned above. Instead of inputting a range, just click the internet gateway switch. the range of ``0.0.0.0/0`` will be automatically put in for you. (The IPv6 version ``::/0`` is still under construction) Click create.

.. image:: images/internet-gateway.png
   :width: 80%
   :alt: Internet Gateway
   :align: center

After that, your public traffic will be routed through your egressing client.


.. image:: images/egress5.png
   :width: 50%
   :alt: Gateway
   :align: center

Advanced Use Cases
======================

1) Segmenting Traffic Flow Through Egress Gateways

   User will need to add these additional routing rules on the egress machine to segment traffic to multiple egress ranges as desired when on multiple networks

.. code-block::
   
   iptables -t filter -N netmakeregress 
   iptables -t filter -I FORWARD -i netmaker -d egressGwRangeA,egressGwRangeB -j netmakeregress

   iptables -t filter -I netmakeregress -s networkRangeA -d egressRangeA -j ACCEPT
   iptables -t filter -I netmakeregress -s networkRangeB -d egressRangeB -j ACCEPT
   iptables -t filter -A netmakeregress -j DROP

   # NAT Rules

   iptables -t nat -I POSTROUTING -s networkRangeA -d egressGwRangeA -j MASQUERADE

   iptables -t nat -I POSTROUTING -s egressGwRangeA -d networkRangeA -j MASQUERADE

   iptables -t nat -I POSTROUTING -s networkRangeB -d egressGwRangeB -j MASQUERADE

   iptables -t nat -I POSTROUTING -s egressGwRangeB -d networkRangeB  -j MASQUERADE 