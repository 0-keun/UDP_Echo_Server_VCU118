# UDP_Echo_Server_VCU118
Simple communication example of verilog-ethernet provided from "alexforencich"

# Assumptions
- License
- Windows
- wsl

# Tested Environments
- VCU118
- Vivado 2023.2.2

# VCU118 `fpga_25g` Build Notes

## Build Environment
This example was built in **WSL (Ubuntu)** instead of Windows Command Prompt.  
The `fpga_25g` design depends on the full `verilog-ethernet` repository structure and a working Vivado command environment.
Not 

## Build Procedure

### 1. Move to the example directory
```bash
cd ~/okeun_ws/verilog-ethernet-master/example/VCU118/fpga_25g
```

### 2. Check the repository structure
```bash
ls
ls ..
find /home/uripel/okeun_ws/verilog-ethernet-master -name eth_mac_1g_fifo.v
```

The required file was found in the repository root:
```text
/home/uripel/okeun_ws/verilog-ethernet-master/rtl/eth_mac_1g_fifo.v
```

### 3. Move the project to a Windows-mounted path
Vivado was executed through Windows, so the project had to be built under `/mnt/c/...` instead of the WSL home directory.

```bash
cd "/mnt/c/Users/Teacher/Desktop/okeun_ws/VCU118 Developement Note/Code/verilog-ethernet-master/example/VCU118/fpga_25g"
```

### 4. Fix the `lib/eth` path
The example expected `fpga_25g/lib/eth` to point to the repository root.

```bash
rm -rf lib/eth
ln -s ../../../.. lib/eth
```

### 5. Verify the symbolic link
```bash
ls -l lib
ls lib/eth/rtl/eth_mac_1g_fifo.v
```

Expected result:
```text
lib/eth/rtl/eth_mac_1g_fifo.v
```

### 6. Create a Vivado wrapper in WSL
Vivado was installed on Windows, so a wrapper script was created in WSL.

```bash
sudo rm -f /usr/local/bin/vivado

sudo tee /usr/local/bin/vivado > /dev/null <<'EOF'
#!/bin/bash
cmd.exe /c C:\\Xilinx\\Vivado\\2023.2\\bin\\vivado.bat "$@"
EOF

sudo chmod +x /usr/local/bin/vivado
```

### 7. Remove the conflicting `vivado` alias
```bash
unalias vivado
hash -r
```

### 8. Check the Vivado command
```bash
type -a vivado
which vivado
ls -l /usr/local/bin/vivado
cat /usr/local/bin/vivado
```

### 9. Run the build
```bash
make
```

# How to run VCU118 `fpga_25g`

### 1. Connect cables and turn on VCU118
- Power cable to J15
- Ethernet cable to J10
- JTAG cable to J106
- QSFP loopback module to J96 (it must be placed at QSFP1, not QSFP2). Hole which seems L should be top
- SW12: 1000 (1001 can show you the status of QSFP1,2. If GPIO LEDs 0,1,2,3 are turn on, QSFP1 is working well.) 
- SW16: 1010

### 2. Upload bitstream file to VCU118
Using Vivado, you can upload bitstream file.
"PROGRAM AND DEBUG > Open Hardware Manager > Open Target > Auto connect"
After the device is connected you can upload bitstream.
"Program Device > "
Now VCU1118 is ready for etherent communication

### 3. Set IP same as VCU118
To communicate with VCU118, we have to set the IP same as VCU118.

Check IP using cmd,
```bash
ipconfig
```
If you are using 192.168.1.x, you are all set.

Otherwise,
Press Win button, and go to "Ethernet Setting".
Find "IP assignment" and press "Edit".
Change Automatic to Manual and turn on IPv4.
IP address 192.168.1.10
Subnet mask 255.255.255.0
Save

### 4. Send data through Ethernet
Using WSL,
```bash
echo {data} | nc -u 192.168.1.128 1234
```

### 5. Results
You can see same data on the terminal.
SW12.1 == 0, You can check a first byte of data via GPIO LEDs.
When you send "A", the ASCII Code b'0100_0001 will be displayed.

## Notes
- The main issues were related to environment setup and path resolution, not the HDL design itself.
- The project must be built from a Windows-mounted path such as `/mnt/c/...` when Vivado is launched through Windows.
- The `lib/eth` symbolic link must point to the repository root so that the example can find the required Ethernet source files.
