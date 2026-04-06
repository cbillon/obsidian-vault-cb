---
link: https://blogs.oracle.com/linux/introduction-to-netfilter
byline: Mohith Kumar Thummaluru
excerpt: An introduction Netfoler, the advanced networking framework in the Linux kernel.
tags:
  - network
  - ip
slurped: 2026-04-04T09:49
title: Introduction to Netfilter
---

## Introduction

In this page, we are going to understand the various features of netfilter hooks, their functionality, and how to utilize them.

Netfilter is a subsystem that was introduced in the Linux 2.4 kernel that provides a framework for implementing advanced network functionalities such as packet filtering, network address translation (NAT), and connection tracking. It achieves this by leveraging hooks in the kernel’s network code, which are the locations where kernel code can register functions to be invoked for specific network events. For instance, when a packet is received, it triggers a handler for the event and performs module specified actions.

## Background – Netfilter Architecture

The netfilter framework provides a powerful mechanism for intercepting and manipulating network packets in the Linux kernel. This framework has 2 components to it – Netfilter hooks and conntrack. In this blog, we will focus mostly on the netfilter hooks. Whilst there are different Netfilter hooks available for different protocols, in this blog we will concentrate on _IPv4_ netfilter hooks.

### Netfilter Hooks

Netfilter hooks are functions that are registered with the kernel to be called at specific points in the network stack. These hooks can be viewed as checkpoints in the different layers of the network stack. They allow the kernel modules to implement custom firewall rules, network address translation, packet filtering, logging, and more. Pre-defined functions that are performing tasks such as filtering and logging can be hot-plugged into the kernel, which makes them easier to use.

There are five predefined netfilter hook points:

- **NF_IP_PRE_ROUTING:** The callbacks registered to this hook will be triggered by any incoming traffic very soon after entering the network stack. This hook is processed before any routing decisions have been made regarding where to send the packet i.e. to check if this packet is destined for another interface, or a local process. The routing code may drop packets that are unrouteable.
- **NF_IP_LOCAL_IN:** The callbacks registered to this hook are triggered after an incoming packet has been routed and the packet is destined for the local system.
- **NF_IP_FORWARD:** The callbacks registered to this hook are triggered after an incoming packet has been routed and the packet is to be forwarded to another host.
- **NF_IP_LOCAL_OUT:** The callbacks registered to this hook are triggered by any locally created outbound traffic as soon it hits the network stack.
- **NF_IP_POST_ROUTING:** The callbacks registered to this hook are triggered by any outgoing or forwarded traffic after routing has taken place and just before being put out on the wire.

Each hook point corresponds to a different stage of packet processing, as shown in the diagram below:

![Hooks](https://blogs.oracle.com/wp-content/uploads/sites/49/2025/10/01_Hooks.png)

When a packet arrives at or leaves a network interface, it passes through each of these hook points in order. At each hook point, the kernel calls all the registered netfilter hook functions for that hook point. Each netfilter hook function can inspect the packet and decide what to do with it. The possible actions are:

- Accept: The packet is allowed to continue to the next hook point or the destination.
- Drop: The packet is silently discarded and no further processing is done.
- Queue: The packet is queued for userspace processing by a daemon such as iptables or nftables.
- Repeat: The packet is re-injected at the same hook point for another round of processing.
- Stop: The packet is accepted and no further processing is done.

### Netfilter APIs

One needs to call _nf_register_net_hook()_ with a pointer to a net structure (usually _&init_net_) and a pointer to the _nf_hook_ops_ structure. This will register the netfilter hook function with the kernel and make it active.

To unregister a netfilter hook function, one needs to call _nf_unregister_net_hook()_ with the same parameters.

One can attach multiple callback functions to the same hooks and perform different operations based on the requirement. Let’s try to visualize what this looks like.

![hookfn](https://blogs.oracle.com/wp-content/uploads/sites/49/2025/10/02_hookfn.png)

## Kernel Code analysis of NF Hooks

To understand the code flow of these netfilter hooks in the kernel, let’s take a simple example of the _NF_INET_PRE_ROUTING_ hook.

When an IPv4 packet is received, its handler function _ip_rcv_ is triggered as follows:

Source taken from `net/ipv4/input.c` in linux kernel [1].

/*
* IP receive entry point
*/
int ip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt, struct net_device *orig_dev)
{
    struct net *net = dev_net(dev);

    skb = ip_rcv_core(skb, net);
    if (skb == NULL) {
        return NET_RX_DROP;
    }

    return NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING, net, NULL, skb, dev, NULL, ip_rcv_finish);
}

