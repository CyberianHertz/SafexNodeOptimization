# SafexNodeOptimization
Tutourial to get the most out of your Safex mining machine

**How I squeezed blood from a stone, or, "overclocking your virtual machine for fun... and hopefully profit!"**

**Synopsis**: Get more hashrate out of your Safex mining rig!

**Requirements**: Ubuntu 18.04 running a minimum 4GB of RAM (preferably on a virtual machine) and with safexd 5.0.0 or xmrig already installed, configured, and running. Any configurations not following the requirements will not be able to follow along exactly.

**Disclaimer**: I cannot guarantee this will improve your hash rate, and I cannot guarantee this will not ruin your machine. I highly suggest you not have a wallet on the machine you are mining with. This tutorial works best on a virtual machine that can be easily spun back up if something goes haywaire or if the machine becomes unstable as a result of entering these commands. I cannot be held responsible for any data loss or loss of cryptocurrency as a result of following this tutorial. While everything we are doing is fairly basic, use this information at your own risk and discression.
		
**Tutourial Note**: I am being extra verbose here to give Linux newbies a chance to learn and understand what they are doing along the way, *and I thank seasoned Linux users for coping with the long-windedness.* Ok, here we go!

Before starting, and before exiting your miner (if it is running), take note of your hashrate by typing status and hitting enter (for safexd) or pressing h (in xmrig) to compare it to the hash you get after making these changes.
Now, before proceeding, exit your miner by typing status and hitting enter (for safexd) or pressing h (in xmrig).

## Getting setup to survive a disconnect
Connect to your VM via ssh (I use Remina on Linux)
1. Log into your VM and at the prompt type `tmux` and press enter. This creates a session that will continue running so you can you can disconnect from it.
2. To disconnect from the tmux, **hold CTRL and press b** then release. This sequence puts you in tmux command mode. Note: you won't see any change in tmux command mode.
3. Now that you are in tmux command mode, **hold down SHIFT and press d** then press `<ENTER>`to close the tmux session and be dropped back into your shell.
4. To renter the running tmux session, type `tmux attach -t 0` and you be where you started.
	1. Its always good to start everything in tmux to avoid loss of work in case your internet connection drops out. Without tmux, all jobs initiated at the shell will be lost upon network connection close.
	2. You can read more about working with tmux at https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/

## Configure tmux with two panes for ultimate perfomance visibility
1. Now in tmux at a shell promt, find your safexd or xmrig and run it like you normally would.
2. Once the above is running as you like, **hold CTRL and press b** and then let go to enter tmux command mode (like above) then **hold down SHIFT and press %** to create a new pane.
4. In this new pane opens up a new shell prompt while your miner is running in the other pane! Now, type `htop` and press enter. htop is a good performance monitor with low overhead.
5. Optional (change htop colors): Now in htop use the `F2` key to enter htop setup. In setup you use arrow keys to move around, spacebar to make a change, and Esc key to exit setup. I change my htop colors to "Black Night".
6. Optional (Sort processes into a tree): Back at htop main screen, press the `F5` key to sort processes, and use arrow down to highlight the main thread of your mining software for ease of visibility once we leave htop.
7. Once htop is configured to your liking, switch back to the other tmux pane where your miner is running by **holding CTRL and pressing b** and let those go and use arrow left key to refocus cursor there. Now htop is doing its thing while you can control your stuffs.
8. Watch how htop displays info about your hardware and watch the bar graphs, especially in the next steps.

## Setup Swap File for system stability
This will extend a defined area of the machine's hard drive to act as RAM. Since I am running only 4GB of RAM and want to turn on 4GB huge pages in the next step, I want to make sure the system doesn't crap out when the miner steals all the RAM. The easiest way to do this is by creating a shell script, authorizing it to execute, and executing it. Lets do it!
1. Go home by entering `cd ~` at the shell prompt.
2. Create a directory for scripts and switch to it by entering `mkdir scripts && cd scripts` at the shell prompt
3. Create the makeSwap script by entering `nano makeSwap.sh` at the shell prompt, which will open a blank Nano editor.
4. Copy the 11 lines of code below and paste it into Nano (probably via CTRL+SHIFT+V or right-click paste):
```
#!/bin/bash
## Create Swap Space on Vultr Instances
## To create a 4 GB swap file enter the following command and hit enter.
dd if=/dev/zero of=/swapfile count=4096 bs=1M
## Activate Swap File
chmod 600 /swapfile
mkswap /swapfile
## Enable Swap File
swapon /swapfile
## Enable swap on system reboot
sudo echo "/swapfile none swap sw 0 0" >> /etc/fstab
## end
```
5. Once pasted, press **CTRL+X** to exit Nano edit mode, press **y** to save, and hit **ENTER** to save the code to the file *makeSwap.sh* and be dropped back to the shell prompt.
6. Now type `chmod u+x makeSwap.sh` at the shell promt and hit **ENTER** to make *makeSwap.sh* executable.
7. Execute it and make your swap file by typing `./makeSwap.sh` and hitting **ENTER**. Watch htop and you'll see the CPU usage rise for a moment and the 4GB of swap space appear under the RAM graph.

