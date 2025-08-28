# Walkthrough for Nvidia Jetson Orin AGX
> [!CAUTION]
> This is a work in progress walkthrough, do not attempt to follow unless you are debugging

- Read each step carefully and do not blindly copy and paste
- Run the following in a bash shell on a non-privileged user, unless expicit stated otherwise.
- After flashing the Jetson Orin, perform the following steps on the Nvidia Jetson Orin AGX Node itself
## Initial Setup

Flash the Nvidia Jetson Orin AGX Jetpack 6.2.1 firmware via the sdk-manager from an external machine.
  
### Extras

Extra tools I installed; just in case this affects the repeatability of the install process

```bash
sudo apt-get install htop python3-pip tmux \
sudo pip3 install -U jetson-stats
```

### Testing Ollama

Run the following to confirm ollama was using GPU acceleration, you can view the system metrics via `sudo jtop`

```bash
sudo mkdir -p /mnt/ollama && sudo docker run -d --runtime nvidia -v /mnt/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

Pull and test

```bash
sudo docker exec ollama ollama pull gemma3:27b \
sudo docker exec ollama ollama pull gpt-oss:latest \
sudo docker exec ollama ollama run gpt-oss:latest write me an extended essay on wildlife in Africa.
```

## Volunteer Node Setup

### Install K3s

```bash
curl -sfL https://get.k3s.io | sh -
# Check for Ready node, takes ~30 seconds
sudo k3s kubectl get node
```

### Install Waggle OS (Distribution Specific Files)

Pull Setup files

```bash
mkdir -p ~/Downloads \
cd ~/Downloads \
git clone https://github.com/cleeuh/volunteer-node-setup
```

> [!NOTE]
> The build-rootfs.sh file indicates that it is used to create a new installation image/distribution, however I just ran it directly on the device (It should theoretically work the same). I ran the following as **root** user (i.e. sudo -i).  However, if there are no special env variables needed/conflicts provided by root user, running as `sudo ./build-rootfs.sh` should work the same.

```bash
cd ~/Downloads/volunteer-node-setup/setup && sudo ./build-rootfs.sh
```

### Install WES

Pull WES files

```bash
cd ~/Downloads \
git clone https://github.com/waggle-sensor/waggle-edge-stack
```

Temporary credential files generation

```bash
sudo touch /etc/waggle/cacert.pem \
sudo touch /etc/waggle/cert.pem \
sudo touch /etc/waggle/key.pem \
sudo touch /etc/waggle/ca.pem \
sudo touch /etc/waggle/ssh-key.pem \
sudo touch /etc/waggle/ssh-key-cert.pem
```

Configure your system.

> [!WARNING]
> I am not entirely sure of the schema or correctness of setting the ID or what the desired IPs should be. So they are left default and the ID has been changed to prevent any potential conflicts

```bash
export WAGGLE_NODE_ID=5555000000000001 \
export WAGGLE_BEEHIVE_HOST=10.31.81.200 \
export WAGGLE_BEEHIVE_RABBITMQ_HOST=10.31.81.200 \
export WAGGLE_BEEHIVE_RABBITMQ_PORT=30000 \
export WAGGLE_BEEHIVE_UPLOAD_HOST=10.31.81.200 \
export WAGGLE_BEEHIVE_UPLOAD_PORT=30002 \

cd ~/Downloads/waggle-edge-stack/kubernetes && sudo ./deploy-stack.sh
```
