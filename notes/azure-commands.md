# Explain each azure resource command
---

```
az --version
```
Use: this command ensures that the az-cli is installed.


```
az group create -n kubernetes -l eastus
```
Use: creates the resourse group -n kubernetes and in the eastus2 region


```
az account list-locations
```
Use: have azure list available regions to use if desired for above command (az group create)


```
az network vnet create -g kubernetes \
  -n kubernetes-vnet \
  --address-prefix 10.240.0.0/24 \
  --subnet-name kubernetes-subnet
```
Use: creates a vnet -n kubernetes-vnet in the resource group we created. assigns an ip range of 254 available hosts. also creates a subnet called kubernetes-subnet which we will use for the internal kubernetes networking.

```
az network nsg create -g kubernetes -n kubernetes-nsg
```
Use: create a network security group policy and apply it to the resource group we are using.

```
az network vnet subnet update -g kubernetes \
  -n kubernetes-subnet \
  --vnet-name kubernetes-vnet \
  --network-security-group kubernetes-nsg
```
Use: assign and update the subnet to use the nsg


```
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-ssh \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range 22 \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1000
```

```
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-api-server \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range 6443 \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1001
```
Use: both of the above create a firewall rule through the NSG that allows external SSH and HTTPS

```
az network nsg rule list -g kubernetes \
  --nsg-name kubernetes-nsg \
  --query "[].{Name:name, Direction:direction, Priority:priority, \   Port:destinationPortRange}" -o table
```
Use: shows a table of the rules we created, their priority, and port. for the Vnet we are using. 22 being ssh and 6443 being kubernetes over ssl.

```
20.231.220.123
```
Use: this is the public ip of the load balancer

```
az network public-ip list --query="[?name=='kubernetes-pip'].{ResourceGroup:resourceGroup, \
  Region:location,Allocation:publicIpAllocationMethod,IP:ipAddress}" -o table
```
Use: to find the ip of the load balancer. query example within

```
az vm availability-set create -g kubernetes -n controller-as
```
Use: this is used to create an availability set. this creates two compute instances which host the kubernetes control plane.

```
for i in 0 1 2; do
    echo "[Controller ${i}] Creating public IP..."
    az network public-ip create -n controller-${i}-pip -g kubernetes > /dev/null

    echo "[Controller ${i}] Creating NIC..."
    az network nic create -g kubernetes \
        -n controller-${i}-nic \
        --private-ip-address 10.240.0.1${i} \
        --public-ip-address controller-${i}-pip \
        --vnet kubernetes-vnet \
        --subnet kubernetes-subnet \
        --ip-forwarding \
        --lb-name kubernetes-lb \
        --lb-address-pools kubernetes-lb-pool > /dev/null

    echo "[Controller ${i}] Creating VM..."
    az vm create -g kubernetes \
        -n controller-${i} \
        --image ${UBUNTULTS} \
        --nics controller-${i}-nic \
        --availability-set controller-as \
        --nsg '' \
        --admin-username 'kuberoot' \
        --generate-ssh-keys > /dev/null
done
```
Use: bash script for configuring the 2 vms to host the control plane. first it creates a NIC with priv/pub ip addresses based on the controller #, then puts nic in the vnet and subnet we are using, enables ip-forwarding, and points to the load balancer. need to look into what the load balancer is doing here with the back-end pool thing. then each vm is created, with the image based off the ubuntu image we looked for. attaches it to the nic that was created for it (1 to 1), targets the controller as, sets nsg blank so the policy is enforced by the group, create kuberoot, then generates local ssh keys.

```
az vm availability-set create -g kubernetes -n worker-as
```
```
for i in 0 1; do
    echo "[Worker ${i}] Creating public IP..."
    az network public-ip create -n worker-${i}-pip -g kubernetes > /dev/null

    echo "[Worker ${i}] Creating NIC..."
    az network nic create -g kubernetes \
        -n worker-${i}-nic \
        --private-ip-address 10.240.0.2${i} \
        --public-ip-address worker-${i}-pip \
        --vnet kubernetes-vnet \
        --subnet kubernetes-subnet \
        --ip-forwarding > /dev/null

    echo "[Worker ${i}] Creating VM..."
    az vm create -g kubernetes \
        -n worker-${i} \
        --image ${UBUNTULTS} \
        --nics worker-${i}-nic \
        --tags pod-cidr=10.200.${i}.0/24 \
        --availability-set worker-as \
        --nsg '' \
        --generate-ssh-keys \
        --admin-username 'kuberoot' > /dev/null
done
```
Use: bash script to set up the worker hosts. creates public ip. creates nic. creates vm.