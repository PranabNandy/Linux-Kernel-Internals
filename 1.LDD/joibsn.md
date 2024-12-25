# LDD

Device driver communicates with device for reading and writing 
- through a bus to Device pheripheral Register (HCI)
- Bus is a data path between device controller(UFSHC) and processor
- [Processor] <----------bus-----------> [UFS HC] <-------MIPI--------> [UFS Device]
- pheripheral controller = UFS HC  ~~~~ & & ~~~~~  Pheripheral Device = UFS Device


Device driver is called as **Module** as per kernel language.

Module uses kernel services
- INT
- memory allocation
- wait queue

**insmod** is a user space application

