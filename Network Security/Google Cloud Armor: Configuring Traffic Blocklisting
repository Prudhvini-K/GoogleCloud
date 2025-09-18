# **Google Cloud Armor: Configuring Traffic Blocklisting**

### **Objective**

This document provides a detailed walkthrough of the "Configuring Traffic Blocklisting with Google Cloud Armor" lab from the Google Cloud Skills Boost platform. The primary objective of this lab is to demonstrate how to use Google Cloud Armor to protect an application by blocklisting malicious or unwanted IP addresses at the edge of Google's network.

By following this lab, you learn how to:

- Verify the deployment of a global Application Load Balancer.
- Create a test VM to simulate a client.
- Implement a Google Cloud Armor security policy to block traffic from a specific IP address.
- Inspect logs to confirm the policy's effectiveness.

### **What is Google Cloud Armor?**

Google Cloud Armor is a network security service that provides protection for your Google Cloud applications and websites against threats like Distributed Denial of Service (DDoS) attacks and other web-based attacks. It operates at the edge of Google's global network, allowing you to filter and block traffic before it ever reaches your virtual machine instances, saving valuable compute resources and ensuring the availability of your services.

### **Lab Breakdown**

The lab is divided into several tasks, each with a specific purpose.

#### **Task 1: Verify the Application Load Balancer is Deployed**

**What are we doing?**

We are checking that the Application Load Balancer, which is the front-end for our web application, is correctly set up and running. A load balancer is essential here because it distributes incoming traffic across multiple backend instances, ensuring high availability and performance. Google Cloud Armor policies are always attached to a backend service of an external Application Load Balancer.

**Commands Used:**

```bash
gcloud compute backend-services get-health web-backend --global
````

**Explanation:** This command checks the health of the instances that are part of the web-backend backend service. A backend service defines a group of instances that will receive traffic. The `--global` flag is crucial as it specifies that we are querying a global backend service, which is a component of a global Application Load Balancer. You should see three healthy instances, confirming the backends are ready to serve traffic.

```bash
gcloud compute forwarding-rules describe web-rule --global
```

**Explanation:** This command retrieves details about the web-rule forwarding rule. A forwarding rule directs traffic from a public IP address to the load balancer's target proxy. We run this command to find the IPAddress property, which is the public IP of our load balancer. We need this IP to test access to our application.

```bash
while true; do curl -m1 {IP_ADDRESS}; done
```

**Explanation:** This is a simple stress test to confirm the load balancer is actively distributing traffic. The `while true; do ... done` loop runs the `curl` command repeatedly. The `curl -m1` command attempts to fetch the webpage at the given IP address with a 1-second timeout. When you run this, you will see responses from different backend instances, proving that the load balancer is distributing the requests. Use CTRL+C to stop the command.

#### **Task 2: Create a VM to Test Access to the Load Balancer**

**What are we doing?**

We are creating a new Compute Engine VM instance. This VM will simulate a client from which we will later block traffic. This step is a critical part of the experiment as it provides a controlled environment to verify that our Cloud Armor policy works as intended.

**Commands Used:**

```bash
curl -m1 {IP_ADDRESS}
```

**Explanation:** Once the VM is created, we use this command from within its SSH session to confirm that it can successfully access the web application through the load balancer's IP. The output should show the web server's HTML content, confirming successful connectivity.

#### **Task 3: Create a Security Policy with Google Cloud Armor**

**What are we doing?**

This is the core task where we create and apply a security policy. The goal is to specifically block the IP address of the VM we created in the previous step. This simulates blocklisting a malicious actor.

**Steps Explained:**

* **Get the IP:** We first locate and copy the External IP address of the access-test VM. This is the IP we want to blocklist.

* **Create the Policy:** We navigate to Google Cloud Armor and create a new policy with the following key settings:

  * **Name:** `blocklist-access-test` - A descriptive name.
  * **Default rule action:** Allow - This is a crucial setting. It means the policy will by default allow all traffic. We will then add a more specific rule to block a single IP. This demonstrates a "deny specific traffic" strategy. Alternatively, you could set the default to Deny and only Allow traffic from trusted IPs.

* **Add a Rule:** We add a new rule with the following configuration:

  * **Mode:** Basic mode - For simple IP-based filtering.
  * **Match:** The External IP address of the access-test VM. This tells the policy what traffic to look for.
  * **Action:** Deny - This tells the policy what to do with the matching traffic.
  * **Response Code:** 404 (Not Found) - This is the HTTP response code that will be returned to the blocked client. It's a useful code as it doesn't give away information about why the request was blocked.
  * **Priority:** 1000 - Lower priority numbers are evaluated first. Cloud Armor evaluates rules from lowest to highest priority. By setting this to a low number (compared to the default), we ensure that this specific Deny rule is evaluated before the default Allow rule.

* **Attach the Policy:** Finally, we attach the newly created policy to the web-backend backend service. This step links the policy to our Application Load Balancer, making it active.

#### **Task 4: View Google Cloud Armor Logs**

**What are we doing?**

We are confirming that our security policy is working and that the blocked traffic is being logged as expected.

**Steps Explained:**

* **Re-run curl:** We go back to the access-test VM's SSH session and run the `curl -m1 {IP_ADDRESS}` command again. This time, because our policy is in effect, the output should be 404 Not Found. This is the ultimate verification that the blocklisting worked.

* **Check Logs:** We navigate to the Cloud Armor policy logs. We inspect the log entries and locate a log with a 404 status. Upon expanding the log, we can verify that the `httpRequest` was made from the IP address of our access-test VM, confirming that Cloud Armor successfully blocked the request and logged the event.

### **Conclusion**

This lab successfully demonstrates the power and simplicity of Google Cloud Armor for protecting your web applications. By the end, you have not only secured an Application Load Balancer from a specific IP address but also gained a deeper understanding of how Google Cloud Armor functions at the network edge to provide robust, configurable security.
