1. The topic of my presentation is Communication-aware and Energy Saving VirtualMachineAllocation Algorithm in Data Center

2. I will introduce my paper with the following points.

   ### **First Introduction**

3. Cloud computing has been extensively used with the maturity of virtualization technology. Users can customize their own cloud services with an on-demand, pay-as-you-go service model . Data centers can isolate applications from each another by using virtualization technology,thereby allowing multiple users to share their physical resources [2]. I will refer to the physical machine as PM，virtual machine as VM in the following.

4. Efficient VM allocation strategies are important for improving data center performance and increasing allocation efficiency.At present, an increasing number of studies on the placement strategy of VM are being conducted and different placement strategies focus on different optimization goals.as energy consumption,QOS。

   For example: Maximize Resource Utilization Placement Strategy Use minimal PMs hold all VMs，but theMinimize Communication Costs Placement Strategy Place VMs that communicate with each other on the same or adjacent PMs

5. that most data centers currently use a three-tier architecture to deploy network equipment (see left part of Fig. 1).However, the existing allocation algorithms have limited consideration for the communication delay between VMs caused by the data center topology.Therefore, this study formalizes the network communication problem of VMs in data centers by studying these centers network topology.

   ### second Research Content

6. There are two  Research Content about this paper

   - We propose a communication costs model based on network topology-aware. 

   - and We propose an energy saving allocation algorithm(ESBCA) based on communication-aware, It can reduce the communication costs of the data center while reducing energy consumption.

7. this paper uses an undirected graph to represent user requests as show figure five. A node denotes the VM applied by users. The value between these nodes is the traffic between VMs.And at the same time, we defined Communication distances matrix. the number of switches passingthrough the communication path is used as the communication distance between PMs.based on the Communication distances matrix ，We define the communication cost of two VMS as the product of traffic and distance,like Formulate 1,The total communication costof the user is the sum of the communication cost between the VMs used. Like formulate two

8. During the experiment we found With the operation of the communication aware algorithm, the VMs of the same user are gradually gathered to a PM,If the PM goes down, the QoS of that user will be greatly affected ,So We propose fault tolerance to set for each user’s request to limited The maximum number of VMs belonging to a user placed on the same PM to guarantee the reliability of the user’s services.

9. We proposed algorithm ESBCA is divided into two phases. migration phase and placement phase.During the migration process, a suitable VM is selected based on the communication cost to be migrated. For the placement process,a two-step placement algorithm is designed to reduce the communication cost while considering the energy consumption of data centers.

   First,it Slecet the target rack with the lowest communication cost after the migration VM is placed.

   Then,it Select the target PM that has the lowest energy consumption before and after VM placement and meets the fault tolerance from the target rack

   Finally , Migrate the VM to the target PM

   ### third  Experimental Simulation

10. Figure .7shows that the communication costs of ESBCA is evidently smaller than compare algorithms because the former completely considers the communication costs between VMs during migration and placement.Figure. 8 shows that the ESBCA can effectively reduce the number of VMs migrations,thereby resulting in efficient task completion.

11. Fig. 9 show that the proposed ESBCA is the most energy-efficient, The main reason is that in the phase ofselecting the target PM, we select the PM with the smallest increase in energy consumption, the reduction of traffic and the number of migration will lead to the reduction of energy consumption These factors lead to the good energy-saving effect of the ESBCA

    Fig. 10 shows the comparison of the communication costs of each algorithm at different thresholds. Evidently,the ESBCA algorithm has substantially lower communication costs than the compare algorithms regardless of thethreshold value.

    ### fourth Conclusion & Future work

12. This paper designs and implements energy saving allocation algorithm(**ESBCA**)
    based on communication-aware，to reduce  the energy consumption and communication costs of the data center 

    In the future, we will further study energy consumption and real communication
    scenarios of network equipment, such as data center switches.

13. My report is complete,thank you for your attention