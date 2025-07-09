# Lab 03: Process Management and Scheduling

**Course:** IS-3003 – Operating Systems Security  
**Platform:** Red Hat Linux  
**Student:** Braulio Hurtado

---
Lab Summary

This lab explores core concepts in Linux process management and CPU scheduling using a Red Hat environment. Key topics include managing background processes, observing parent-child process behavior, handling orphaned and zombie processes, and analyzing scheduling strategies like FIFO, SJF, and STCF. Through hands-on commands and scenario-based testing, the lab reinforces foundational skills in system monitoring, scheduling algorithms, and process lifecycle analysis


Part 1: Process Management

Background Process Execution

- We executed a program in the background using the `&` operator:

        ./lab03 &

This confirmed that the process was running independently of the shell.

![image](https://github.com/user-attachments/assets/3025f0b4-74ef-42e9-b9e1-6ffb6f0f73c6)


To verify the process was running:

    ps aux | grep lab03



*Parent-Child Relationship*:

- We killed the parent process while the child was still running. The child became an orphan process, adopted by the init process. This was verified by seeing a Parent PID (PPID) of 1 for the child process in ps output.

![image](https://github.com/user-attachments/assets/a39a295e-4f7f-4395-bd22-5c29d14ab536)


*Zombie Process Behavior*:


- We killed the child process and kept the parent running. 
The result:

-- The child process entered a zombie state.

-- The parent remained active.

Command used:

    ps -u $USER

Observations:

CPU usage: 3.1%

Memory usage: 0%

High VSZ (Virtual Memory Size
![image](https://github.com/user-attachments/assets/495e6858-ae03-46b0-9ba2-049a0e25ec3b)
---
Part 2: Process Scheduling

Problem 1 – FIFO vs SJF

- FIFO (First In, First Out)

--Order: P1, P2, P3, P4, P5

Total turnaround time: 121 ms

Average turnaround time: 24.2 ms

- SJF (Shortest Job First)

--Order: P5, P1, P4, P2, P3

Total turnaround time: 78 ms

Average turnaround time: 15.6 ms

Conclusion:
SJF outperforms FIFO by prioritizing shorter processes, reducing average turnaround time. However, SJF may lead to starvation if short jobs keep arriving.
