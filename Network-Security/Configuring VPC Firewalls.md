# Configuring VPC Firewalls

### Objective

This lab explores **Virtual Private Cloud (VPC)** networks in Google Cloud and demonstrates how to create and manage **firewall rules** to control traffic to and from VM instances.

By completing this lab, you will learn how to:

- Create **auto-mode** and **custom-mode** VPC networks.
- Launch VM instances in different networks and subnets.
- Understand **default firewall rules**.
- Create custom **ingress** (incoming) and **egress** (outgoing) rules.
- Use **priorities** and **tags** to control which rules apply.
- Allow and block traffic based on **protocol**, **port**, and **source IP**.

---

### 🧱 What is a VPC?

A **Virtual Private Cloud (VPC)** is a **virtual network** in Google Cloud that provides **IP address space**, **subnetworks**, **routing**, and **firewall rules** to control communication between your cloud resources.

Think of a VPC as your **own private data center in the cloud**, where you can define which machines (VMs) are allowed to talk to each other — and who from the outside world can access them.

---

### 🔐 What are Firewall Rules in Google Cloud?

Firewall rules act like **security gates** for your VPC networks. They define:

- Whether traffic is **allowed** or **denied**.
- Whether the rule is for **ingress (incoming)** or **egress (outgoing)** traffic.
- Which **ports**, **protocols**, and **IP ranges** are involved.
- Which **VM instances** are affected, based on **tags** or **service accounts**.

> 🔥 Google Cloud firewall rules are **stateful**: If a connection is allowed in one direction, the return traffic is automatically allowed.

---

### LAB BREAKDOWN

---

### Task 1: Create VPC Networks and Instances

#### What You Do:

You create:

- An **auto-mode VPC** called `mynetwork`
- A **custom-mode VPC** called `privatenet` with a subnet
- Several **VM instances** in each network to test firewall behavior

#### Commands Used:

#### Create an auto-mode VPC:

```bash
gcloud compute networks create mynetwork --subnet-mode=auto
```

> Auto-mode creates **one subnet per region**, each with a pre-defined IP range.

#### Create a custom-mode VPC:

```bash
gcloud compute networks create privatenet --subnet-mode=custom
```

> Custom-mode lets you define **your own subnets and IP ranges**.

#### Create a subnet in `privatenet`:

```bash
gcloud compute networks subnets create privatesubnet \
  --network=privatenet \
  --region=us-central1 \
  --range=10.0.0.0/24 \
  --enable-private-ip-google-access
```

> `--enable-private-ip-google-access` allows private VMs to access Google APIs.

#### Launch VMs in various networks:

```bash
gcloud compute instances create default-vm-1 \
  --machine-type=e2-micro \
  --zone=us-central1-a \
  --network=default
```

```bash
gcloud compute instances create mynet-vm-1 \
  --machine-type=e2-micro \
  --zone=us-central1-a \
  --network=mynetwork
```

```bash
gcloud compute instances create mynet-vm-2 \
  --machine-type=e2-micro \
  --zone=us-central1-b \
  --network=mynetwork
```

```bash
gcloud compute instances create privatenet-bastion \
  --machine-type=e2-micro \
  --zone=us-central1-a \
  --subnet=privatesubnet \
  --can-ip-forward
```

```bash
gcloud compute instances create privatenet-vm-1 \
  --machine-type=e2-micro \
  --zone=us-central1-a \
  --subnet=privatesubnet
```

---

### Task 2: Investigate the Default Network

#### Key Points:

- The **default network** comes with **predefined firewall rules**, such as:
  - `default-allow-ssh` – allows port 22 (SSH)
  - `default-allow-internal` – allows traffic between instances
  - `default-allow-rdp` – allows port 3389 (Remote Desktop)
  - `default-allow-icmp` – allows ping (ICMP)

#### Steps:

1. Connect to `default-vm-1` using the **SSH button**.
2. Test **internet access** by running:

   ```bash
   ping www.google.com
   ```

3. Delete the default instance and the **entire default network**.

#### Why Delete It?

