# Zangata #2 - Configure Docker Daemon to use a Proxy

There are times when you cannot make a connection with your docker daemon directly. In these times, 
use a ``proxy`` server. 

In this blog post, we are going through a simple configuration guide to open the gates of the docker world!

## Setup

First, we need to create a systemd drop-in directory for the docker service:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Now, depending on the proxy server you are using, if it's an ``http`` server create a file named ``http-proxy.conf``. If an ``https`` server, create ``https-proxy.conf``, if an ``ftp`` server, create ``ftp-proxy.conf`` and for ``no-proxy`` exclusions, create ``no-proxy.conf``.

Here are a couple of samples of the contents of these files:

- ``http-proxy.conf``
```
[Service] 
Environment="HTTP_PROXY="http://USERNAME:PASSWORD@[your.proxy.server]:[port]"
```

- ``no-proxy.conf``
```
[Service] 
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.yallalabs.com"
```

After setting up these config files, you should reload the docker daemon by running the below command:

```bash
sudo systemctl daemon-reload
```

Then, restart docker service:

```bash
sudo systemctl restart docker
```

Finally, you can check out the environment variables set in the configs by running the following command:

```bash
systemctl show docker --property Environment
```

That's it! Hope you can connect to the docker world now easily!