In this handler function, you can see the hook is passed to the funciton _NF_HOOK_ . Now this triggers hook functions attached to _NF_INET_PRE_ROUTING_ and once they all are returned then _ip_rcv_finish_ would be called, which is again an argument to the _NF_HOOK_ callback.

Source taken from `include/linux/netfilter.h` in linux kernel [2].

static inline int NF_HOOK(uint8_t pf, unsigned int hook, struct net *net, struct sock *sk, struct sk_buff *skb, struct net_device *in, struct net_device *out, int (*okfn)(struct net *, struct sock *, struct sk_buff *))
{
    int ret = nf_hook(pf, hook, net, sk, skb, in, out, okfn);

    if (ret == 1) {
        ret = okfn(net, sk, skb);
    }
    return ret;
}

This returns 1, if the hook has allowed the packet to pass. The function _okfn_ must be invoked by the caller in this case. Any other return value indicates the packet has been consumed by the hook.

## Writing a Netfilter module

Now that we are clear on the concepts of netfilter hooks. Let’s understand how to use these hooks to perform a simple logging operation of packets in the network stack. To hot-plug the callbacks to these hooks, users need to write a kernel module that defines and registers one or more netfilter hook functions.

Source taken from `include/linux/netfilter.h` in linux kernel [2].

1. A netfilter hook function has the following prototype
    
    unsigned int nf_hookfn(void *priv, struct sk_buff *skb, const struct nf_hook_state *state);
    
    The parameters are:
    
    - **priv** – A pointer to private data that was passed when registering the hook function.
    - **skb** – A pointer to the network packet as a _sk_buff_ structure.
    - **state** – A pointer to a _nf_hook_state_ structure that contains information about the hook point, such as the network protocol, the network interface, and the routing information.
    
    The return value is one of the possible actions mentioned in the previous subsection
    
2. To register a netfilter hook function, using _nf_register_net_hook_
    
    int nf_register_net_hook(struct net *net, const struct nf_hook_ops *reg);
    
    Definition of _struct nf_hook_ops_ looks as follows:
    
    struct nf_hook_ops {
        /* User fills in from here down. */
        nf_hookfn *hook;
        struct net_device *dev;
        void *priv;
        u_int8_t pf;
        unsigned int hooknum;
        /* Hooks are ordered in ascending priority. */
        int priority;
    };
    
    Let’s look into the significant fields:
    
    - **pf** – The protocol family of the packets to intercept, such as PF_INET for IPv4 or PF_INET6 for IPv6.
    - **hooknum** – The hook point where the function should be called, such as NF_INET_PRE_ROUTING or NF_INET_LOCAL_OUT.
    - **priority** – The priority of the function within the same hook point. Lower values mean higher priority.
    - **hook** – A pointer to the netfilter hook function.
    - **priv** – A pointer to private data that will be passed to the hook function.
3. Initialize the nf_hooks_ops structure
    
    static struct nf_hook_ops *nf_tracer_ops = NULL;
    
    static unsigned int nf_tracer_handler(void *priv, struct sk_buff *skb, const struct nf_hook_state *state);
    static int __init nf_tracer_init(void) {
        nf_tracer_ops = (struct nf_hook_ops*)kcalloc(1, sizeof(struct nf_hook_ops), GFP_KERNEL);
    
        if(nf_tracer_ops!=NULL) {
            nf_tracer_ops->hook = (nf_hookfn*)nf_tracer_handler;
            nf_tracer_ops->hooknum = NF_INET_PRE_ROUTING;
            nf_tracer_ops->pf = NFPROTO_IPV4;
            nf_tracer_ops->priority = NF_IP_PRI_FIRST;
    
            nf_register_net_hook(&init_net, nf_tracer_ops);
        }
        return 0;
    }
    module_init(nf_tracer_init);
    
