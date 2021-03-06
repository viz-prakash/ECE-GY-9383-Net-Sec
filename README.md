<h2><p align="center">Security & Privacy Issues in Home Networks Stemming from IoT Devices</p></h2>

## Motivation
Home networks (private networks) are considered safe in our traditional network security threat model. Safe in the sense that these are secure because the attacker can't reach them from the Internet and nobody can't infringe upon the user's privacy. On the privacy side, devices inside the home network are owned by users so they are trusted; devices outside the network (from the Internet) can't access the network so they can't impact users' privacy. On the security side, the network is secure for the same reason that it's unreachable from the outside. But now, with the pervasive adoption of IoT (smart) devices, things are different. IoT devices are capable of initiating network connections and can do so without user interaction, which puts them in the gray area of trusted and untrusted. Why? To start we have seen previous research pointing out that IoT devices are invading users' privacy.  They not only profile and track users but also transmit and share users' profiles, possibly for advertisement and content recommendation, see the paper [Watching You Watch: The Tracking Ecosystem of Over-the-Top TV Streaming Devices](https://hdanny.org/static/ccs-19.pdf). Further, having a smart device inside the home network which is capable of initiating network connections is like putting a digital tap that could infer or collect information by scanning for devices; then scan for services running on devices, and further communicate with the services that allow so on those devices. Because of these reasons home network is not private anymore -- it's infiltrated by devices that are capable of acting in ways that users might not intend it to and they have no guarantee from the vendor that it will not act unintended way. There are no regulations or policies which disallow or deter the vendors and assure the users, so they are left on vendors' choice. This whole argument for security and privacy in the home network set us up for looking at suspicious behaviors of IoT devices locally. Specifically, in this project we are looking for: 

1. On the privacy side, do IoT devices collect or infer information about the user by looking at another IoT device traffic/service/port in the home network? This is perfectly fine if the user gives the IoT device consent to do so but it's suspicious if a user didn't give permission. For example, a smart home assistant in a home network looks for printers without users' permission, if found, it checks for the status of the printer's cartridge or toner status and suggests the user buy one. We put this issue in the device-to-device communication category.

2. Also on the privacy side, if the user has given consent to IoT devices to talk to each other, do they communicate with each other using encryption? If so, how strong is the used encryption? Same for if an IoT device is talking to the host on the internet? An example for this would be - user having multiple devices from a vendor and setting the whole smart home ecosystem and the vendor asks the user to allow all the devices from the vendor to talk with each other. For example, the smart home ecosystem consists of a smart home assistant, TV  stick, thermostat, and camera, all from the same vendor. What if a user buys another device from another vendor and a set of devices from the first vendor communicates with each other without encryption? The second vendor's device could talk to the services hosted on the first vendor's device and it could collect or infer information about the service used by the user on that device. We put this issue in encryption in the IoT devices category.
 
