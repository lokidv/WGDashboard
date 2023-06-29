sudo apt update -y && sudo apt upgrade -y
reboot
----------install wireguard----------
apt install wireguard -y
sudo nano /etc/sysctl.d/99-sysctl.conf >>> then:
net.ipv4.ip_forward=1
reboot
----------Generate server config file---------
wgconfgen script:
copy file to the server and go to directory
apt install python3-pip -y
pip install qrcode
python3 wgconfgen.py

[Interface]
Address =  172.16.0.1/24
ListenPort = 22450
PrivateKey = WK6+xMsTIu+s+1P4JiGHRPJJ5RQKIv2iHpNvxptUrW8=
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
SaveConfig = true
[Peer]
PublicKey = SZSBz5dhHtWKxET+VlJGLAndRxTPCH8tvwPNyVBuUnQ=
AllowedIPs = 172.16.0.2/32

-------- make a config file and enable wireguard service-----------
nano /etc/wireguard/wg0.conf
copy commands > cntl+x > y
systemctl enable --now wg-quick@wg0.service
systemctl restart --now wg-quick@wg0.service

-----------install dashboard----------------
be in root directory, then:
git clone -b v3.0.6 https://github.com/donaldzou/WGDashboard.git wgdashboard
cd wgdashboard/src

apt install gunicorn -y
sudo apt-get -y install python3-pip
pip install -r requirements.txt
apt install net-tools
ifconfig eth0

sudo chmod u+x wgd.sh
sudo ./wgd.sh install
sudo chmod -R 755 /etc/wireguard
./wgd.sh start

Access your server with port 10086 (e.g. http://your_server_ip:10086), using username admin and password admin. See below how to change port and ip that the dashboard is running with.

-------dashboard usage------------------

cd wgdashboard/src
-----------------------------
./wgd.sh start    # Start the dashboard in background
-----------------------------
./wgd.sh debug    # Start the dashboard in foreground (debug mode)
-----------------------------
./wgd.sh stop     # Stop the dashboard
-----------------------------
./wgd.sh restart  # Restart the dasboard

---------------------- creat dashboard service-----------
cd wgdashboard/src
nano wg-dashboard.service

[Unit]
After=netword.service

[Service]
WorkingDirectory=/root/wgdashboard/src
ExecStart=/usr/bin/python3 /root/wgdashboard/src/dashboard.py
Restart=always


[Install]
WantedBy=default.target

$ cp wg-dashboard.service /etc/systemd/system/wg-dashboard.service
$ sudo chmod 664 /etc/systemd/system/wg-dashboard.service
$ sudo systemctl daemon-reload
$ sudo systemctl enable wg-dashboard.service
$ sudo systemctl start wg-dashboard.service  # <-- To start the service
$ sudo systemctl status wg-dashboard.service
reboot

sudo systemctl stop wg-dashboard.service      # <-- To stop the service
sudo systemctl start wg-dashboard.service     # <-- To start the service
sudo systemctl restart wg-dashboard.service   # <-- To restart the service
