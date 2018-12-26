# Route Configuration

This repository is a lab for NCTU course "Introduction to Computer Networks 2018".

---
## Abstract

In this lab, we are going to write a Python program with Ryu SDN framework to build a simple software-defined network and compare the different between two forwarding rules.

---
## Objectives

1. Learn how to build a simple software-defined networking with Ryu SDN framework
2. Learn how to add forwarding rule into each OpenFlow switch

---
## Execution

> TODO:
> * How to run your program?
```bash
# Change directory to where files at
$ cd src/

# Execute Topo First

# Execute SimpleTopo.py
$ [sudo] mn --custom SimpleTopo.py --topo topo --link tc --controller remote
# Or Execute topo.py
$ [sudo] mn --custom topo.py --topo topo --link tc --controller remote

# Then Execute Controller "at another terminal"

# Execute SimpleController.py
$ [sudo] ryu-manager SimpleController.py --observe-links
# Or Execute controller.py
$ [sudo] ryu-manager controller.py --observe-links 
```
> * What is the meaning of the executing command (both Mininet and Ryu controller)?
> * Show the screenshot of using iPerf command in Mininet (both `SimpleController.py` and `controller.py`)
>> SimpleController.py <br>
>> ![Screenshot_iPerf](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/iperf_cut.png) <br>
>> controller.py <br>
>> ![Screenshot_iPerf2](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/iperf2_cut.png)

---
## Description

### Tasks

> TODO:
> * Describe how you finish this work in detail

