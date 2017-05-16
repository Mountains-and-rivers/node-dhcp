
![DHCPD.js](https://github.com/infusion/DHCP.js/blob/master/res/logo.png?raw=true "JavaScript DHCP Server")

DHCP.js is a RFC compliant DHCP client and server implementation on top of node.js.


Motivation
===

A DHCP server can be used to configure the entire local network. Typical parameters that can be organized with a DHCP server are ip-addresses, gateways / router, DNS server and really a lot more. DHCP is quite old and well establishd solutions are on the market, commercially and open source - so why a new implementation?

I was searching for a minimalistic DHCP server, which is robust and highly configurable. The first problem I had was: I wanted to deliver an IP address to a Raspberry PI without static configuration right out of my Macbook. However, Apple made it almost impossible to configure the onboard DHCP-server with newer versions of OSX.

In times of **home automation** and **IoT**, I was thinking of a solution, which can trigger something when I come home. DHCP is a good idea here, since any device will broadcast to the network, as soon as it connects. So why not playing the imperial march when you come back home?

Another problem I had was, I wanted to query DHCP servers without actually change the local configuration.

These problems were the trigger to start reading the RFC's and the protocol is really not that complicated. As such, this project was born. 


Usage
===

When installed globally, DHCP.js provides two executables, a client `dhcp` and a server `dhcpd`. The client simply retrieves network configuration from a DHCP server and prints the configuration after a complete handshake. All additional fields can be specified as list of arguments:

```
# sudo dhcp hostname

// netmask :  255.255.255.0
// router :  192.168.1.1
// dns :  8.8.8.8, 8.8.4.4
// server :  192.168.1.1
// hostname :  web392
```

On the other hand, the server can be used to provide the data:

```
sudo dhcpd --range 192.168.1.2-192.168.1.99 --hostname web392 --server 192.168.1.1 --router 192.168.1.1
```

The more powerful interface however, is the JavaScript API.

Simple DHCP Server
---

```js
var dhcp = require('dhcp');

var s = dhcp.createServer({
  // System settings
  range: [
    "192.168.3.10", "192.168.3.99"
  ],
  static: {
    "11:22:33:44:55:66": "192.168.3.100"
  },

  // Option settings (there are MUCH more)
  netmask: '255.255.255.0',
  router: [
    '192.168.0.1'
  ],
  bootFile: function (req) {

    if (req.clientId === 'foo bar') {
      return 'x86linux.0';
    } else {
      return 'x64linux.0';
    }
  }
});

s.listen();
```

Any config directive can be a function, like illustrated with the bootFile directive for PXE boot. This way you get a fully programable DHCP server.

Simple DHCP Client
---

```js
var dhcp = require('dhcp');

var s = dhcp.createClient();

s.on('bound', function () {

  console.log("State: ", this._state);

  // Configure your host system, based on the current state:
  // `ip address add IP/MASK dev eth0`
  // `echo HOSTNAME > /etc/hostname && hostname HOSTNAME`
  // `ip route add default via 192.168.1.254`
  // `sysctl -w net.inet.ip.forwarding=1`

});

s.listen();

s.sendDiscover();
```

Sniff DHCP Traffic
---

For research purposes it's also possible to just get triggered when broadcast events occur. This way an own DHCP server can be implemented. It's also possible to just listen to the traffic in the network, without answering. This can be used to automate something, when a device enters the network: 

```
var dhcp = require('dhcp');

var s = dhcp.createBroadcastHandler();

s.on('message', function (data) {

  if (data.options[53] === dhcp.DHCPDISCOVER) {
    if (data.chaddr === '12-34-56-78-90-AB') {
      console.log('Welcome home!');
    }
  }
});

s.listen();
```





Installation
===
Installing DHCP.js is as easy as cloning this repo or use npmjs:

```bash
npm install --save dhcp
```

If command line tools `dhcp` and `dhcpd` shall be installed, npmjs can be used as well:

```bash
npm install dhcp -g
```


Troubleshooting
===

EACCESS Error
---

Since the programs need to bind to port 67 and 68, root privileges are required. If you want to use dhcp and dhcpd without root privileges, change the port to something above 1024.


No data is received
---

A broadcast is typically not spread across all interfaces. In order to route the broadcast to a specific interface, you can reroute 255.255.255.255.

Linux
---
```bash
route add -host 255.255.255.255 dev eth0
```

OS-X
---
```bash
sudo route add -host 255.255.255.255 -interface en4 
```


Testing
===
If you plan to enhance the library, make sure you add test cases and all the previous tests are passing. You can test the library with

```bash
npm test
```

Copyright and licensing
===
Copyright (c) 2017, [Robert Eisele](http://www.xarg.org/)
Dual licensed under the MIT or GPL Version 2 licenses.
