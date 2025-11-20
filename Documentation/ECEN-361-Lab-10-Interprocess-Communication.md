# ECEN-361 Lab-09: IPC-Examples
     Student Name:  ___________________________________

## Introduction and Objectives of the Lab

This lab shows how various independent tasks can communicate through the
RTOS-controlled signaling mechanisms. It also shows the good practice of
handling interrupts by using semaphores to signal their respective
tasks.

This lab will demonstrate the following Interprocess Communication
mechanisms:

1.  Semaphores and Task-Notifications

2.  Mutexes

3.  Protected Global Variables

4.  Process Notification via Software Timers

In the provided FreeRTOS system, tasks are set up to use these
mechanisms to communicate with each other. The mechanisms and associated
tasks (and assignments) are described in the respective sections of the
lab.
___
<!--------------------------------------------------------------------------------->
## Part 1: RTOS-friendly debounced buttons -- Notifications and Semaphores
<!--------------------------------------------------------------------------------->

As discussed in class, all mechanical buttons come with the mechanically
generated issue of 'bouncing' which can create multiple input requests
to a system where only a single entry is required. Methods to alleviate
this problem include external RC circuits on the button input,
dedicating a timer to the incoming button, and software counting loops
to debounce the input. Note that the 'bounce' characteristics of the
switch may vary, but in general they are much less than a human response
time. Typically, a debounce time on the order of 10 -30 milliseconds is
long enough to get rid of the mechanical flaw and not miss a human
reaction time.

In an RTOS environment, the timeout necessary to debug a switch should
not come within the ISR servicing the button event. Leaving the RTOS to
'wait' for a debounce inside of the ISR is bad practice as it can be
blocking. One simple solution is to use notification mechanisms in the
RTOS and trigger them from within the ISR. Two such mechanisms are
'**Semaphores'** and '**Task-Notifications'**. **Semaphores** can be
binary or counting and can track numbers of events from multiple
sources. Task-Notifications are simply a signal from one process to
another.

By using a signaling mechanism this way, we create a 'Thread-safe'
interrupt service routine. This means that the ISR will not create
unexpected behavior in a multi-process environment and supports scalable
grow with more/less processes. It's 'safe' to threads.

For a task(s) waiting for a clean, debounced button event, the initial
button press can't trigger the task(s), it must first trigger the
debounce wait, that then starts the task(s):

<img src="media/safe_debounce1.png" alt="safe-debounce" width="85%" style="display: block; margin: 0 auto;" :w
/>

<!-------------------------------------------------------------------------------->
### Part 1:  Instructions and Questions (2 pts)


>**Create a Task**<br>
(Either use the GUI -- *Under Middleware/FreeRTOS* or manually in the code) that will toggle LED_D4 each time Button_1 is pressed. <br>
<br>
<mark>1. How did your task ‘wait’ for the debounced button? <br>
So the task 'waits' because once we press the button in the callback it has the debounce task which waits 30 ms and then after that time it then releases the semaphore</mark>
<br>
> <br><br>

><mark>2.)	How long is the time between the button interrupt coming in and it being enabled again? <br>
30 ms</mark>
><br>
> <br>

>**Modify Task to be Safe**<br>
You’re given the framework of a second task (Semaphore_Toggle_D3) -- <p>
>Modify this so that it that also waits for the same Button_1_Semaphore.  It should then toggle LED_D3 each time Button_1 is pressed.  Note that they both have been created with the same Priority..<br><br>
><mark>3.)<br>
>><mark>[x] Yes, I modified and got it working and tested it by<br>
>>creating the task and then once I created the task, I added the code that would allow the LED_D3 to toggle.<br><br></mark>
><br>
><br>
> </p>

><mark> <br>
4.)	Do both of (D4 and D3) toggle with a single button press?  Describe the behavior?  <br>
>No if D3 is toggled then when I press the button again D4 will toggle and vice versa with D4.<br><br>

><mark> 5.)	Now change one of the priorities of these two tasks, re-compile,  and re-run. <br>
How has the behavior changed?
>I set D4 to low priority and because they are not on the same priority they do not take turn but rather D3 is the only LED that toggles.<br><br>

<mark>



<!--------------------------------------------------------------------------------->
___
<!--------------------------------------------------------------------------------->
## Part 2: Mutexes
<!--------------------------------------------------------------------------------->

Mutex's are used in embedded systems to guarantee exclusive access to a
resource that more than one process may need to use. Examples include
hardware devices or shared memory locations that could be requested
simultaneously.

For the hardware set of our labs, we have only a single 4-digit
seven-segment display. If there were multiple producers trying to change
the value of this display, one way to guarantee its integrity would be
to protect it with a mutex: only one process at a time gets to send
output to it.

For this example, the following picture shows how a mutex could protect
a value being display on the seven-segment:

<img src="media/mutex.png" alt="safe-debounce" width="85%" style="display: block; margin: 0 auto;" />

