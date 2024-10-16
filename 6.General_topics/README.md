## What is ABI?
- Application Binary Interface (ABI) is a set of rules and conventions that define how software components (communicates effectively) interact with each other at the machine code level.
- Operating systems play a crucial role in defining and enforcing ABIs. 
The ABI for a particular operating system, such as Windows, macOS, or Linux, determines how software components interact with that system. 
This ensures that applications can run consistently across different hardware platforms.

### A Real-World Example:
Let's imagine you're building a Lego castle. The bricks are like software components, and the instructions are like the ABI. 
If you follow the instructions correctly, you can build a sturdy and stable castle. 
If you don't, the castle might collapse. Similarly, if software components don't adhere to the ABI, they might not work together as expected.

-------------------------------------------------------------------------
In an embedded system (like a smart device with sensors and motors), different parts (like a sensor, a fan, and a screen) need to talk to each other. Instead of making them talk directly, they send messages to the mediator (the leader). The **mediator** tells each part what to do.

Example
- Sensor: "The temperature is too high!"
- Mediator: "Okay, let me tell the fan to turn on and the screen to show the temperature."
![Screenshot 2024-10-16 135402](https://github.com/user-attachments/assets/02de4d9c-4b84-4812-b736-9788af607edc)

  
