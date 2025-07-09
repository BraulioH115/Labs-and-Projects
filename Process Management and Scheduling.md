# Lab 03: Process Management and Scheduling

**Course:** IS-3003 – Operating Systems Security  
**Platform:** Red Hat Linux  
**Student:** Braulio Hurtado

---


Part 1: Process Management

Background Process Execution

--We executed a program in the background using the `&` operator:

    ./lab03 &

This confirmed that the process was running independently of the shell.

![image](https://github.com/user-attachments/assets/3025f0b4-74ef-42e9-b9e1-6ffb6f0f73c6)


To verify the process was running:

    ps aux | grep lab03



*Parent-Child Relationship*:

--We killed the parent process while the child was still running. The child became an orphan process, adopted by the init process. This was verified by seeing a Parent PID (PPID) of 1 for the child process in ps output.

![image](https://github.com/user-attachments/assets/a39a295e-4f7f-4395-bd22-5c29d14ab536)


*Zombie Process Behavior*:


--We killed the child process and kept the parent running. 
The result:

-The child process entered a zombie state.

-The parent remained active.

Command used:

    ps -u $USER

Observations:

CPU usage: 3.1%

Memory usage: 0%

High VSZ (Virtual Memory Size
![image](https://github.com/user-attachments/assets/495e6858-ae03-46b0-9ba2-049a0e25ec3b)