4. Unregister the netfilter hook while exiting the module
    
    static void __exit nf_tracer_exit(void) {
        if(nf_tracer_ops != NULL) {
            nf_unregister_net_hook(&init_net, nf_tracer_ops);
            kfree(nf_tracer_ops);
        }
    }
    module_exit(nf_tracer_exit);
    

Now lets put all the pieces together to make an useful kernel module that can trace the TCP traffic going out and coming in through the interface.

#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/netfilter.h>
#include <linux/netfilter_ipv4.h>
#include <linux/ip.h>
#include <linux/tcp.h>
#include <linux/udp.h>
#include <linux/string.h>
#include <linux/byteorder/generic.h>

static struct nf_hook_ops *nf_tracer_ops = NULL;
static struct nf_hook_ops *nf_tracer_out_ops = NULL;

static unsigned int nf_tracer_handler(void *priv, struct sk_buff *skb, const struct nf_hook_state *state) {
    if(skb==NULL) {
        return NF_ACCEPT;
    }

    struct iphdr * iph;
    iph = ip_hdr(skb);

    if(iph && iph->protocol == IPPROTO_TCP) {
        struct tcphdr *tcph = tcp_hdr(skb);

        pr_info("source : %pI4:%hu | dest : %pI4:%hu | seq : %u | ack_seq : %u | window : %hu | csum : 0x%hx | urg_ptr %hu\n", &(iph->saddr),ntohs(tcph->source),&(iph->saddr),ntohs(tcph->dest), ntohl(tcph->seq), ntohl(tcph->ack_seq), ntohs(tcph->window), ntohs(tcph->check), ntohs(tcph->urg_ptr));
    }

    return NF_ACCEPT;
}


static int __init nf_tracer_init(void) {

    nf_tracer_ops = (struct nf_hook_ops*)kcalloc(1,  sizeof(struct nf_hook_ops), GFP_KERNEL);

    if(nf_tracer_ops!=NULL) {
        nf_tracer_ops->hook = (nf_hookfn*)nf_tracer_handler;
        nf_tracer_ops->hooknum = NF_INET_PRE_ROUTING;
        nf_tracer_ops->pf = NFPROTO_IPV4;
        nf_tracer_ops->priority = NF_IP_PRI_FIRST;

        nf_register_net_hook(&init_net, nf_tracer_ops);
    }

    nf_tracer_out_ops = (struct nf_hook_ops*)kcalloc(1, sizeof(struct nf_hook_ops), GFP_KERNEL);

    if(nf_tracer_out_ops!=NULL) {
        nf_tracer_out_ops->hook = (nf_hookfn*)nf_tracer_handler;
        nf_tracer_out_ops->hooknum = NF_INET_LOCAL_OUT;
        nf_tracer_out_ops->pf = NFPROTO_IPV4;
        nf_tracer_out_ops->priority = NF_IP_PRI_FIRST;

        nf_register_net_hook(&init_net, nf_tracer_out_ops);
    }

    return 0;
}

static void __exit nf_tracer_exit(void) {

    if(nf_tracer_ops != NULL) {
        nf_unregister_net_hook(&init_net, nf_tracer_ops);
        kfree(nf_tracer_ops);
    }

    if(nf_tracer_out_ops != NULL) {
        nf_unregister_net_hook(&init_net, nf_tracer_out_ops);
        kfree(nf_tracer_out_ops);
    }
}

module_init(nf_tracer_init);
module_exit(nf_tracer_exit);

MODULE_LICENSE("GPL");

## Demo/Output

Once the above kernel module is built and inserted into the kernel, one can see the messages constantly getting logged into _/var/log/messages_ .

$ insmod nf_tracer.ko

