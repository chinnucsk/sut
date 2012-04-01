sut, an IPv6 in IPv4 Userlspace Tunnel (RFC 4213)


## DEPENDENCIES

* https://github.com/msantos/procket

* https://github.com/msantos/pkt

* https://github.com/msantos/tunctl


## SETUP

* Sign up for an IPv6 tunnel with Hurricane Electric

    http://tunnelbroker.net/

* Start the IPv6 tunnel:

    * Serverv4 = HE IPv4 tunnel end

    * Clientv4 = Your local IP address

    * Clientv6 = The IPv6 address assigned by HE to your end of the tunnel

            sut:start([{serverv4, "216.66.22.2"}, {clientv4, "192.168.1.72"}, {clientv6, "2001:3:3:3::2"}]).

    * Set up MTU and routing (as root)

            ifconfig sut-ipv6 mtu 1480
            ip route add ::/0 dev sut-ipv6

    * Test the tunnel!

            ping6 ipv6.google.com


## EXPORTS

    start(Options) -> {ok, Ref}
    start_link(Options) -> {ok, Ref}

        Types   Options = [Option]
                Option = {ifname, Ifname}
                    | {serverv4, IPv4Address}
                    | {clientv4, IPv4Address}
                    | {clientv6, IPv6Address}
                    | {out, Fun}
                    | {in, Fun}
                Ifname = string() | binary()
                IPv4Address = string() | tuple()
                IPv6Address = string() | tuple()
                Fun = fun()
                Ref = pid()

        Starts a IPv6 over IPv4 configured tunnel.

        The default tun device is named "sut-ipv6". To specify the name,
        use {ifname, <<"devname">>}. Note the user running the tunnel
        must have sudo permissions to confifgure this device.

        {serverv4, Server4} is the IPv4 address of the peer.

        {clientv4, Client4} is the IPv4 address of the local end. If the
        client is on a private network (the tunnel will be NAT'ed by
        the gateway), specify the private IPv4 address here.

        {clientv6, Client6} is the IPv6 address of the local end. This
        address will usually be assigned by the tunnel broker.

        {in, Fun} allows filtering of IPv6 packets received from the
        network. All packets undergo the mandatory checks specified by
        RFC 4213 before being passed to user checks.

        {out, Fun} allows filtering of IPv6 packets received from the
        tun device.

        Filtering functions take 2 argments: the packet payload (a binary)
        and the tunnel state:

            -include("sut.hrl").

            -record(sut_state, {
                serverv4,
                clientv4,
                clientv6
                }.

        Filtering functions return ok to allow the packet. Any other
        return value causes the packet to be dropped. The default filter
        for both incoming and outgoing packets is a noop:

            fun(_Packet, _State) -> ok end.


    destroy(Ref) -> ok

        Types   Ref = pid()

        Shutdown the tunnel. On Linux, the tunnel device will be removed.


## TODO

* Support other checks required by RFC

* Support inbound/outbound IPv6 firewalling

* Decide how to handle write failures to the network and tun device

    * possible packets may fail before interface is fully configured
      (routes set up, etc)

* Make a firewall ruleset to Erlang compiler
