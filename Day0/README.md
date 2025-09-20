# Tool check #

### Yosys ###
```
sudo apt-get update
git clone https://github.com/YosysHQ/yosys.git
cd yosys
sudo apt install make (If make is not installed please install it)
sudo apt-get install build-essential clang bison flex \
 libreadline-dev gawk tcl-dev libffi-dev git \
 graphviz xdot pkg-config python3 libboost-system-dev \
 libboost-python-dev libboost-filesystem-dev zlib1g-dev
make config-gcc
make
sudo make install 
```
<img width="1207" height="595" alt="yosys" src="https://github.com/user-attachments/assets/c3341e94-19f2-4526-830b-6bf055cc3f13" />

### Iverilog ###
```
sudo apt-get update
sudo apt-get install iverilog 
```
<img width="1190" height="560" alt="iverilog" src="https://github.com/user-attachments/assets/d45a520b-a348-4f43-bfcf-a2d4c47b1d3d" />

### gtkwave ###
```
sudo apt-get update
sudo apt install gtkwave  
```
<img width="1165" height="714" alt="gtkwavem" src="https://github.com/user-attachments/assets/da2826ce-6077-4546-969c-e621469596d3" />

### ngspice ###
Download the tarball from https://sourceforge.net/projects/ngspice/files/ to a local directory, unpack it using: 

```
tar -zxvf ngspice-37.tar.gz
cd ngspice-37
mkdir release
cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install 
```
<img width="851" height="219" alt="ngspice" src="https://github.com/user-attachments/assets/093f6319-a721-464d-85bd-48746082e680" />

### magic ###
```
sudo apt-get install m4
sudo apt-get install tcsh
sudo apt-get install csh
sudo apt-get install libx11-dev
sudo apt-get install tcl-dev tk-dev
sudo apt-get install libcairo2-dev
sudo apt-get install mesa-common-dev libglu1-mesa-dev
sudo apt-get install libncurses-dev
git clone https://github.com/RTimothyEdwards/magic
cd magic
./configure
make
make install
```
<img width="1178" height="682" alt="magic" src="https://github.com/user-attachments/assets/a27866e2-3580-4aea-a78c-271eefc408cd" />

### OpenLANE ###
```
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o
/usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg]
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee
/etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot
# After reboot
docker run hello-world
```
- Docker printing Hello World
<img width="1189" height="407" alt="docker" src="https://github.com/user-attachments/assets/ec9983d5-184c-4100-aa60-03cb0bd60368" />
- Openlane Version Check
<img width="804" height="78" alt="openlane" src="https://github.com/user-attachments/assets/e8a42095-80f3-4307-b68f-a92d38514f7f" />

### Check dependencies ###
```
git --version
docker --version
python3 --version
python3 -m pip --version
make --version
python3 -m venv -h 
```
<img width="1192" height="726" alt="Screenshot from 2025-09-20 19-45-20" src="https://github.com/user-attachments/assets/a2bdcb3e-eac3-4594-b39c-df9228772feb" />
 
 ### Installs PDKs and Tools ### 
```
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane
cd OpenLane
make
make test 
```
- Test successful
<img width="1192" height="632" alt="Screenshot from 2025-09-20 20-07-56" src="https://github.com/user-attachments/assets/41cf6d67-7f5b-41d5-9fa4-16a2f78975a8" />



