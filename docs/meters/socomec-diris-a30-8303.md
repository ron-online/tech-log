Title: Socomec DIRIS A-30 Returning Modbus Exception 83 03 (01 83 03 01 31) on All Registers — Root Cause & Fix

Problem Summary

When polling a Socomec DIRIS A-30 multifunction power meter via Modbus RTU (RS-485), the meter replies with a valid CRC—proving that RS-485 wiring (A/B polarity), baud rate, parity, and Slave ID are all correct. However, every Function Code 03 (Read Holding Registers) request fails with Exception 03 (Illegal Data Value) across all standard register blocks.

Exact Test Frames & Responses (Slave ID 01)

• Phase Currents (I1–I3, Extended Table 50528 / 0xC560, 6 words):
	• Sent: 01 03 C5 60 00 06 F9 1A
	• Received: 01 83 03 01 31
• Phase Currents (I1–I3, Standard Table 768 / 0x0300, 6 words):
	• Sent: 01 03 03 00 00 06 C5 8C
	• Received: 01 83 03 01 31
• Device Identification (256 / 0x0100, 1 word):
	• Sent: 01 03 01 00 00 01 85 F6
	• Received: 01 83 03 01 31
• Product Name String (50050 / 0xC382, 8 words):
	• Sent: 01 03 C3 82 00 08 D8 60
	• Received: 01 83 03 01 31
• Function Code 04 Check (Input Registers):
	• Sent: 01 04 01 00 00 01 30 36
	• Received: 01 84 01 82 C0 (Exception 01: Illegal Function — confirms the module uses Function 03 only)

Root Cause

Hardware failure of the plug-in RS-485 JBUS/Modbus communication module (Socomec Part No. 48250092).
The RS-485 transceiver inside the 48250092 module stays alive enough to receive frames, check CRCs, and reply on the bus, but internal communication between the plug-in module and the DIRIS A-30 main meter has failed—causing every register read to return Illegal Data Value (0x03).

Solution & Working Decoding Example

Replace the faulty Socomec 48250092 RS-485 plug-in module. Once replaced, sending 01 03 C5 60 00 06 F9 1A returns 12 data bytes (3 × U32, Big-Endian, in milliamps):
• Example Response: 01 03 0C 00 01 E2 40 00 01 D4 C0 00 01 F0 10 [CRC]
	• Byte Count (0C): 12 bytes (6 registers)
	• Phase 1 Current (I1): 0x0001E240 = 123,456 mA ÷ 1000 = 123.456 A
	• Phase 2 Current (I2): 0x0001D4C0 = 120,000 mA ÷ 1000 = 120.000 A
	• Phase 3 Current (I3): 0x0001F010 = 126,992 mA ÷ 1000 = 126.992 A
