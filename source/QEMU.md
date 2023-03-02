<h1>Insatll QEMU on ubuntu 18.04.4</h1>

1. Insatll QEMU <br>
<code>sudo apt-get update</code> <br>
<code>sudo apt-get install qemu</code>

2. check version number <br>
<code> qemu-system-x86_64 --version </code>

3. Please make sure that ubuntu 14.04 ISO file is in ~/QEMU (ubuntu-14.04.iso) <br>
<code>mkdir ~/QEMU </code> <br>
<code>cd ~/QEMU </code> <br>


4. Create img file for new guest OS <br>
<code>qemu-img create -f qcow2 Ubuntu_14.04.img 40G</code> <br>

5. Install guest os <br>
<code>sudo qemu-system-x86_64 -hda Ubuntu_14.04.img -boot d -cdrom ubuntu-14.04.iso -m 512 -enable-kvm</code> <br>
folow the same steps for installing OS <br>
When system as for reboot, click "reboot now" and close the window when screen turns black <br>

6. Start ubuntu 14.04 <br>
<code>sudo qemu-system-x86_64 -m 1024 -machine ubuntu,accel=kvm -hda  Ubuntu_14.04.img</code> <br>


<h2> After installing you can add launch this os from launch pad itself </h2> 

1. create new desktop file <br> <br>
<code> sudo nano /usr/share/applications/ubuntu14.0.desktop</code> <br> <br>

2. Create new file inside ~/QEMU <br>
<code>nano ~/QEMU/run.sh </code><br>
Paste followinf code into that file <br>
<code>#!/bin/bash</code><br>
<code>sudo qemu-system-x86_64 -m 1024 -machine ubuntu,accel=kvm -hda  /home/\<user name\>/QEMU/Ubuntu_14.04.img </code><br><br>
  save the file <code> ctrl + X </code> <br>
  <code>chmod +x ~/QEMU/run.sh</code> <br>
  
3. Paste the following code into that file <br> <br>
<code> [Desktop Entry] </code> <br>
<code> Version = 14.04.0 </code> <br>
<code> Type = Application </code> <br>
<code> Terminal = false </code> <br>
<code> Name = Ubuntu 14.04</code> <br>
<code> Exec = /home/\<user name\>/QEMU/run.sh </code> <br>
<code> Icon = /home/\<user name\>/QEMU/icon/ubuntu.png </code> <br>
<code> Categories = Application; </code> <br> <br>

<body> save the file <code>ctrl+X</code> </body>