$ tail -f /var/log/messages
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:55560 | dest : 127.0.0.1:41705 | seq : 2257457827 | ack_seq : 4262703343 | window : 20462 | csum : 0xfe28 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.211.14.255:22 | dest : 10.211.14.255:49827 | seq : 2433568599 | ack_seq : 2778679564 | window : 4944 | csum : 0x9dc | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.213.228.238:49827 | dest : 10.213.228.238:22 | seq : 2778679564 | ack_seq : 2433568599 | window : 2067 | csum : 0xc62 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.213.228.238:49827 | dest : 10.213.228.238:22 | seq : 2778679564 | ack_seq : 2433568599 | window : 2067 | csum : 0x1532 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:55560 | dest : 127.0.0.1:41705 | seq : 2257457827 | ack_seq : 4262703343 | window : 20462 | csum : 0xfe38 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:55560 | dest : 127.0.0.1:41705 | seq : 2257457827 | ack_seq : 4262703343 | window : 20462 | csum : 0xfe38 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.211.14.255:22 | dest : 10.211.14.255:49827 | seq : 2433568643 | ack_seq : 2778679616 | window : 4944 | csum : 0x9b0 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:41705 | dest : 127.0.0.1:55560 | seq : 4262703343 | ack_seq : 2257457843 | window : 512 | csum : 0xfe28 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:41705 | dest : 127.0.0.1:55560 | seq : 4262703343 | ack_seq : 2257457843 | window : 512 | csum : 0xfe28 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.213.228.238:49827 | dest : 10.213.228.238:22 | seq : 2778679616 | ack_seq : 2433568643 | window : 2067 | csum : 0xc02 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.213.228.238:49827 | dest : 10.213.228.238:22 | seq : 2778679616 | ack_seq : 2433568643 | window : 2067 | csum : 0x1451 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.211.14.255:22 | dest : 10.211.14.255:49827 | seq : 2433568643 | ack_seq : 2778679676 | window : 4944 | csum : 0x9b0 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.213.228.238:49827 | dest : 10.213.228.238:22 | seq : 2778679676 | ack_seq : 2433568643 | window : 2067 | csum : 0xc346 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:39690 | dest : 127.0.0.1:41705 | seq : 3243759386 | ack_seq : 309546900 | window : 512 | csum : 0xfe3b | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:39690 | dest : 127.0.0.1:41705 | seq : 3243759386 | ack_seq : 309546900 | window : 512 | csum : 0xfe3b | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 10.211.14.255:22 | dest : 10.211.14.255:49827 | seq : 2433568643 | ack_seq : 2778679736 | window : 4944 | csum : 0x9b0 | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:41705 | dest : 127.0.0.1:39690 | seq : 309546900 | ack_seq : 3243759405 | window : 512 | csum : 0xfe3f | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:55560 | dest : 127.0.0.1:41705 | seq : 2257457843 | ack_seq : 4262703343 | window : 20462 | csum : 0xfe3c | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:41705 | dest : 127.0.0.1:39690 | seq : 309546900 | ack_seq : 3243759405 | window : 512 | csum : 0xfe3f | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:55560 | dest : 127.0.0.1:41705 | seq : 2257457843 | ack_seq : 4262703343 | window : 20462 | csum : 0xfe3c | urg_ptr : 0
Jul 21 07:31:04 demo kernel: source : 127.0.0.1:39690 | dest : 127.0.0.1:41705 | seq : 3243759405 | ack_seq : 309546923 | window : 512 | csum : 0xfe28 | urg_ptr : 0

## Conclusion

This blog explains how you can see the network packets that are travelling across the linux kernels using your own tracers. You can also extend these modules and write your own kernel modules to filter packets based on you’re individual requirements. For instance, you can view network packets from a specific host, add filters for a specific network protocol, and so on.

## References

1. Kernel code – 5.4.17-2136.328.3.el8uek.x86_64 [net/ipv4/ip_input.c](https://github.com/torvalds/linux/blob/master/net/ipv4/ip_input.c)
2. Kernel code – 5.4.17-2136.328.3.el8uek.x86_64 [include/linux/netfilter.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/netfilter.h)