The default network allows **too much open access** — not suitable for production environments.

---

### Task 3: Investigate Custom Networks – No Default Access

Custom VPCs don’t have any firewall rules by default, except:

- **Implicit deny all ingress**
- **Implicit allow all egress**

So, even if a VM is running, you **cannot connect to it** via SSH or ping **unless you create rules**.

#### Example:

Try SSH from Cloud Shell:

```bash
gcloud compute ssh mynet-vm-2 --zone=us-central1-b
```

❌ You’ll get an error — because no rule allows incoming SSH.

---

### Task 4: Create Custom Ingress Firewall Rules

#### Goal:

Allow SSH only from **your Cloud Shell IP**.

#### Steps:

1. Get your Cloud Shell IP:

   ```bash
   ip=$(curl -s https://api.ipify.org)
   echo "My External IP: $ip"
   ```

2. Create a firewall rule to allow SSH **only from your IP**:

   ```bash
   gcloud compute firewall-rules create mynetwork-ingress-allow-ssh-from-cs \
     --network=mynetwork \
     --action=ALLOW \
     --direction=INGRESS \
     --rules=tcp:22 \
     --source-ranges=$ip \
     --target-tags=lab-ssh
   ```

3. Add the `lab-ssh` tag to your VMs:

   ```bash
   gcloud compute instances add-tags mynet-vm-1 --zone=us-central1-a --tags=lab-ssh
   gcloud compute instances add-tags mynet-vm-2 --zone=us-central1-b --tags=lab-ssh
   ```

4. Now try SSH again:

   ```bash
   gcloud compute ssh mynet-vm-2 --zone=us-central1-b
   ```

✅ Success!

---

### Task 5: Ping Between VMs (Internal Communication)

By default, **internal traffic is denied** in custom VPCs. Even though both VMs are in the same network, **ping (ICMP) won’t work**.

#### Allow Internal Ping:

```bash
gcloud compute firewall-rules create mynetwork-ingress-allow-icmp-internal \
  --network=mynetwork \
  --action=ALLOW \
  --direction=INGRESS \
  --rules=icmp \
  --source-ranges=10.128.0.0/9
```

Now try to ping from `mynet-vm-1` to `mynet-vm-2`:

```bash
ping mynet-vm-2.us-central1-b.c.[PROJECT_ID].internal
```

✅ It should now work.

But try pinging the **external IP** of `mynet-vm-2` — it **won’t work**. That’s because the rule only allows traffic from internal IP ranges.

---

### Task 6: Using Firewall Rule Priority

Google Cloud Firewall rules have a **priority number**:

- **Lower numbers = higher priority**
- Rules are evaluated in order of priority

#### Step 1: Deny ICMP with higher priority

```bash
gcloud compute firewall-rules create mynetwork-ingress-deny-icmp-all \
  --network=mynetwork \
  --action=DENY \
  --direction=INGRESS \
  --rules=icmp \
  --priority=500
```

Now try ping again — ❌ It fails.

#### Step 2: Lower the priority of the Deny Rule (Make it weaker):

```bash
gcloud compute firewall-rules update mynetwork-ingress-deny-icmp-all --priority=2000
```

Now ping again — ✅ It works.

---

### Task 7: Configure Egress Firewall Rules

So far, you’ve controlled **incoming traffic**.

Now let’s block **outgoing** traffic (egress):

```bash
gcloud compute firewall-rules create mynetwork-egress-deny-icmp-all \
  --network=mynetwork \
  --action=DENY \
  --direction=EGRESS \
  --rules=icmp \
  --priority=10000
```

Try ping again — ❌ It fails.

> Even though the rule has a high number (low priority), it's the only egress rule for ICMP, so it's enforced.

---

### Conclusion

In this lab, you learned:

- How to **create and manage VPCs** in auto and custom modes.
- How to use **firewall rules** to control access.
- The difference between **ingress and egress** traffic.
- How to use **tags** and **priorities** to target specific instances and resolve rule conflicts.
- That **default behavior in custom VPCs is to deny ingress**, making them secure by default.