3. On the security side, do IoT devices punch holes in the NAT ([using UPnP](https://information.rapid7.com/rs/411-NAK-970/images/SecurityFlawsUPnP%20%281%29.pdf)) to make themselves accessible from the internet? For example, a smart camera punching hole in the NAT so that it can stream the house video feed to the user when they are outside of the house. This suspicion is based on some evidence that IoT devices are doing so. If this happens, it breaks the traditional home network trust model because now the network is accessible from the Internet. And if the device accessible from the Internet is hosting a vulnerable service, attackers can exploit the vulnerability in the service and then compromise the device, and eventually move laterally to compromise the entire network. We put this issue in the external communication category.


## Methodology
We decided to use the IoT Inspector's crowdsourced anonymized labeled data from more than 50K separate installations to look for evidence of suspicious behaviors mentioned in the motivation. IoT Inspector doesn't capture full network traffic, it only captures the network tuple and some indicators like what port is client/server, no TCP/UDP payload. To be certain about the issues, once we find evidence of those suspicious behaviors in IoT Inspector data, we are going to create a list of devices showing suspicious behavior. Then, we are going to create a network in our lab with these devices and verify the suspicious behavior by capturing and analyzing their network traffic.

As mentioned earlier, IoT Inspector collects anonymized traffic and doesn't collect entire communication, we decided to look for evidence in the following way for all the three cases mentioned in the motivation: 

1. For device-to-device communication, the first step is to find if a device is a port scanning other devices in the network. From the anonymized partial traffic data we can only infer if the device is communicating with another device in the network.

2. For encryption in IoT devices, the first step is to find out what are some common ports/services/protocols used by IoT devices. We can infer the service from the port numbers, for example, if a device is using 443 it's probably using TLS, 80 then HTTP protocol. Once we get a list of common protocols being used, we will handpick interesting protocols and collect a list of devices using that protocol. This is possible from IoT Inspector data because volunteers label the devices when they install IoT Inspector, they can mislabel devices but for now, we are considering that as a ground truth because we are just looking for evidence and a list of devices to where to look suspicious activity. Once we have a list of devices, we will analyze their behavior with our lab setup. For this case, we will analyze the traffic to see what kind of security and privacy measures they are using while using network protocols.

3. For external communication, similar to the previous step, we first needed to find what are some common ports/protocols that are exposed by IoT devices which is also possible from IoT inspector data. First, we need to find evidence that a device is communicating with a device on the Internet. Further, to make sure that a port is accessible from the internet, we need to make sure that connection was initiated by a device on the Internet, which means the device in the home network is acting as a server and the external device is acting as a client. We can be sure of this if we see a large number of remote ports talking with a device local port -- for TCP/UDP connection client randomly selects a port to talk to the server at a fixed port. Further, just like the previous step, we will create a list of devices to add to our lab setup for analysis to verify this kind of behavior.

## Results
For each category of issues results are discussed below:

1. For device-to-device communication, we haven't figured out a way to find out if a device is doing a port scan from the IoT Inspector data. This work is still in progress.

2. For encryption in IoT devices, we characterized some common ports used by devices. By looking at the source port of a device and what are the remote ports this port is communicating with can give us an idea about if a port is acting as a server or a client, as discussed previously. If we augment this device source port and remote port connection with all the devices using the source port and all the devices using the remote port and see how much connectivity flows from a device using the source port to devices using the destination port, we can characterize the ports used by devices in the home networks for communication. This will help us find out a list of devices that talk with each other and what ports they communicate on. A chart of this is below. Using this chart we can find some of the most chatty devices and pick ones we find interesting for our lab setup. In the lab setup we can monitor their traffic and analyze the communication on interesting ports we find here. So far, some interesting ports we found are 55444, 10001, 443, 80, 445, 53, 554, 2002, 8899, 12300, 8009, 10101, 8088, and 1900, used by devices on the far-left and far-right column of the chart. 

![alt text](/output_files/plots/device_to_device_comm_sankey_plot.png)
*Device to device communication in IoT devices from the perspective of device local ports to characterize the ports used by devices when they communicate in the local network*

Let's look at the biggest device local port contributor port 55444. We can see that this port is used by Amazon-based devices for local communication, as it's used as both client and server ports. Some of the devices on the far left are: Amazon Alexa, Echo Show 5,  Amazon Echo Dot, Firestick, Ring Doorbell, etc, whereas on the far right side are also Amazon products. Similarly, we can conclude that port 2002 is used by a Google device. To analyze another port, let's look at the source port 10001. Devices using this as a source port are mostly Google devices, whereas remote ports are distinct, large, and very random, which suggests this port act as a client on these random port and server on devices using port 10001. Client devices talking to a Google port 10001 are Wyze Cam from fixed port 38622, and a few Google devices using fixed ports 46953, 59501, and a few more. 

Another big contributor source port is 1900. We can tell this port is used by Philips Hue Bridge, Wemo Smart Plug, from different clients because there are a large number of random remote ports. But it also has few fixed ports, which suggests these devices are talking on a mutually agreed port which is also an interesting find. 

Some device and port combinations directly show that the device is acting as a server on that port, for example,  Sonos port 12300 -- it communicates with a large number of random remote ports.

Some other interesting ports we can see in this chart are 443, 53, and 80. Let's look at port 443, another major contributor on the source port side. We can say this port is used by Philips Hue as a server and the device talking to it is Dyson Fan. Most probably a few more but we don't list all the devices attached to the remote port to keep the graph digestible but if we want to find out we could query on our data to find out exact devices talking with Philips Hue on port 443. Next, port 53 is an interesting find because we don't expect devices to act as DNS servers and send DNS queries in the local network to a local device (except for the DNS sink scenario). The biggest contributor to port 53 is a device named Ambient Weather from remote port 4096. We can't tell what kind of interaction this is but, this device and port would be a good candidate to put in our lab for analysis. Doing analysis like this, we can tell port 80 is also acting as a server on devices like Philips Hue, Canon Printer, Cam Vrum, etc. 

Another way to characterize the ports used by devices is by putting them in the below charts, we can quickly tell what port is used by what kind of devices to talk to what remote device on what port.

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_55444.jpeg)
*On the left, the remote ports communicate with port 55444, and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 55444*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_2002.jpeg)
*On left, the remote ports communication with port 2002 and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 2002*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_1900.jpeg)
*On left, the remote ports communication with port 1900 and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 1900*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_12300.jpeg)
*On left, the remote ports communication with port 12300 and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 12300*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_443.jpeg)
*On the left, the remote ports communicate with port 443, and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 443*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_53.jpeg)
*On the left, the remote ports communicate with port 53, and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 53*