## Setup huge pages
1. Stop your mining software (type `exit` and press enter for safexd or `CTRL + C` for xmrig). Watch htop graphs before and after doing it so you know how the graphs work. 
2. Type `cd ~` and press **ENTER** to enter your home directory (I do this as a best practice to avoid putting something somewhere I didn't mean to and losing it later).
3. Check to see if huge pages exist by entering `cat /sys/devices/system/node/node*/meminfo | fgrep Huge`
	1. If huge pages aren't present, you'll see something like below and will need to continue to step 4.
				<br/>Node 0 AnonHugePages:         0 kB
				<br/>Node 0 ShmemHugePages:        0 kB
				<br/>Node 0 HugePages_Total:     0
				<br/>Node 0 HugePages_Free:      0
				<br/>Node 0 HugePages_Surp:      0
4. At the shell, enter `sudo sysctl -w vm.nr_hugepages=4096` to setup 4GB huge pages. This allocates 4GB of RAM in a chunk. Its all or nothing now RAM-wise, and without the swap file you could experience degraded performance or an unresponsive system. Watch your htop pane to see the CPU in action and the swap file to appear.
	<br/>For larger or smaller allocations, refer to:
	<br/>https://medium.com/@tomas_savenas/30-increase-in-cpu-mining-hash-rate-by-enabling-huge-pages-8af5eedb7d62
	<br/>and
	<br/>https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html	
5. Now we want to write the huge pages change to the system so they survive a reboot. At the shell prompt, enter `sudo bash -c "echo vm.nr_hugepages=4096 >> /etc/sysctl.conf"`
6. Check that the huge pages were activated by checking again with `cat /sys/devices/system/node/node*/meminfo | fgrep Huge`

## Test it!
1. If you followed along, you are ssh'd into your VM and have two tmux panes open, one with a shell promp and the other with htop.
2. Navigate to your mining software (safexd or xmrig) and start it as you normally would.
3. Watch to see the htop graphs fill up for RAM as the miner begins its work. In xmrig you will see that hash pages are at 100%, and you should see a good 20 to 40 percent increase in your hash.
4. Watch htop and you'll see the swap file filling up over time... meaning, your system should be more stable!

Note: Watch for a bit, and if you get a `Segmentation fault (core dumped)` error, resubmit your mining command and it should continue fine. 

## Closing Remarks
I am providing this as a goodwill gesture to the community in an effort to bring positive network performance to Safex by increasing the hash of connected miners. If you enjoyed this tutorial and need a VPS provider to expand your efforts with, I recommend and use Vultr which you can sign up with my referal code at https://www.vultr.com/?ref=8343588 My current setup consists of the $24/month 2 vCPU with 4GB of RAM and 128GB SSD instances either solo mining with safexd 5.0.0 or pool mining to https://pool.safexnews.net/ with xmrig. Following the above I am able to get between 700H/s and 1.3kH/s. I cannot guarantee the same results to you due to geographical data center and other variances. All that said, VPS mining is tricky, and you really have to watch your profit vs cost basis carefully in addition to supply and demand to avoid losing real fiat. I hope this worked for you and improved your speed! Cheers! -- Cyberian

## Reference Links
https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/
<br/>https://medium.com/@tomas_savenas/30-increase-in-cpu-mining-hash-rate-by-enabling-huge-pages-8af5eedb7d62
<br/>https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html
<br/>https://www.vultr.com/?ref=8343588
<br/>https://pool.safexnews.net/
