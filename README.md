tx433
=======
This source code was taken from: http://www.jwhitham.org/ri/tx433.c

Software "kernel module", an add-on driver that may be loaded into the kernel after system startup. It's very similar to the user-mode application. It uses the internal Linux kernel function "do_gettimeofday" to obtain a microsecond-accurate clock. It transmits codes that are written to the device "/dev/tx433". For example, to send the code shown above, I can enter the following shell command:
<pre>echo 03133790 >/dev/tx433</pre>
However, merely moving the application into the kernel does not prevent other parts of the kernel preempting the transmission process. A further step is needed. Here's the code:
<pre>
    unsigned long flags;
    local_irq_save(flags);
    transmit_code(code, 10);
    local_irq_restore(flags);
</pre>
This is the ultra-privileged operation that is only allowed within the kernel: interrupts are disabled. This means that no other code can run at the same time as the "transmit_code" function, which has completely exclusive use of the CPU. Interrupts are re-enabled when "transmit_code" is done. In effect, "transmit_code" becomes the highest-priority task on the RPi.

This is sufficient to get the microcontroller-like behaviour that is required. Transmissions work perfectly. The timing is accurate to the microsecond.

It is possible that features of the ARM CPU architecture would act against predictable operation if sub-microsecond timing were required, but there is no need for that here. The problem, in this case, is the non-real-time design of Linux. The workaround is to demand higher privileges. 

The disadvantage is that the CPU is otherwise unavailable during transmission. During this time, network packets may be lost, and no other code can run. That would be a big deal in a more complex system handling other real-time tasks. That's the sort of situation where you would need a proper real-time operating system (RTOS) which would make timing guarantees that ordinary Linux cannot. Or, perhaps, a number of microcontrollers!