The three processes on the right are all competing for access to write
to the protected variable. The **mutex_protected_count** is continuously
updated to the seven-segment display on the right. The CountDown and
CountUp are both trying at random intervals, and the Reset_MutexCount
waits for the Button_3 (via its associated semaphore), then resets the
current count.

<img src="media/def_sematoggle4.png" alt="safe-debounce" width="50%" style="display: block; margin: 0 auto;" />

### Part 2:  Instructions and Questions (4 pt)

><mark>6.) Add another process that uses the mutex to take control and put a “—” on the Two_Digit UpDown SevenSegment Display.
<br>Use the following code in your process:<br>
><br>
```c
    for(;;) 
        {
        /* This doesn't change the value, it just clears the display  */
        /* If asked to display a negative number, the function displays a "--"
        */
        osMutexWait(UpDownMutexHandle,100000);
        MultiFunctionShield_Display_Two_Digits(-1);
        osDelay(200);
        osMutexRelease(UpDownMutexHandle);
        osDelay(2);
        }
```
>
><br>
>7.)	Comment on the Up/Down/ ”—” display that you see.  <br><br>
><mark>They are all the same priority so they are all going one after the other fighting to be the only one going so the numbers are tweaking along with the dashes.<br><br><p>


>8.)	Is there a ‘priority’ associated with the Mutex?  If so, how can it be changed?
><br>  
><mark>Yes there is priority as we see the mutex uses tasks which in the GUI we can change which one has the supreme priority overall.<br><br>
<p>

><br>
>9.)	Button_3 resets the mutex-protected global variable to “50.”  It too, has to wait for the mutex to be granted.<br>
> <mark> Change the priority of the Reset to be osPriorityIdle.  This is the lowest priority available.<br>
><br> Did you see any effect on the ability of Button_3 to reset the count?<br><br>
><mark>There is no disability to the reset however it does take longer to restart the tasks.<br><br>
>
---
<!--------------------------------------------------------------------------------->
##  Part 3: Software Timers
<!--------------------------------------------------------------------------------->

In a previous lab, we learned how to setup any of the hardware timer
blocks to generate periodic interrupts. This is limited by the number of
actual timers available on the silicon, and still requires careful
coding to insure RTOS-friendly servicing, as discussed with the buttons
above. Software timers are easy to deploy, flexible to use, and have no
concerns operating in the RTOS environment.

Software timers don't generate an interrupt, they simply execute a
procedure when they expire.

Button_2 is used to toggle the SW_Timer_7Seg between running/stopped.
When the timer expires, it calls a routine to decrement the left-most
display digit.


### Part 3:  Instructions and Questions (2 pts)

>
>9.)	 Change the timer period from the current “200” mS to something different.
>Verify that the decrementing count changes accordingly.
>
>><mark>[x] Yes, I modified and got it working and tested it by<br>
>>looking at the left of the 7 seg display I can see that it is now counting much slower because from 200 I switched it to 1000<br><br></mark>
>
>This timer was created via the GUI  (.IOC file).  It’s type is *“osTimerPeriodic”* which means it repeats over and over.<br><br>
What other options can a Software Timer take to change its Type and operation? <br>
><mark>10.)  So you can either have it be periodic which it is currently at or you can have it count once everytime you press the button.<br><br>

<mark>>11).    The debounce for the switches here used an osDelay() call (non-blocking).  Is there any advantage to using a SWTimer here instead?<br>
Explain why or why not?
>
><mark>Yes there would be a benefit here as it would be just as fast if not faster and it would be non-blocking. Also we would not need to make a task<br><br>


<!--------------------------------------------------------------------------------->
## EXTRA CREDIT Ideas (5 pt. for any)
<!--------------------------------------------------------------------------------->

>1.	The Seven-Seg Display is currently refreshed with a hardware (TIM17) timer.  Make this more thread-safe by changing the refresh as a process that is based off a S/W timer.
>
>Write about how you did it, and what the slowest period could be to keep the persistence looking good:
><mark>___________________________________________________________________________________________________________<br><br>

>2.	We only used a binary semaphore in this lab for the switch presses.  Change it so that presses are accumulated through a counting semaphore and then handled as they are taken off.<br><br>
>Describe any issues with this approach
>
><mark>___________________________________________________________________________________________________________<br><br>
>

>
>3.	The debounce for the switches here used a uwDelay() non-blocking call.  Is there any advantage to using a SWTimer here instead?<br>
> Explain why or why not?
>
><mark>In this lab there would not be any real benefit because it would just do the same thing and they are both non-blocking calls. So it doesn't seem like there is much benefit here.<br><br>

>
>4.	Any other relevant uses for semaphores, mutexes, or S/W timers ?   Describe what you’ve done and why?
>
><mark>___________________________________________________________________________________________________________<br><br>
>
><br>



