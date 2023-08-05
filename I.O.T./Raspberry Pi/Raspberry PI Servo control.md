## Project Overview
This Project Aims to create a raspberry pi program that can control a servo motor for a specified period of time. The scope of the project is to create a working test model that can be controlled via remote access to turn the servo motor.

## How to....
### Basic Logic formation
I have to write a program that will move the servo arm from on location to another then hold for 5 sec the return to original position. So I created a function that will do the same using basic library.
Currently it is doing just a basic print statement but later it will be controlling servo motor.
```python
import time 

def servo_control():
    print("servo On")
    time.sleep(5)
    print("servo Off")

servo_control()
```

Next, I have to make sure this function is called during specific time of the day. The below code makes it so that the print statement will occur only if current time hour is 21.

```python
import datetime

now = datetime.datetime.now()
setime = 21
print("The Current time is ",now.hour)
if now.hour == setime:
    print("Time is up its",setime)

```

Combining both these gives, us this thing which control the servo when certain time is up.
```python
import time 
import datetime

def servo_control():
    print("servo On")
    time.sleep(5)
    print("servo Off")

# servo_control()
now = datetime.datetime.now()
setime = 21
print("The Current time is ",now.hour)
if now.hour == setime:
    print("Time is up its",setime)
    servo_control()
    # print(now.hour)

```
### Schematic of Raspberry pi pin diagram connection 

![[I.O.T./Raspberry Pi/Images/Screenshot from 2023-08-05 20-57-01.png]]