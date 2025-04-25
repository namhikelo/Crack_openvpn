# Crack_openvpn

Run on server: Ubuntu 22.04

## Config IP forwarding:
```
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Install package
```
apt update && apt -y install ca-certificates wget net-tools gnupg
wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repository.asc
echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repository.asc] http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt-get update && apt-get -y install openvpn-as
apt-get install -y openvpn-dco-dkms
sudo apt install python3-pip unzip zip
```

## Stop services openvpnas
```
systemctl stop openvpnas
```

### Build
```
mkdir /root/crack && cd /root/crack
cp /usr/local/openvpn_as/lib/python/pyovpn-2.0-py3.10.egg{,.bak}
cp /usr/local/openvpn_as/lib/python/pyovpn-2.0-py3.10.egg .
unzip -q pyovpn-2.0-py3.10.egg
cd ./pyovpn/lic/
mv uprop.pyc uprop2.pyc
```

### Create file uprop.py
```
from pyovpn.lic import uprop2
old_figure = None

def new_figure(self, licdict):
    ret = old_figure(self, licdict)
    ret['concurrent_connections'] = 1024
    return ret

for x in dir(uprop2):
    if x[:2] == '__':
        continue
    if x == 'UsageProperties':
        exec('old_figure = uprop2.UsageProperties.figure')
        exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))
```

### Compile:
```
python3 -O -m compileall uprop.py && mv __pycache__/uprop.*.pyc uprop.pyc
cd ../../
zip -rq pyovpn-2.0-py3.10.egg ./pyovpn ./EGG-INFO ./common
mv ./pyovpn-2.0-py3.10.egg /usr/local/openvpn_as/lib/python/pyovpn-2.0-py3.10.egg
systemctl start openvpnas
```

### Info login openvpn

During the build process, there will be login information. Remember to pay attention to the terminal.
