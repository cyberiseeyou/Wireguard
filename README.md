# Wireguard Configuration

<!--toc:start-->

- [Wireguard Configuration](#wireguard-configuration)

  - [Server Configuration](#server-configuration)
  - [Generating the Private and Public keys](#generating-the-private-and-public-keys)
  - [Client Configuration](#client-configuration)
  - [Starting the wireguard server and client](#starting-the-wireguard-server-and-client)

  <!--toc:end-->

## Server Configuration

---

The following should be done on the device you wish to use as your server
(The one who's ip will be used instead when making queries)

NOTE: These commands are only for arch linux based systems.

Start by updating your packages

```bash
sudo pacman -Syu
```

Now install Wireguard

```bash
sudo pacman -S wireguard-tools
```

Now we will change to the sudo user and change directory tocreate the wireguard config.

```bash
sudo su -
cd /etc/wireguard
```

When you create a wireguard connection, you are not just making
a connection but rather creating an interface. It is
good practice to name your wireguard config as the
interface that it will be attached to. E.g. wg0, wg1 etc.

We will use wg0 for this example.

Create and open the config file in your favorite text editor. I will be using neovim.

```bash
nvim wg0.conf
```

```conf
[Interface]
Address = 192.168.1.1/24 # Be sure to write this in CIDR notation
ListenPort = 51820 # This will be the port that the server listens on
PrivateKey = ServersPrivateKey # See below for generating a private key

[Peer]
# This will be the public key that we will generate on the client device
PublicKey = PeersPublicKey
# The IP addresses that will be allowed when using that the set public key
AllowedIPs = 192.168.1.2

# This is just additional functionality that allows
# the server to come up automatically upon reboot

PostUp = sysctl -w net.ipv4_forward=1
PostDown = sysctl -w net.ipv4_forward=0
```

## Generating the Private and Public keys

---

This is how you generate the private and public key for the server

```bash
umask 077 # This is so the keys are not accessible to anyone else
wg genkey > privatekey
wg pubkey < privatekey > publickey
```

That's all you need to do to generate the private and public keys.
These should be saved in /etc/wireguard

## Client Configuration

---

Now on the client we will install wireguard the and change into the
directory the same as the server.
and now create the file using neovim the same as above.

The Client configuration will look something like this:

```conf
[Interface]
Address: 192.168.1.2/32
PrivateKey: ClientPrivateKey

[Peer]
PublicKey = ServerPublicKey
# You can use this or specify the IP address you put in the server's configuration.
AllowedIPs = 0.0.0.0/0
Endpoint = ServersPublicIPAddress
PersistentKeepAlive = 25 # This helps to keep the connection alive
```

## Starting the wireguard server and client

---

make sure you save both the client and server configs at /etc/wireguard and
make the filename wg0.conf. If later you want to use the same server but a
different connection you can create another file and name it wg1.conf, wg2.conf respectively.
Now starting with the server all you need to do is make sure that the port
you specified in the configuration file is forwarded to your server
from your router (Preferred), or open the port on your router.
Now we only need to run one command to start wireguard.

```bash
wg-quick up [ConfigurationFile|InterfaceName]

# Example:
wg-quick up /etc/wireguard/wg0.conf

# You can also do it like this:

wg-quick up wg0

# This will look in /etc/wireguard for a wg0 configuration

# NOTE: You can also name the interface something different like
# wireguard-interface all
# We need to do is rename the configuration file to
# wireguard-interface.conf and then run
# the wg-quick command as follows:

wg-quick up /etc/wireguard/wireguard-interface.conf

# Or:

wg-quick up wireguard-interface

# Again this will look in /etc/wireguard for a configuration file by that name
```

For more information, check out the documentation at [Wireguard Quickstart](https://www.wireguard.com/quickstart)