![alt text](/output_files/plots/device_to_device_comm/images/device_to_device_comm_for_80.jpeg)
*On the left, the remote ports communicate with port 80, and what is the name of the devices on the remote port side. On the right, shows what devices are using the source port 80*

From this initial analysis, if we have to buy devices to analyze their traffic they would be following:
|Device | Port (default is source port)|
|:---:|:---:|
|Amazon devices: Alexas, Echo, Firestick, Ring, etc | 55444, 55443|
|Google devices: Google Home, Google Nest, Chromecast, Google WiFi, etc| 10001, 2002, 10101, 8009|
|Philips Hue and Philips Hue bridge| 80, 443, 1900|
|Printers like Samsung, Canon, Octoprint| 80|
|Wemo smart plug| 1900, 49152|
|Wyze camera| remote 10001|
|Sonos devices| 12300, 443, 80|
|Harmony Hub| 8088|
|Ambient Weather|remote 53|
|Dyson Fan| remote 443|
|Dlink Camera| 80| 
|Ubiquity AP| 443, 10001|
|Fritz!Box| 443|
|Sky HD| 49153|
|Alecto IP cameras| remote 53|
|Panasonic| remote 80|

3. For external communication - we have found a lot of devices exposing ports to the Internet and these ports are getting accessed by the hosts on the Internet. 

The below chart shows the external communication between the device's local port and remote ports used by the hosts on the Internet. It also shows the breakdown of user-assigned labels of IoT devices using that local port. In the chart, port 123 is communicating with only 4 remote ports and 36178 flows out of a total of 36180 flows that are going out of this port are going to port 123, which indicates that devices using this port are not acting as a server. Port 123 is used by NTP as per RFC standards, where both client and server use this port, which matches our hypothesis. Port 443, 80, 8080, and 22 are communicating with a large number of different remote ports, which means they are acting as a server.

![alt text](output_files/plots/external_comm_sankey_plot.png)
*External communication wrt. device local ports and device streamflow. Device local ports are in the middle and devices are on the left and remote port on right.*


Below charts are a breakdown of the number of flows remote ports contribute to a port for a few interesting ports. 

![alt text](/output_files/plots/external_comm/images/external_comm_for_123.svg)
*Number of remote ports communicating with device local port 123 on left and number of devices that use port 123 on right.*

![alt text](/output_files/plots/external_comm/images/external_comm_for_443.svg)
*Number of remote ports communicating with device local port 443 on left and number of devices that use port 443 on right.*

![alt text](/output_files/plots/external_comm/images/external_comm_for_80.svg)
*Number of remote ports communicating with device local port 80 on left and number of devices that use port 80 on right.*

![alt text](/output_files/plots/external_comm/images/external_comm_for_8080.svg)
*Number of remote ports communicating with device local port 8080 on left and number of devices that use port 8080 on right.*

![alt text](/output_files/plots/external_comm/images/external_comm_for_22.svg)
*Number of remote ports communicating with device local port 22 on left and number of devices that use port 22 on right.*

From this initial analysis of this issue, a list of devices we would go to by for lab verification are mentioned in the table below:

|Device | Port (default is source port)|
|:---:|:---:|
|D-Link Camera| 443|
|Synology DS and NAS| 443|
|WD NAS | 80|
|Space Monkey | 8080|
|Direct TV| 8080|
|TP link Mesh
|Printers| 389|
|Netgeat GS108T| 53| 
|Vizio TV| 80|
|Luxor devices| 80|
|Eon smart box| 445|
|Amazon Echo| 389|


To see the interactive graphs download the respective HTML files from the [directory](/output_files/plots/).

## Next steps

For each category we plan to take these steps:

1. Extract out a list of devices that have shown evidence of port scanning.

2. Refine the list of suggested devices in internal communication with another round of analysis. Then buy them and set them up in our lab and start capturing the traffic. Once we get the traffic we can analyze the traffic on interesting ports.

3. Same as step 2.

4. Also, we would like to analyze why so many IoT devices are communicating on port 123/NTP to outside hosts. If these devices are accessible from outside they could be used to launch DDoS amplification attacks. Our hunch, for now, would be that devices in the local network are initiating these connections.
