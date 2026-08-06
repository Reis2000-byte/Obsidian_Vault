- Half duplex where only one person can talk on the bus at a time
- Differential because there is normally a ton of noise on the line due to this being used in a car. If noise is induced then both lines will go up by the same amount
- Two wire. CANL and CANH lines
- Asynchronous but the devices must agree on a baud rate similar to UART
- CRC to help verify that the received data is correct

Data Packet:
- starts with a start bit, then arbitration field (ID), then control bit, then actual data, then CRC bits, then end frame bits