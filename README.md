# OpenVPN for Docker

OpenVPN server in a Docker container complete with an EasyRSA PKI CA.

#### Upstream Links

* Docker Registry @ [jasonyslow/docker-openvpn](https://hub.docker.com/r/jasonyslow/docker-openvpn)
* GitHub @ [jasonlow83/docker-openvpn](https://github.com/jasonlow83/docker-openvpn)

## Quick Start

* You can refer to [kylemanna/docker-openvpn](https://github.com/kylemanna/docker-openvpn) for the Quick Start guide.

## To use tls-crypt or tls-crypt-v2

* Set your Data Volume, VPN URL/IP and Port

      OVPN_DATA="/home/user/Docker/ovpn/data"
      OVPN_URL="vpn.myserver.com"
      OVPN_PORT="1194"

* Create the local data folder that will hold the configuration files
  and certificates.  The container will prompt for a passphrase to protect the
  private key used by the newly generated certificate authority.

      mkdir -p $OVPN_DATA

* Use the following when generating the ovpn server configuration (change the
  parameters according to your needs).
  
      docker run -v $OVPN_DATA:/etc/openvpn --rm jasonyslow/docker-openvpn ovpn_genconfig -u udp://$OVPN_URL$OVPN_PORT -a SHA256 -C AES-256-CBC -g AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305 -T TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256 -o tls-crypt
      docker run -v $OVPN_DATA:/etc/openvpn --rm -it jasonyslow/docker-openvpn ovpn_initpki

* If you wish to use tls-crypt-v2 use the following to generate the ovpn server config and create the tls-crypt-v2 server key.

      docker run -v $OVPN_DATA:/etc/openvpn --rm jasonyslow/docker-openvpn ovpn_genconfig -u udp://$OVPN_URL:$OVPN_PORT -a SHA256 -C AES-256-CBC -g AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305 -T TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256 -o tls-crypt-v2
      docker run -v $OVPN_DATA:/etc/openvpn --rm -it jasonyslow/docker-openvpn openvpn --genkey tls-crypt-v2-server > ./data/pki/tc-server.key

* Start OpenVPN server process

      docker run -v $OVPN_DATA:/etc/openvpn -d -p $OVPN_PORT:1194/udp --cap-add=NET_ADMIN jasonyslow/docker-openvpn

* Prepare clientName variable

      CLIENTNAME="ovpnClient-01"

* Generate a client certificate without a passphrase

      docker run -v $OVPN_DATA:/etc/openvpn --rm -it jasonyslow/docker-openvpn easyrsa build-client-full $CLIENTNAME nopass

* If your ovpn server is configured to use tls-crypt-v2, you'll need to generate a tls-crypt-v2 client key for each client.

      docker run -v $OVPN_DATA:/etc/openvpn --rm -it jasonyslow/docker-openvpn openvpn --genkey tls-crypt-v2-client --tls-crypt-v2 /etc/openvpn/pki/tc-server.key > ./data/pki/tc-$CLIENTNAME.key

* Retrieve the client configuration with embedded certificates

      docker run -v $OVPN_DATA:/etc/openvpn --rm jasonyslow/docker-openvpn ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn

## Next Steps

### Docker Compose

If you prefer to use `docker-compose` please refer to the [documentation](docs/docker-compose.md).

