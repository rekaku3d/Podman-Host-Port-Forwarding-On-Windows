# Podman-Host-Port-Forwarding-On-Windows
Podman Host Port Forwarding or Expose On Windows Troubleshoot

Troubleshooting: Exposing Podman Container to LAN on Windows (WSL2)
If you're running Podman on Windows using WSL2 and your container is not accessible via your local network IP (e.g., http://192.168.x.x:8000), follow this guide to forward traffic from your Windows host to the container inside WSL2.
🧩 Problem
Even with ports: "0.0.0.0:8000:8000" or network_mode: host, the container is not reachable from other devices on the LAN. This is because WSL2 uses a virtual network interface, and the container is bound to a WSL2 IP, not your Windows host IP.

✅ Solution: Use netsh to Forward Ports
- Open PowerShell
- Find the WSL2 IP address: Open WSL2 (podman-machine-default) and run: 
  ip addr | grep eth0
- Look for an IP like example :172.18.57.111.
- Forward port 8000 from Windows to WSL2: Open PowerShell as Administrator and run:
  
```cmd
netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=172.18.57.111
```

- Allow the port through Windows Firewall:
New-NetFirewallRule -DisplayName "Allow Port 8000" -Direction Inbound -LocalPort 8000 -Protocol TCP -Action Allow
- Test from another device: Open a browser and go to:
http://<your-windows-LAN-IP>:8000
- Example:
http://192.168.0.163:8000



🧼 Optional: Remove Port Forwarding
To remove the forwarding rule later:
netsh interface portproxy delete v4tov4 listenport=8000 listenaddress=0.0.0.0

🙂‍↔️ bat script

```cmd
@echo off
REM Replace with your Podman VM IP
set VM_IP=192.168.127.2

echo Binding ports to Podman VM at %VM_IP%...

netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=%VM_IP%
netsh interface portproxy add v4tov4 listenport=3000 listenaddress=0.0.0.0 connectport=3000 connectaddress=%VM_IP%
netsh interface portproxy add v4tov4 listenport=9000 listenaddress=0.0.0.0 connectport=9000 connectaddress=%VM_IP%

echo Adding firewall rules...

netsh advfirewall firewall add rule name="Podman Port 8080" dir=in action=allow protocol=TCP localport=8080
netsh advfirewall firewall add rule name="Podman Port 3000" dir=in action=allow protocol=TCP localport=3000
netsh advfirewall firewall add rule name="Podman Port 9000" dir=in action=allow protocol=TCP localport=9000

echo Done. Ports 8080, 3000, and 9000 are now forwarded to %VM_IP%.
pause


```



🧠 Notes
- Ensure your app inside the container is listening on 0.0.0.0, not localhost.
- This method works for any port and can be repeated for multiple services.
- You can automate this setup with a PowerShell script for convenience.
