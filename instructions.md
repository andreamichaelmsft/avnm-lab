# Welcome to our lab for virtual network management at scale with AVNM!

> Check out [Azure Virtual Network Manager's public documentation](https://learn.microsoft.com/en-us/azure/virtual-network-manager/) for more info! It also contains how-to guides for all available features in the Azure Portal if you encounter issues on the lab's steps.

## Check prerequisites and deploy the lab's ARM template.

Before you get started with this lab, you'll want to check whether you meet the following prerequisites:

1. [General prerequisites](https://learn.microsoft.com/en-us/azure/virtual-network-manager/create-virtual-network-manager-portal?tabs=manualmembership#prerequisites)

2. [Network group-related permissions](https://learn.microsoft.com/en-us/azure/virtual-network-manager/concept-network-groups#network-groups-and-azure-policy)

3. [Azure Policy permissions](https://learn.microsoft.com/en-us/azure/virtual-network-manager/concept-azure-policy-integration#required-permissions)

If you're confident you have the right set of permissions and access level to the subscription(s) where you want to test AVNM, then go ahead and deploy the [lab's ARM template](template.json). This template will deploy virtual networks and their subnets, a network security group, and virtual machines and their network interface cards.

> Remember to clean up these resources after your lab -- especially the VMs -- for security and cost purposes!

1. Log in to the Azure Portal and search for `Deploy a custom template`.

2. Select `Build your own template in the editor`, copy-paste the contents of the lab's ARM template, and select the **Save** button.

3. Select the desired subscription, resource group, and region where you want to deploy this template's resources, then **Review + create**.

## Create your Azure Virtual Network Manager instance.

You're tasked with seting up connectivity, security, routing, and more among several virtual networks. To achieve this, you need an instance of Azure Virtual Network Manager (AVNM) -- a network manager.

1. In the Azure Portal, search for `Network managers` and create a new one. Use the following values for the **Basics** tab's fields:

    | Field | Value |
    |------|------|
    | Subscription | `<Desired subscription>` |
    | Resource group | `<Desired resource group>` |
    | Region | `<Desired region>` |
    | Features | `<Select all options>` |

    > Your network manager can manage resources outside of the region it's created in!

1. Proceed to the **Management scope** tab and only add your current test subscription to this network manager's scope. This step defines the boundary around the network resource that this network manager will be able to manage.

    > Please **only add your lab's subscription** to the network manager's scope! Otherwise, you could impact your teammates' environments. In general, you can add several subscriptions and/or management groups to the network manager's scope.

1. Finish creating this network manager.

1. Navigate to your new network manager instance. This is where you'll group your network resources, configure your desired settings, and more!

---

## Segment your virtual networks into network groups.

Before diving into setting up connectivity, security, routing, and more across your network resources, you first need to group the 50+ virtual networks in your scope.

1. Inside your network manager, expand the **Settings** in the left-hand menu and navigate to the **Network groups** blade.

1. Create 3 network groups of **Virtual network** member type. One group will represent all the virtual networks within the subscription, another will represent trusted virtual networks, and the last will represent non-trusted virtual networks.

1. The network groups you created are currently empty. To populate them, you can manually add members, or you can dynamically add members with Azure Policy.
   
    1. **All virtual networks group**
       
        1. Navigate into this network group and select the **Create Azure Policy** button to automatically populate its members.
           
        1. Use the GUI to include any virtual network that belongs to the resource group where you deployed the ARM template. You can select the **Preview resources** button to check what members this Policy will pick up. There should be 51 virtual networks shown.
           
        1. Save and exit from this network group.
           
    1. **Trusted virtual networks group**
       
        1. Navigate into this network group and select the **Create Azure Policy** button to automatically populate its members.
           
        1. Use the GUI to include any virtual network that has a tag with the key value pair with **env** as the key and **Trusted** as the value. You can select the **Preview resources** button to check what members this Policy will pick up. There should be 25 virtual networks shown.
           
        1. Save and exit from this network group.
           
    1. **Non-trusted virtual networks group**
       
        1. Navigate into this network group and select the **Create Azure Policy** button to automatically populate its members.
           
        1. Use the GUI to include any virtual network that has a tag with the key value pair with **env** as the key and **nonTrusted** as the value. You can select the **Preview resources** button to check what members this Policy will pick up. There should be 25 virtual networks shown.
           
        1. Save and exit from this network group.

> It may take up to a couple minutes for the Azure Policy to fully populate the network groups with member virtual networks. You can proceed without needing to wait for network groups to be fully populated.

You're finished setting up your network groups! Now it's time to build your desired configurations for connectivity, security, and routing, and deploy them across your network groups.

---

## Set up a hub and spoke topology.

You want to build bi-directional connectivity between a hub virtual network and all of your trusted and non-trusted virtual networks as spokes. You also want your trusted virtual networks to be able to talk to one another (but _not_ with the non-trusted virtual networks). Depending on how many spoke virtual networks you have, this could be very tedious -- but with AVNM, this can be set up in just a few clicks!

1. Inside your network manager, expand the **Settings** in the left-hand menu and navigate to the **Configurations** blade.

1. Create a **Connectivity configuration**.
   
    1. On the **Topology** tab, create a **Hub and spoke** topology with the **hubVNet** as its hub.
       
    1. For the **Spoke network groups**, add your trusted virtual networks group and your non-trusted virtual networks group.
   
        > You may need to **zoom out (Ctrl -)** in order to add network groups on this tab.
    
    1. For your trusted virtual networks group, you'll want those virtual networks to be able to communicate directly to one another without needing to hop through the hub. **Enable direct connectivity** for this network group **across regions**.

        > This step builds a mini global mesh among the members of this network group. The trusted virtual networks can all talk to each other directly, but not to the non-trusted virtual networks.

    1. Check out the **Visualization** tab, then **Review + create** the configuration.
    
    1. Select **Create and start deployment** to get a headstart on pushing this connectivity to your virtual networks.
       
        1. In the **Deploy a configuration** pane, your connectivity configuration should be populated already. Select the region where you deployed the ARM template, then review and **deploy**.
           
        1. If the site brings you back to the **Configurations** page, **refresh** the pane, select your connectivity configuration, and select the **Deploy** button to deploy the configuration to the region where you deployed the ARM template.
        
        > Creating a configuration alone won't affect your target virtual networks. **You must deploy your configurations into your desired regions to take effect.**

You've built your desired network topology! Let's take a look at securing your virtual networks next.

---

## Secure your virtual networks with a baseline ruleset.

Your organization has identified some high-risk network ports that you need to block across all of your virtual networks. You also need to block additional ports specifically for your trusted virtual networks. 

1. Inside your network manager, expand the **Settings** in the left-hand menu and navigate to the **Configurations** blade.

1. Create a **Security admin configuration**.
   
    1. On the **Rule collections** tab, add a rule collection that will contain the security admin rules covering all your virtual networks.
       
        1. This rule collection should target the network group containing all the virtual networks.
           
        1. Add a security admin rule to **deny inbound TCP** traffic to the **destination ports 20-23**. Or if you'd like, you can create 4 separate rules for each destination port 20, 21, 22, and 23.
           
           > You do not need to include any network groups in the source or destination of the rule itself. By targeting the network group at the rule collection level, any rules defined in the rule collection will be applied onto the target network group.
           
        1. On the **Add a rule collection** pane, select the **Add** button to finish adding this complete rule collection to the security admin configuration.
           
    1. Add another rule collection that will contain an extra rule just for your trusted virtual networks.
       
        1. This rule collection should target the network group containing the trusted virtual networks.
           
        1. Add a security admin rule to **deny inbound TCP** traffic to the **destination port 445**.
           
        1. On the **Add a rule collection** pane, select the **Add** button to finish adding this complete rule collection to the security admin configuration.

        > See what you did here? You can associate rule collections with different network groups to achieve modularity for your security rules. This same mechanism can be used to provide "exceptions" in org-wide security rules to particular virtual networks.

    1. **Review + create** this configuration and select **Create and start deployment** to get a headstart on pushing these security admin rules to your virtual networks.
       
        1. In the **Deploy a configuration** pane, your security admin configuration should be populated already. Select the region where you deployed the ARM template, then review and **deploy**.

All of your virtual networks now have security guardrails! Downstream NSGs will not be able to conflict with these security admin rules, as traffic denied by the security admin rules will be dropped upon contact with those rules.

---

## Route non-trusted spoke-to-spoke traffic through an Azure Firewall.

Don't forget to route traffic between your non-trusted virtual networks through the Azure Firewall residing in your hub virtual network! 

1. Inside your network manager, expand the **Settings** in the left-hand menu and navigate to the **Configurations** blade.

1. Create a **Routing configuration**.
   
    1. On the **Rule collections** tab, add a rule collection that will contain the routing rule for your non-trusted virtual networks.
       
        1. This rule collection should target the network group containing non-trusted virtual networks.
           
        1. Add a routing rule to steer spoke virtual network traffic toward the hub virtual network's Azure Firewall.
           
            1. The **Destination** should describe the default route (**IP address 0.0.0.0/0**).
               
            1. The **Next hop** should be a **Virtual appliance** and its address will be `10.0.3.68`, which represents the Azure Firewall's IP address.
               
            1. **Add** the routing rule to the routing rule collection.
               
        1. On the **Add a rule collection** pane, select the **Add** button to finish adding this complete rule collection to the routing configuration.

    1. **Review + create** this configuration and select **Create and start deployment** to get a headstart on deploying these routing rules to your virtual networks' subnets.
       
        1. In the **Deploy a configuration** pane, your routing configuration should be populated already. Select the region where you deployed the ARM template, then review and **deploy**.

Upon deployment, AVNM will create the user-defined routes (UDRs) for all your non-trusted virtual networks' subnets. Traffic between these virtual networks will be routed through the IP address of the Azure Firewall in your hub virtual network.

> Network groups containing subnets can also be used with AVNM's routing configuration. Check out AVNM's public documentation or [UDR management blog](https://techcommunity.microsoft.com/blog/azurenetworkingblog/how-to-use-azure-virtual-network-managers-udr-management-feature/4129759) for more scenarios that AVNM's routing configuration can address.

---

## Manage the IP addresses of your virtual networks.

Now let's take a look outside of AVNM's group-configure-deploy mechanisms. AVNM's IP address management (IPAM) feature lets you create pools for IP address planning, automatically assign non-overlapping CIDR addresses to Azure resources, and prevent address space conflicts across on-premises and multicloud environments.

You're trying to plan for another hub and spoke topology between your hub virtual network and 5 spoke virtual networks. You know you'll also have to create a new virtual network and connect it to this topology. Let's walk through how IPAM can help you ensure there are no overlapping address spaces between the virtual networks that you want to connect in this topology, and even create a new spoke virtual network with guaranteed non-overlapping address space.

1. Inside your network manager, expand the **IP address management (Preview)** in the left-hand menu and navigate to the **IP address pools (Preview)** blade.

1. **Create** an IP address pool.
   
    1. On the **IP addresses** tab, specify the address space `10.0.0.0/16` that will cover the address space of 5 spoke virtual networks and hub virtual network.
       
    1. **Review + create** this IP address pool and select **Create**.

1. Navigate into your new IP address pool. Expand the **Settings** in the left-hand menu and navigate to the **Allocations** blade.

1. Let's associate 6 of your virtual networks -- 1 hub virtual network and 5 spoke virtual networks -- to this pool so you can check if there are overlaps in address space and monitor your IP utilization.
   
    1. Select the **Associate resources** button and associate any 5 of the spoke virtual networks and the 1 hub virtual network (named **hubVNet**).
       
    1. **Refresh** the pane.
    
    > Notice how you can also allocate from this IP address pool by carving out address space for child pools and for static CIDR blocks to represent on-premises or non-Azure resources.

1. What happens when you need to create another virtual network? During creation, you can actually set up the virtual network's IP address space from the available IP address space in this pool.
   
    1. Search in the Azure Portal for `Virtual networks` and create a new one in your resource group and region.
       
    1. On the **IP addresses** tab, check the box to **Allocate using IP address pools** and **Select an IP address pool** -- the one you just created!
       
    1. **Save** and **Review + create** this virtual network. 

In just a few minutes, you were able to create an IP address pool to track your hub and spoke virtual networks' IP address space, and even create a new virtual network from this IP address pool! You can also delegate IP address pools to non-AVNM users so they can create their virtual networks from their corresponding pool and enforce IP address management.

---

## Verify reachability between some of your spokes' virtual machines.

You just set your organization up for success with AVNM and its features! There are several moving parts between connectivity, security, routing, and resource-specific configurations. So how do you know that what you've set up in your Azure environment is actually achieving the reachability you desire among your network resources?

This is where AVNM's virtual network verifier tool can help us check this reachability. Whether you're troubleshooting traffic disallowance, diagnosing unexpected traffic allowance, or proving conformance to your organization's security requirements, virtual network verifier can provide the answers.

You're confident in your AVNM setup, but somehow some traffic still isn't being delivered between two of your spoke virtual networks' VMs. Let's use virtual network verifier to pinpoint where the issue lies.

1. Navigate back to your network manager by searching in the Azure Portal for its name or by searching for `Network managers`.

1. Inside your network manager, expand the **Virtual network verifier (Preview)** in the left-hand menu and navigate to the **Verifier workspaces (Preview)** blade.

1. **Create** a verifier workspace.

    > Did you know that **you can delegate verifier workspaces to non-AVNM users**? This will _not_ give them access to the parent network manager, but it will enable them to run reachability analyses on their resources that evaluate over the scope of the network manager -- all without elevating their permissions. For example, a delegated user can check why their VM can't reach the internet, and if a security admin rule coming from a network manager they don't have access to is denying that traffic, they'll still be able to see metadata about that security admin rule.

1. Navigate into your new verifier workspace and select the **Define a reachability analysis intent** button.

1. Create a **reachability analysis intent**. This is where you can describe the traffic details of the source-to-destination path you want to verify -- in this case, that of one VM to another VM, which reside in two of your trusted spoke virtual networks.
   
    1. The **protocol** of the path you want to check is **TCP**.
       
    1. The **source** and **destination types** should be **Virtual machines**.
       
        1. The **source** VM is **vm1** and the **destination** VM is **vm2**.
           
        1. The IP addresses should autofill after selecting the source and destination resources. If not, the **source IP address** is `10.0.49.4` and the **destination IP address** is `10.0.50.4`.
           
        1. The **destination port** should be `80`.
           
    1. **Create** this reachability analysis intent.

1. Inside your verifier workspace, expand the **Settings** in the left-hand menu and navigate to the **Reachability analysis intents** blade.

1. On the **Reachability analysis intents** blade, **Refresh** the pane, select the intent you just created, and **Start analysis**.
   
    1. Name the analysis and select the **Start analysis** button.

1. This analysis may take 1-2 minutes to process. **Refresh** the pane to see when the analysis is finished, at which point you'll see **View results** available for this intent.

1. **View results** of this intent's analysis run. A new pane should open where you can see a visualization of the reachability path of the intent you created.
   
    1. You can interact with this visualization by clicking on the resource icons and the paths between each node. This will open more details about the resource or step. You can also switch to the **JSON output** tab to see the full analysis result. 

**What's preventing vm1 from reaching vm2?**

> **Hint**: check out the visualization and select the path right before the traffic-blocked icon!

---

## Thanks for participating in our lab!
### Remember to clean up all network manager and template resources after your lab for security and cost purposes!
