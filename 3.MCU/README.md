# MCU

## Architecture
- what a car can do like go straight, go backward, go left, go right
- It basically specifies what a system should do 

## Micro-architecture
- How to implement all of these features in a real car
- Depending on the design and manufacturing company, car design might vary
- The basic functionality of the car should be the same 


![Screenshot from 2024-08-17 22-18-25](https://github.com/user-attachments/assets/9364f1a3-e0db-4d6a-800d-399c6450aa6a)

## To master CPU
- Programmers' model: details out the mode of operation and internal register that CPU has
  - How can a programmer think about CPU to do the computation
- Memory model: It specifies how the CPU interacts with the memory
  - like cpu stores some result in registers and sent it out in memory 
- Exception Model : 
- Debug Model : Like external HW is connected to debugging circuit of cpu
  - like **gdb** debugger has some power and limitation in terms of debugging the cpu

## Exception vs Interrupts
- **Exception** happens internal to the CPU (like accessing wrong memory address)
- **Interrupt** happens external to the CPU

![Screenshot from 2024-08-19 22-50-39](https://github.com/user-attachments/assets/832913e2-da6c-4a7a-a8b3-e654d25a13e6)
![Screenshot from 2024-08-19 22-50-16](https://github.com/user-attachments/assets/8bfb1bba-f751-4bb6-9344-3b84b9f93bc7)
![Screenshot from 2024-08-19 22-49-06](https://github.com/user-attachments/assets/e8c5d1bc-f404-4055-b4aa-ff70ce31f1ae)


## Different modes
- **Un-Privilege thread** mode generally made for application program execution.
- 
![Screenshot from 2024-08-19 23-13-29](https://github.com/user-attachments/assets/7f209e28-7008-4abe-b29e-45325f401d81)
![Screenshot from 2024-08-19 23-25-29](https://github.com/user-attachments/assets/de869072-99af-47b8-8a4f-61c8114e5d47)


