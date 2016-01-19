
Docker container to send traffic for dev/troubleshooting to another container in docker using [TCPREPLAY](http://tcpreplay.synfin.net/wiki/tcpreplay)

tcpreplay is a great tool to modify and replay existing packets capture file (pcap) into the network
Unfortunately, it doesn't support to send traffic to the same host.

In a Docker environment, it's easy to have a dedicated container to replay packets capture file. That's the goal of this project.

# How to run

You can either :
- start the container in CLI mode and use tcpreplay from there
- execute tcpreplay directly at runtime
Either way, you'll need to mount the local directory where your capture files are into the container.  

> If you run the container with the option "-v $(pwd):/data", docker will automatically mount the local directory into the container.

Execute in CLI mode
```
docker run --rm -t -v $(pwd):/data -i dgarros/tcpreplay /sbin/my_init -- bash -l
```

Execute directly
```
docker run --rm -t -v $(pwd):/data -i dgarros/tcpreplay /usr/bin/tcpreplay --intf1=eth0 output.pcap
```

# How to send traffic to another container using tcpreplay

In order to send traffic to another Container with tcpreplay, you need to modify your existing capture file with the target container IP and Mac address.
tcpreplay come with a tool named **tcprewrite** to take care of that  

> Within a docker environment, IP and Mac addresses are allocated dynamically and it can be complicated to rewrite all capture files  with the right IP and MAc addresses.
> In order to workaround that, it's possible to use a Broadcast IP and mac addresses

Update IP and Mac addresses (you need to provide the previous dst IP)
```
tcprewrite --infile=input.pcap --outfile=output.pcap --dstipmap=<previous dest IP>:172.17.255.255 --enet-dmac=01:00:05:11:00:06 --fixcsum
```

Update destination port
```
tcprewrite --infile=input.pcap --outfile=output.pcap --portmap=50020:51020
```

Send traffic out at 100pps
```
tcpreplay --intf1=eth0  --pps=100 output.pcap
```

You can find some good information on :
- tcpreplay [http://xmodulo.com/how-to-capture-and-replay-network-traffic-on-linux.html]
- tcprewrite [http://tcpreplay.synfin.net/wiki/tcprewrite]
