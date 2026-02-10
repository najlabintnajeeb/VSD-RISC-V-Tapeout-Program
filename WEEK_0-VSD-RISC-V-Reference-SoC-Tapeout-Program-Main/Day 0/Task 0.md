
<h1 align="center">Task 0: Tool Installation</h1>

This report documents the successful completion of Task 0, which
involved the installation of essential hardware design and verification
tools.

**1. Virtual Machine Configuration**

The tools were installed on a virtual machine configured using UTM on a
macOS system.

| Operating System: Ubuntu 24.04 |
|--------------------------------|
|                                |
| RAM: 6 GB                      |
|                                |
| HDD: 50 GB                     |
|                                |
| vCPU: 4                        |


**2. Tool Installation and Verification**

The following tools were installed and verified. The commands used for
each step are listed below.


<h2><u>Yosys</u></h2>

Yosys, an open-source Verilog RTL synthesis framework, was installed by
cloning its repository and compiling from source.

**Installation Commands:**

<img width="652" height="490" alt="yosys installation commands" src="https://github.com/user-attachments/assets/8bb4e091-505a-4268-a34c-c02dab7b6259" />

<details>
  <summary>Terminal snapshots...</summary>
  <img width="600" height="300" alt="Screenshot 2025-09-20 at 2 40 31 pm" src="https://github.com/user-attachments/assets/be638e2a-26a6-497d-a008-efe55d47d470" />

  <img width="600" height="337" alt="make install" src="https://github.com/user-attachments/assets/b3eb3301-c7f7-41de-810d-9cfdc5edd4da" />

<img width="600" height="300" alt="dependencies" src="https://github.com/user-attachments/assets/bea916bf-1678-4d44-b0d1-7b9855cdb4ad" />



</details>




**Verification Command: yosys**

<img width="600" height="300" alt="yosys verification" src="https://github.com/user-attachments/assets/40182a04-415d-4d9e-8163-a8f522b18c42" />





<h2><u>Icarus Verilog (Iverilog)</u></h2>

Iverilog, a Verilog compiler, was installed directly from the Ubuntu
package repository.

**Installation Commands:**

<img width="900" height="224" alt="image" src="https://github.com/user-attachments/assets/3aa4afa9-1d46-4e20-bda9-6227260e2cd8" />


<details>
  <summary> Terminal snapshots...</summary>
  
 <img width="600" height="260" alt="update" src="https://github.com/user-attachments/assets/589e0af1-3b64-4ac4-ab04-948c750f3c16" />
<img width="600" height="275" alt="iverilog installation" src="https://github.com/user-attachments/assets/7d50a38d-4e6d-4bcf-a0b2-303923cdc318" />


</details>

**Verification Command: iverilog -v**

<img width="600" height="300" alt="iverilog verification" src="https://github.com/user-attachments/assets/215ac963-73c3-4522-be44-40f8afc22420" />




<h2><u>GTKwave</u></h2>

GTKWave, a waveform viewer, was also installed directly from the Ubuntu
package repository.

**Installation Commands:**

<img width="650" height="112" alt="Screenshot 2025-09-19 at 4 14 33 pm" src="https://github.com/user-attachments/assets/5404ce2c-a29e-48f5-9312-bf7869559476" />

<details>
  <summary> Terminal snapshots...</summary>
 <img width="600" height="286" alt="gtkwave installation" src="https://github.com/user-attachments/assets/e0f3437d-cd7a-4747-b791-6553439a9460" />
</details>



***Verification Command: gtkwave***

<img width="600" height="263" alt="gtkwave verification" src="https://github.com/user-attachments/assets/6b8a0b67-02aa-497d-99c4-a9fcfb050aad" />


<img width="600" height="300" alt="gtkwave" src="https://github.com/user-attachments/assets/34e1aa7e-8590-40d8-bcce-ff509ee940c8" />


