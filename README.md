# Laptop-to-Home-Router

## DOCUMENTATION OVERVIEW
Early into 2026, I turned my laptop into my own personal router. I made sure to document and write down most of my experience, but I did not record everything. Working on the home lab is significantly more fun and engaging than writing documentation. Though as I return to my documentation to replicate the project on newer hardware that is not my laptop, I immediately run into context I am lacking. For now, this documentation will also serve as an additional good lesson for Future Me: Write EVERYTHING down next time!!!

## PROJECT OVERVIEW
Built a virtualized homelab on my laptop by using Proxmox VE and pfSense to simulate a real-world network. The project included VLAN segmentation, personalized firewall rules, and remote VPN access through pfSense. It also included DNS filtering with Pi-hole set in a container.

## NETWORK ARCHITECTURE
Visual Diagram:
![network diagram](Network_Diagram.png)

Written Diagram:
- ISP Modem ---> TP-Link Router ---> Laptop (Proxmox Virtual Env) ---> TP-Link Switch ---> Desktop
                          nic0 -> vmbr0                  expname -> vmbr1
                          
        	TP-Link Switch (Physical ethernet port) --> Desktop: 192.168.1.101
        	vmbr1 (VLAN 20) --> "testpfsense" VM: 192.168.20.20
        	vmbr1 (VLAN 40) --> "testpfsense2" VM: 192.168.40.40

## KEY FEATURES
- Pi-hole DNS Filtering
- VLAN Segmentation
- Firewall Rules & Port Forwarding
- Wireguard VPN for Remote Access
- IDS and IPS Functionality w/ Snort

## INSTALLATION & SETUP
### Proxmox Setup
1) Find a computer that can run Proxmox; I used my Lenovo laptop.
2) Install Proxmox VE onto a flash drive.
3) Load into the BIOS menu on the computer, do this by clicking the BIOS key.
	a) Unfortunately there is no universal BIOS key. My laptop was F12. My desktop is Del.
4) In the BIOS menu, choose to boot into the flashdrive instead of windows.
5) Here's an important step for laptop users: I wanted to be able to close the lid of my laptop without it turning off and losing connection. Here are the steps I took:
   - I used the shell on my web browser on my desktop to access my laptop without actually using it.
   - I then used the command "nano /etc/systemd/logind.conf" to find files that I could change.
   - I proceeded to uncomment and change the following: 
   	> "HandleLidSwitch=suspend" - changed suspend to ignore
   	> "HandleLidSwitchExternalPower=suspend" - changed suspend to ignore
   	> "HandleLidSwitchDocked=ignore"
   	> "LidSwitchIgnoreInhibited=yes"- changed yes to no
   - I then pressed Ctrl-X to save, Y to save the changes, and then Enter to get out.
   - Finally, I used the command "reboot" to reboot the system and finish saving these changes.
   
### pfSense Setup
1. Installed the pfSense ISO file onto my desktop
2. Uploaded the ISO file to the ISO images tab in proxmox.
3. Created a VM for pfSense and followed installation documentation.
   - https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-proxmox-ve.html
4. I did not create two new vmbridges under the host's network tab because I just do not have the 
   resources. I used vmbr0 and vmbr1 for WAN and LAN respectively.
5. Next, followed this youtube tutorial which was helpful.
   - https://www.youtube.com/watch?v=he3ENpMLMsc&list=PLKWBLTBSeulxUlna0czysruLsxiD2dvMJ&index=18&t=2031s
6. Ran into an issue upon trying to install Wireguard to pfSense. Refer to Issue #3 below.
7. I was configuring everything with Wireguard when I ran into a few hiccups:
   1) Setting up the VPN on my phone required two settings to be changed specific to my set up.
   2) Endpoint: [NOTPRIVATEIP]:51820. Endpoint: [PUBLICIP]:51820
   3) I found the public IP address by searching "what is my ip" in a search engine.
   4) Addresses cannot be 10.0.0.1/24, it needs to be 10.0.0.2/24.
   5) These changes established the handshake and then internet connectivity respectively.

## TESTING & RESULTS
1) Loaded pfSense onto a Virtual Machine using the Proxmox Virtual Environment
2) Set up two VM bridges as network devices: vmbr0 (default) and vmbr1 (created)
3) After installation and reaching the webpage, set the primary DNS server to pihole IP address
4) Created two VLAN interfaces with the tags 20 and 40.
5) Used these VLANs to set a static IP address to two test VMs.
   1) The "testpfsense" VM was set to a static IP address of 192.168.20.20, where upon boot up, the device had already switched to using that IP address
   2) The "testpfsense2" VM was set to a static IP address of 192.168.40.40. Achieved same behavior
     as "testpfsense"
6) Both virtual machines were unable to ping the other, therefore it was necessary to start creating firewall rules to ensure that the devices connected through LAN could contact and reach each other.
7) I also played around with setting rules that would block access to the VM machine connected to VLAN40 where the VM on VLAN20 was still able to reach the internet. 

## CHALLENGES & LEARNING MOMENTS
1) One major hurdle I ran into was troubleshooting update packages and the package manager in pfSense. Due to instructions I followed for setting my pi-hole as the DNS server, pfSense could not connect to the domain for the packages. I had deactivated DNS Resolver previously, and reactivating it fixed the issue I was experiencing.
   1) Commands used to check DNS: resolvectl status | grep "DNS", nmcli dev show | grep 'IP4.DNS'
2) For pihole: reserving a static IP address functions better than giving it a static IP in the device settings.
