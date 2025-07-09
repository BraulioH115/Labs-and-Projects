Lab 03: Process Management and Scheduling
Course: IS-3003 – Operating Systems Security
Platform: Red Hat Linux
Student: Braulio Hurtado

Part 1: Process Management
Background Process Execution
We executed a program (./lab03) in the background using the & operator:

bash
Copy
Edit
./lab03 &
To verify the process was running, we used:

perl
Copy
Edit
ps aux | grep lab03
This confirmed that the process was running independently of the shell session.

Parent-Child Relationship
We tested what happens when a parent process is terminated while the child continues running. The result: the child process becomes orphaned and is automatically adopted by the init process. In the screenshot, the child shows a PPID (Parent PID) of 1, indicating this behavior.

Zombie Process Behavior
Next, we examined the outcome of killing the child process while allowing the parent to continue. In this case:

The child process changed to a zombie state (process remains in the process table but is no longer executing).

The parent process continued running normally.

We used the following command to observe this:

nginx
Copy
Edit
ps -u $USER
Observations included:

CPU usage: 3.1% while the process was active.

Memory usage: 0%, indicating efficient memory use.

High VSZ (Virtual Memory Size) relative to other processes.

These results helped demonstrate real-world scenarios of process hierarchy and cleanup in Linux.
