<h2><p align="center">Security & Privacy Issues in Home Networks Stemming from IoT Devices</p></h2>

## Motivation
Home networks (private networks) are consider safe in our traditional network security threat model, means they are secure and attackers can't infringe upon users' privacy. To elaborate, on the privacy side - devices inside the home network are owned by users so they are trusted; devices outside the netork (from the Internet) can't access the network so they can't impact users' privacy. On the security side, network is secure for the same reason that attackers can't reach reach the network from the Internet. Now, with the pervasive adoption of IoT (smart) devices things are different. IoT devices are capable of initiating network connection and can do so without user interaction, which puts them the gray area of trusted and untrusted. Why? To start we have seen previous research pointing out that IoT devices not only profile and track users, but also transmit and share users' profile , possibly for advertisement and content recommendation, see the paper [Watching You Watch: The Tracking Ecosystem of Over-the-Top TV Streaming Devices](https://hdanny.org/static/ccs-19.pdf). Further, having a smart device inside the home network which is capable of initiating network connections is like putting a digital tap that could infer or collect information by scanning for devices, then scan for services running on devices, and further communicate with the services that allow so. Because of these reasons home network is not that private anymore. It's infilterated by a device which is capable of acting smart but user has no gurantee from the vendor that it will not act in a way user doesn't intend it to. There are no regulations or policies which disallow the vendors and assure the users, so they are left on vendors' choice. This set us up for looking suspicous behaviors of IoT devices in the home network, both on security, and privacy side. Specifically, what we were looking for during this project are: 

1. On the privacy side, do IoT devices collect or infer information about the user by looking at another IoT device in the home network? This is perfectly fine if user give the IoT device consent to do so but it's suspicious if user didn't give permission. For example, a smart home assistant in a home network looks for printers in the network without users' permission, if found, it checks for the status of printer's cartridge or toner status and suggests the user to buy one. We put this issue in device-to-deivce communication category. We decided to look for devices scanning for devices, then port scanning discovered deivces. 

2. Also on the privacy side, if user has given consent to IoT devices to talk to each other, do they communicate with each other using encryption? If so, how good is the used encryption? Same for if a IoT device is talking to the host on a internet? Example for this would be - user having multiple devices from a vendor and to set the whole smart home ecosystem vendor ask user to allow all the devices from the vendor to talk with each other, for example, it consist of smart home assistant, smart TV  stick, smart thermostat, and smart camera all from same vendor. What if user buys another device from another vendor and set of devices from the first vendor communicate with each other without encryption? Second vendor's device could talk to the services hosted on first vendor's device and it could collect or infer information about the service used by the user from first vendor. We put this issue in encryption in IoT devices category.
 
3. On the security side, do devices punch hole in the NAT ([using UPnP](https://information.rapid7.com/rs/411-NAK-970/images/SecurityFlawsUPnP%20%281%29.pdf)) to make themselves accessible from the internet? For example, smart camera punching hole in the NAT so that it can stream the house video feed to user when they are outside of the house. This conjecture is based on some evidence that IoT devices are doing so. If this happens, it breaks the traditional home network threat model becasue now network is accessible from the Internet. And if the device accessible from the Internet is hosting a vulnerable serivce, attackers can exploit the vulnerability in the service and then compromise the deivce, and further move laterally to compromise the entire network. We put this issue in extertal communication categorty.


## Methodology
Use the IoT Inspector's crowdsourced anonymyzed labled data from more than 50K separate installation to look for eveidence of suspicous behaviors mentioned in the motivation. IoT Inspector doesn't capture full network traffic, so once we find evidence of those suspicious behaviors in IoT Inspector data, we are going to create a list of devices showing suspicious behavior. Then, we are going to create a network in our lab with devices and verify the suspicious behavior by capturing and analyzing the traffic.

As mentioned earlier, IoT Inspector collect anonymized traffic and doesn't collect entire communication we decided to look for evidence in following way for all the three cases mentioned in the motivation: 

1. For device-to-device communication - first step is find if a deviec is port scanning other devices in the network. From the anonymized partial traffic data we can only infer if devie is talking to another device in the network (partial traffic data because payload of traffic is not collected).

2. For encryption in IoT devices - we first need to find out what are some common ports/services/protocols used by IoT devices. We can infer the service from the port numbers, for example, if a device is using 443 it's probably TLS, 80 then HTTP, etc. Once we get a list of common protocols being used, we will hand pick interesting protocols and collect a list of devices using that protocol. This is possible from IoT Inspector data because volunteers label the devices when they install IoT Inspector, they can mis-label but for now we are considering that as ground truth because we are just looking for evidences and list of devices to where to look suspicious activity. Once we have list of devices, next we will buy these devices and set them up in our lab to monitor their network activity and collect entire network traffic. Next, we analyze the traffic to see what kind of security and privacy measure they are using while using these protocols.

3. For external communication - similar to previous step, we first need to find what are some common ports/protocols that are exposed by IoT devices. This is also possible from IoT inspector data. First we need to find evidence that a device is communicating with a device on the Internet. Futher, to make sure that a port is accessible from the internet, we need to make sure that connection was initiated by a device on the Internet, which means then device in the home network is acting as server and external device is acting as a client. To be sure of this we expect to see large number of remote ports talking with a local port on the device.

## Results
For each category of issues results are below:

1. For device-to-deivce communication - we haven't figured out a way to find out if a device is doing port scan from IoT Inspector data. This work is still in progress.

2. For encryption in IoT devices - we are able to characterize some commong ports used by devices. Some interesting ports we found are ... 

3. For external communication - we have found a lot of devices exposing ports to the Internet and these ports are accessed by the hosts on Internet. 

## Next step

For each category we plan to do take these steps:

1. Extact out a list of devices that have shown evidence of port scanning.

2. Curate a list of devices using the interesting ports mentioned in result section for internal communication. Then buy them and set them up in our lab and start capturing the traffic. Once we get the traffic we can analyze the traffic on interesting ports.

3. Same as step 2.

