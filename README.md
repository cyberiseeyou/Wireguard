# Wireguard Configuration


## Server Configuration

---

The following should be done on the device you wish to use as your server (The one who's ip will be used instead when making queries)


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

When you create a wireguard connection, you are not just making a connection but rather creating an interface. It is good practice to name your wireguard config as the interface that it will be attached to. E.g. wg0, wg1 etc.

We will use wg0 for this example.

Create and open the config file in your favorite text editor. I will be using neovim.

````bash
nvim wg0.conf
````

```
[Interface]
Address = 192.168.1.1/24 # Be sure to write this in CIDR notation
ListenPort = 51820 # This will be the port that the server listens on
PrivateKey = ServersPrivateKey # See below for generating a private key

[Peer]
PublicKey = PeersPublicKey # This will be the public key that we will generate on the client device
AllowedIPs = 192.168.1.2 # The IP addresses that will be allowed when using that the set public key

# This is just additional functionality that allows the server to come up automatically upon reboot

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

Now on the client we will install wireguard the and change into the directory the same as the server.
and now create the file using neovim the same as above.

The Client configuration will look something like this:

```
[Interface]
Address: 192.168.1.2/32
PrivateKey: ClientPrivateKey

[Peer]
PublicKey = ServerPublicKey
AllowedIPs = 0.0.0.0/0 # You can use this or specify the IP address you put in the server's configuration.
Endpoint = ServersPublicIPAddress
PersistentKeepAlive = 25 # This helps to keep the connection alive
```

For more information, check out the documentation at https://www.wireguard.com/quickstart