1. **Environment Setup**
    1. Login to my container using SSH
        1. Open the PieTTY ( Putty is also OK ) and connect to my container
            > * IP address : 140.113.195.69
            > * Port : 16309
        2. Login as root
            > * Login : root
            > * Password : cn2018
        3. For protecting my own work
            > ```bash
            > # Change password
            > $ passwd
            > Enter new UNIX password: <NewPassword> 
            > Retype new UNIX password: <NewPassword>
            > ```

    2. Clone my GitHub repository to "Route_Configuration"
        > ```bash
        > # Clone my GitHub repository to "Route_Configuration"
        > $ git clone https://github.com/nctucn/lab3-allen880117.git Route_Configuration
        > ```

    3. Run Mininet for testing
        > the first time we execute the mininet, we will face a problem, that is a service didn't start. <br>
        > This is the image from lab2.
        > ![Screenshot_problem_1](https://github.com/nctucn/lab2-allen880117/blob/master/screenshots/Screenshot_problem_1.png)

        > to solve the problem, just start the service
        > ```bash
        > # Start the service of Open vSwitch
        > $ [sudo] service openvswitch-switch start
        > ```    
    
2. **Example of Ryu SDN**
    1. Run Mininet topology 
    >Run `SimpleTopo.py` in one terminal `FIRST`
    > ```bash
    > # Change the directory into
    > # /root/Route_Configuration/src/
    > $ cd /root/Route_Configuration/src/
    > # Run the SimpleTopo.py with Mininet
    > $ [sudo] mn --custom SimpleTopo.py --topo topo --link tc --controller remote
    > ```
    > ![Mininet](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/Mininet.png) <br>
    > The error message,
    > ```bash
    > "Error setting resource limits. Mininet's performance may be affected."
    > ```
    > won't affect this lab.

    1. Run Ryu manager with controller
    > Run `SimpleController.py` in `ANOTHER` terminal
    > ```bash
    > # Change the directory into /root/Route_Configuration/src/
    > $ cd /root/Route_Configuration/src/
    > # Run the SimpleController.py with Ryu manager
    > $ [sudo] ryu-manager SimpleController.py --observe-links
    > ```
    > ![Ryu](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/Ryu.png)

    1. How to leave the Ryu controller
    > Leave SimpleTopo.py in one terminal `first`
    > ```bash
    > #Leave the Mininet CLI
    > mininet> exit
    > #Clean
    > $ [sudo] mn -c
    > ```
    > Then, leave SimpleController.py in another termianl.
    > ```bash
    > # Leave the Ryu-manager
    > Ctrl-z
    > # Make sure "RTNETLINK" is clean indeed
    > $ [sudo] mn -c
    > ```

3. **Mininet Topology**
    1. Build the topology via Mininet
    > Duplicate the example code `SimpleTopo.py` and name it `topo.py`
    > ```bash
    > # Make sure the current directory is /root/Route_Configuration/src/
    > $ cp SimpleTopo.py topo.py
    > ```

    1. Add the constraints
    > Follow the image shows below <br> 
    > ![Ryu_Topo](https://github.com/nctucn/lab3-allen880117/blob/master/src/topo/topo.png)
    > ```py
    > # Modify the code below to meet the requirements
    > # Add links into topology
    > self.addLink(s1, h1, port1=1, port2=1)
    > self.addLink(s3, h2, port1=1, port2=1)
    > self.addLink(s1, s3, port1=3, port2=2, bw = 10, delay = '5ms', loss = 2)
    > self.addLink(s1, s2, port1=2, port2=1, bw = 30, delay = '2ms', loss = 1)
    > self.addLink(s3, s2, port1=3, port2=2, bw = 20, delay = '2ms', loss = 1)
    > ```

    1. Run Mininet topology and controller
    > Run `topo.py` in one termial `FIRST`
    > ```bash
    > # Run the topo.py with Mininet
    > $ [sudo] mn --custom topo.py --topo topo --link tc --controller remote
    > ```
    > Then, run `SimpleController.py` in `ANOTHER` terminal
    > ```bash
    > # Run the SimpleController.py with Ryu Manager
    > $ [sudo] ryu-manager SimpleController.py --observe-links
    > ```
    > You can ping each link respectively by using the following command in the Mininet's CLI mode
    > ```bash
    > # Example of testing the connectivity between h1 and h2
    > mininet> h1 ping h2
    > ```

4. **Ryu Controller**
    1. Trace the code of Ryu controller
    > Below is the `only` part which we should modify.
    > ```py
    >class SimpleController1(app_manager.RyuApp):
    >   #...
    >   
    >   '''
    >   METHOD : switch_features_handler (@set_ev_cls)
    >   Handle the initial feature of each switch
    >   '''
    >   @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
    >   def switch_features_handler(self, ev):
    >
    >   #...
    > ```

    2. Write another Ryu controller
    > Duplicate the example code `SimpleController.py` and name it `controller.py`.
    > ```bash
    > # Make sure the current directory is /root/Route_Configuration/src/
    > $ cp SimpleController.py controller.py
    > ```

    3. Follow the forwarding rules in the below image and modify `controller.py`.
    > ![Forwarding](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/Forwarding.png)
    > ![msg1](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/msg1.png) <br>
    > ![msg23](https://github.com/nctucn/lab3-allen880117/blob/master/screenshots/msg23.png)

5. **Measurement**

### Discussion

> TODO:
> * Answer the following questions

1. Describe the difference between packet-in and packet-out in detail.
   
2. What is “table-miss” in SDN?
   
3. Why is "`(app_manager.RyuApp)`" adding after the declaration of class in `controller.py`?
   
4. Explain the following code in `controller.py`.
    ```python
    @set_ev_cls(ofp_event.EventOFPPacketIn, CONFIG_DISPATCHER)
    ```

5. What is the meaning of “datapath” in `controller.py`?
   
6. Why need to set "`ip_proto=17`" in the flow entry?
   
7. Compare the differences between the iPerf results of `SimpleController.py` and `controller.py` in detail.
   
8. Which forwarding rule is better? Why?

---
## References

> TODO: 
> * Please add your references in the following

* **Ryu SDN**
    * [Ryubook Documentation](https://osrg.github.io/ryu-book/en/html/)
    * [Ryubook [PDF]](https://osrg.github.io/ryu-book/en/Ryubook.pdf)
    * [Ryu 4.30 Documentation](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)
    * [Ryu Controller Tutorial](http://sdnhub.org/tutorials/ryu/)
    * [OpenFlow 1.3 Switch Specification](https://www.opennetworking.org/wp-content/uploads/2014/10/openflow-spec-v1.3.0.pdf)
    * [Ryubook 說明文件](https://osrg.github.io/ryu-book/zh_tw/html/)
    * [GitHub - Ryu Controller 教學專案](https://github.com/OSE-Lab/Learning-SDN/blob/master/Controller/Ryu/README.md)
    * [Ryu SDN 指南 – Pengfei Ni](https://feisky.gitbooks.io/sdn/sdn/ryu.html)
    * [OpenFlow 通訊協定](https://osrg.github.io/ryu-book/zh_tw/html/openflow_protocol.html)
* **Python**
    * [Python 2.7.15 Standard Library](https://docs.python.org/2/library/index.html)
    * [Python Tutorial - Tutorialspoint](https://www.tutorialspoint.com/python/)
* **Others**
    * [Cheat Sheet of Markdown Syntax](https://www.markdownguide.org/cheat-sheet)
    * [Vim Tutorial – Tutorialspoint](https://www.tutorialspoint.com/vim/index.htm)
    * [鳥哥的 Linux 私房菜 – 第九章、vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php)

---
## Contributors

> TODO:
> * Please replace "`YOUR_NAME`" and "`YOUR_GITHUB_LINK`" into yours

* [YOUR_NAME](YOUR_GITHUB_LINK)
* [David Lu](https://github.com/yungshenglu)

---
## License

GNU GENERAL PUBLIC LICENSE Version 3