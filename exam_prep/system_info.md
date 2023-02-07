## System Info

### Server 1
- Hostname: rhcsa9-server1
- IPv4 Method: Manual
- IPv4 Address: 192.168.55.71/24
- Gateway: 192.168.55.1
- DNS: 8.8.8.8
- IPv6 Method: Manual
- IPv6 Address: 2002:fe60:def0::55/64

### Server 2
- Hostname: rhcsa9-server2
- IPv4 Method: Manual
- IPv4 Address: 192.168.55.72/24
- Gateway: 192.168.55.1
- DNS: 8.8.8.8
- IPv6 Method: Manual
- IPv6 Address: 2002:fe60:def0::56/64

### Repo Information
- http://192.168.55.47/repo/BaseOS/
- http://192.168.55.47/repo/AppStream/
- https://download-ib01.fedoraproject.org/pub/fedora/linux/releases/37/Everything/x86_64/os/
  - Only use the Fedora repo for packages unavailable on BaseOS or AppStream
  - The only thing on the task list that the Fedora repo is required for is to install "star" (Unique Standard Tape Archiver)
  - You can leave the Fedora repo disabled and only point to it for the one installation. If your repo ID is "F37", you would run the following command:
  - `dnf install -y star --repo F37`