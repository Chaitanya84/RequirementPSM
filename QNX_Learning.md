### How does resource manager work


```mermaid
flowchart TB
    Client["SW<br/>Application"]
    PM["Process Manager"]
    RM["Resource Manager"]

    Client -->|"1. Query<br/>(Who is responsible?)"| PM
    PM -->|"2. Resource manager<br/>file descriptor"| Client

    Client -->|"3. Open connection"| RM
    RM -->|"4. Connection status<br/>(PASS / FAIL)"| Client

    style Client fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style PM fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#FFFFFF
    style RM fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#FFFFFF
```
----------------
### Shared library
- when multiple process wants to load some library at run time then those shared library are stored only once in the memory. 
- When a particular process calls it then the kernel loads it as <u>_**PROCESS PRIVATE**_ </u>Library in <u> _**read only**_</u> mode


```mermaid
flowchart TB
    PS1["Process 1"]
    PS2["Process 2"]
    PS3["Process 3"]
    SO["Shared Object"]

    PS1 -->|"Load SO at Runtime"| SO
    PS2 -->|"Load SO at Runtime"| SO

    PS3 -->|"Load SO at Runtime"| SO

    style PS1 fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style PS2 fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style PS3 fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style SO fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#FFFFFF
```
-----------------------------------------------------------------
### OS Services

_**Services**_ can be _**dynamically created or removed**_.

Some example of Service/Process are 

```
1. random - Supply random numbers
2. mqueue - POSIX message queue IPC
3. dumper - Core dump creation
4. pipe   - Unix pipe
5. devb-* - Filesytem, usually rotating media e.g. SDCard
6. devf-* - Filesystem NOR flash
7. io-sock - TCP/IP stack
8. slogger2 - QNX System logger
9. pci-server - PCI bus access and configuration
```

Command used to <u>terminate a process</u> _**kill**_ 


----------------

### Boot Sequence 

```mermaid
flowchart LR
    subgraph External source
        PWR["Power"]
        DONE["DONE"]
    end
    subgraph Hardware
        CPU["CPU"]
        BO["BIO\n\n\nROM Monitor"]
    end
    subgraph QNX Part
    IPL["Intial\nProgram\nLoader"]
    STR["StartUp \nCode"]
    PRCNT["Procnto\nKernel\n+\nProcess manager"]
    BTSCR["Boot Script \nCode"]
    end
    PWR-->CPU-->BO-->IPL-->STR-->PRCNT-->BTSCR-->DONE
    
    style PWR fill:#FF0000,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style CPU fill:#189926,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style BO fill:#189926,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style IPL fill:#F54927,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style STR fill:#F54927,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style PRCNT fill:#F54927,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style BTSCR fill:#F54927,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF
    style DONE fill:#F54927,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF    
```

The _**startup**_ code is :
- Board Soecific
- tells _**procnto**_ about core hardware
    - System RAM amount and layout
    - Interrupt Controllers and its address
    - _ClockCycles()_ per second i.e. it is reponsible for intialising clock
    - Special Memory reagin e.g. DMA address, Read Only, Write Only etc
    - **Cluster information**
- Communicate this data through system page (_**syspage**_)
    - **`syspage information is mapped to every PROCESS in read-only mode`**

 **`Drivers are intialised in BOOT SCRIPT section`**

-----------------------------------

### Security
- _**qconn**_ runs everything in root mode **this should never be shipped in production**
- _**setuid()**_ can be used to assign permission to the application like `root` or `qnxuser`
- **`secpolgenerate`** create security policy and stores the security policy **this is loaded on boot**
- QNX uses POSIX style attributes with user/group/other permissions

---------------------

**notes:**
```
- Process own resource and thread run code
- Drivers are processes in QNX
```

--------------
