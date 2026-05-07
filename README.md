<div align="center">

<br/>

<h1>LPC2148 · ARM7 · Vending Machine</h1>

<p><strong>A fully functional embedded vending machine simulation.</strong><br/>
Four beverages. Sequential LED dispensing. 16×2 LCD in 4-bit mode. Pure bare-metal C.</p>

<br/>

[![Platform](https://img.shields.io/badge/MCU-LPC2148-0078d7?style=for-the-badge)](https://www.nxp.com)
[![Core](https://img.shields.io/badge/Core-ARM7TDMI--S-ff6b35?style=for-the-badge)](https://developer.arm.com)
[![Language](https://img.shields.io/badge/Language-C-a8b9cc?style=for-the-badge&logo=c&logoColor=white)]()
[![LCD](https://img.shields.io/badge/Display-16×2%20LCD%204--bit-00d084?style=for-the-badge)]()
[![Simulation](https://img.shields.io/badge/Simulation-Proteus-e74c3c?style=for-the-badge)]()
[![License](https://img.shields.io/badge/License-Open%20Source-9b59b6?style=for-the-badge)]()

<br/>

</div>

---

## 🧭 Table of Contents

- [What This Is](#-what-this-is)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Schematic](#-schematic)
- [Hardware Connections](#-hardware-connections)
- [LCD Driver](#-lcd-driver-4-bit-mode)
- [Beverage Menu](#-beverage-menu)
- [How It Works](#-how-it-works)
- [Build & Simulate](#️-build--simulate)
- [Full Source Code](#-full-source-code)
- [Repository Structure](#-repository-structure)
- [Author](#-author)

---

## 🎯 What This Is

A **bare-metal embedded vending machine** in C on the **NXP LPC2148** (ARM7TDMI-S). No OS, no HAL, no middleware — direct register manipulation, a handwritten 4-bit LCD driver, and GPIO-driven LED dispense sequences.

Press one of four buttons → the **16×2 LCD** shows the beverage name and price → a four-step **LED sequence** simulates the dispensing mechanism. Clean, minimal, and entirely register-level.

---

## ✨ Features

| Feature | Detail |
|---|---|
| 🖥️ LCD display | 16×2 in **4-bit mode** — only 4 data lines used |
| 🧃 4 beverages | Tea · Coffee · Coca Cola · Lime Juice |
| 💡 LED sequences | Each beverage triggers a unique 4-step LED dispense animation |
| ⌨️ Button inputs | 4 push-buttons on Port 1, active-LOW |
| ⚡ Clock speed | Cclk = 60 MHz · delay loop calibrated for 1 ms accuracy |
| 🔬 Proteus ready | Full simulation schematic included |

---

## 📐 System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         LPC2148 (ARM7)                           │
│                                                                  │
│   PORT 1 (Inputs)                PORT 0 (Outputs)               │
│   ┌───────────────────┐          ┌──────────────────────────┐   │
│   │ P1.20  [SW1 Tea]  │          │ P0.4    RS  ─────────►  │   │
│   │ P1.22  [SW2 Coffee│──────►   │ P0.5    RW   16×2 LCD   │   │
│   │ P1.24  [SW3 Cola] │          │ P0.6    EN  ─────────►  │   │
│   │ P1.26  [SW4 Lime] │          │ P0.12–15 Data nibble    │   │
│   └───────────────────┘          │                          │   │
│                                  │ P0.16–19  LED Bank Tea   │   │
│                                  │ P0.20–23  LED Bank Coffee│   │
│                                  │ P0.25–28  LED Bank Cola  │   │
│                                  │ P0.29–31  LED Bank Lime  │   │
│                                  │ P1.16     LED Lime (last)│   │
│                                  └──────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 💡 Schematic

<p align="center">
  <img src="./schematic.png" alt="Proteus Circuit Schematic" width="750"/>
  <br/><em>Proteus simulation — 16×2 LCD, push-buttons, and LED dispense indicators</em>
</p>

---

## ⚡ Hardware Connections

### LCD (4-bit mode) — Port 0

| LCD Signal | LPC2148 Pin | Notes |
|---|---|---|
| D4–D7 (data nibble) | `P0.12 – P0.15` | Upper nibble first, then lower |
| RS (Register Select) | `P0.4` | LOW = command · HIGH = character |
| RW (Read/Write) | `P0.5` | Held LOW (write-only mode) |
| EN (Enable) | `P0.6` | Rising then falling edge latches data |

### Buttons — Port 1 (active LOW)

| Button | Beverage | LPC2148 Pin |
|---|---|---|
| SW1 | Tea | `P1.20` |
| SW2 | Coffee | `P1.22` |
| SW3 | Coca Cola | `P1.24` |
| SW4 | Lime Juice | `P1.26` |

### LED Dispense Bank

| Beverage | LED Pins | Steps |
|---|---|---|
| Tea | `P0.16 → P0.17 → P0.18 → P0.19` | 4 × 100 ms |
| Coffee | `P0.20 → P0.21 → P0.22 → P0.23` | 4 × 100 ms |
| Coca Cola | `P0.25 → P0.26 → P0.27 → P0.28` | 4 × 100 ms |
| Lime Juice | `P0.29 → P0.30 → P0.31 → P1.16` | 4 × 100 ms |

---

## 🖥️ LCD Driver (4-bit mode)

Data is sent as **two nibbles per byte** — saves 4 GPIO pins over 8-bit mode.

```
Sending one byte (command or data):
┌─────────────────────────────────────────────┐
│  1. Place upper nibble on P0.12–P0.15       │
│  2. Set RS accordingly (0=cmd, 1=data)      │
│  3. Pulse EN HIGH then LOW                  │
│  4. Wait 5 ms                               │
│  5. Place lower nibble on P0.12–P0.15       │
│  6. Pulse EN HIGH then LOW                  │
│  7. Wait 5 ms                               │
└─────────────────────────────────────────────┘
```

### Initialization Sequence

```c
LCD_CMD(0x02);  // Enter 4-bit mode
LCD_CMD(0x28);  // 2-line display, 5×8 dot font
LCD_CMD(0x0C);  // Display ON, cursor OFF
LCD_CMD(0x06);  // Cursor auto-increment (left to right)
LCD_CMD(0x01);  // Clear display
LCD_CMD(0x80);  // Cursor to line 1, position 0
```

---

## 🧃 Beverage Menu

| # | Beverage | Price | LED Pins | LCD Output |
|---|---|---|---|---|
| 1 | ☕ Tea | Rs 10 | P0.16–P0.19 | Line 1: `Tea` · Line 2: `Rs 10` |
| 2 | ☕ Coffee | Rs 20 | P0.20–P0.23 | Line 1: `Coffee` · Line 2: `Rs 20` |
| 3 | 🥤 Coca Cola | Rs 35 | P0.25–P0.28 | Line 1: `Coca Cola` · Line 2: `Rs 35` |
| 4 | 🍋 Lime Juice | Rs 15 | P0.29–P0.31 + P1.16 | Line 1: `Lime Juice` · Line 2: `Rs 15` |

---

## 🧠 How It Works

```
Power ON
   │
   ▼
LCD_INIT()  →  4-bit mode, 2-line, cursor off
   │
   ▼
LCD: "Choose your beverage:"
   │
   ▼
┌──────────────── Polling Loop (while 1) ────────────────────┐
│                                                            │
│  P1.20 LOW? → printBeverage("Tea","Rs 10")                │
│               LED: P0.16 → P0.17 → P0.18 → P0.19         │
│                                                            │
│  P1.22 LOW? → printBeverage("Coffee","Rs 20")             │
│               LED: P0.20 → P0.21 → P0.22 → P0.23         │
│                                                            │
│  P1.24 LOW? → printBeverage("Coca Cola","Rs 35")          │
│               LED: P0.25 → P0.26 → P0.27 → P0.28         │
│                                                            │
│  P1.26 LOW? → printBeverage("Lime Juice","Rs 15")         │
│               LED: P0.29 → P0.30 → P0.31 → P1.16         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

Each LED step: clear all LEDs in bank → set one HIGH → wait 100 ms → advance.

---

## ⚙️ Build & Simulate

### Compile with Keil µVision

```
1. Clone this repository
2. Open in Keil µVision 4 or 5
3. Target device: LPC2148
4. Set Cclk = 60 MHz in clock configuration
5. Build → vending_machine.c → .hex generated
```

### Simulate in Proteus

```
1. Open vendingmachine.pdsprj in Proteus 8+
2. Load the .hex into the LPC2148 component
3. Run the simulation
4. Click any button and observe:
   - LCD updates with beverage name + price
   - LEDs light up sequentially (dispense animation)
```

### Expected LCD Output

```
Idle:                          After SW1 (Tea):
┌────────────────┐             ┌────────────────┐
│Choose your     │    ──────►  │Tea             │
│beverage:       │             │Rs 10           │
└────────────────┘             └────────────────┘

After SW3 (Coca Cola):         After SW4 (Lime Juice):
┌────────────────┐             ┌────────────────┐
│Coca Cola       │             │Lime Juice      │
│Rs 35           │             │Rs 15           │
└────────────────┘             └────────────────┘
```

---

## 💻 Full Source Code

<details>
<summary><strong>📄 vending_machine.c — click to expand</strong></summary>

```c
/*
  Vending Machine — LPC2148 (ARM7TDMI-S)
  16×2 LCD in 4-bit mode + GPIO button inputs + LED dispense sequences
  Cclk = 60 MHz
*/

#include <lpc214x.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>

/* ─── Delay (1 ms per count @ 60 MHz) ────────────────── */
void delay_ms(uint16_t j)
{
    uint16_t x, i;
    for (i = 0; i < j; i++)
        for (x = 0; x < 6000; x++);
}

/* ─── LCD: send command byte (4-bit mode) ─────────────── */
void LCD_CMD(char command)
{
    IO0PIN = (IO0PIN & 0xFFFF00FF) | ((command & 0xF0) << 8);
    IO0SET = 0x00000040; IO0CLR = 0x00000030;
    delay_ms(5);
    IO0CLR = 0x00000040;
    delay_ms(5);

    IO0PIN = (IO0PIN & 0xFFFF00FF) | ((command & 0x0F) << 12);
    IO0SET = 0x00000040; IO0CLR = 0x00000030;
    delay_ms(5);
    IO0CLR = 0x00000040;
    delay_ms(5);
}

/* ─── LCD: initialise ─────────────────────────────────── */
void LCD_INIT(void)
{
    IO0DIR = 0x0000FFF0;
    delay_ms(20);
    LCD_CMD(0x02);  /* 4-bit mode            */
    LCD_CMD(0x28);  /* 2 lines, 5×8 font     */
    LCD_CMD(0x0C);  /* Display ON, cursor OFF */
    LCD_CMD(0x06);  /* Auto-increment         */
    LCD_CMD(0x01);  /* Clear                  */
    LCD_CMD(0x80);  /* Line 1, col 0          */
}

/* ─── LCD: send string ────────────────────────────────── */
void LCD_STRING(char *msg)
{
    uint8_t i = 0;
    while (msg[i] != 0)
    {
        IO0PIN = (IO0PIN & 0xFFFF00FF) | ((msg[i] & 0xF0) << 8);
        IO0SET = 0x00000050; IO0CLR = 0x00000020;
        delay_ms(2); IO0CLR = 0x00000040; delay_ms(5);

        IO0PIN = (IO0PIN & 0xFFFF00FF) | ((msg[i] & 0x0F) << 12);
        IO0SET = 0x00000050; IO0CLR = 0x00000020;
        delay_ms(2); IO0CLR = 0x00000040; delay_ms(5);
        i++;
    }
}

/* ─── LCD: clear and print beverage info ─────────────── */
void printBeverage(char *name, char *price)
{
    LCD_CMD(0x01);
    LCD_STRING(name);
    LCD_CMD(0xC0);
    LCD_STRING(price);
}

/* ─── Main ────────────────────────────────────────────── */
int main(void)
{
    LCD_INIT();
    LCD_STRING("Choose your");
    LCD_CMD(0xC0);
    LCD_STRING("beverage:");

    PINSEL1 = 0x00000000;
    PINSEL2 = 0x00000000;
    IO0DIR |= 0xFFFF0000;   /* P0.16–P0.31 output */
    IO1DIR |= 0x00010000;   /* P1.16 output        */

    while (1)
    {
        /* SW1 — Tea */
        if ((IO1PIN & (1 << 20)) == 0) {
            printBeverage("Tea", "Rs 10");
            IO0PIN = (IO0PIN | 0x00010000) & 0xFFF1FFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00020000) & 0xFFF2FFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00040000) & 0xFFF4FFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00080000) & 0xFFF8FFFF; delay_ms(100);
        }
        /* SW2 — Coffee */
        if ((IO1PIN & (1 << 22)) == 0) {
            printBeverage("Coffee", "Rs 20");
            IO0PIN = (IO0PIN | 0x00100000) & 0xFF1FFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00200000) & 0xFF2FFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00400000) & 0xFF4FFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x00800000) & 0xFF8FFFFF; delay_ms(100);
        }
        /* SW3 — Coca Cola */
        if ((IO1PIN & (1 << 24)) == 0) {
            printBeverage("Coca Cola", "Rs 35");
            IO0PIN = (IO0PIN | 0x02000000) & 0xE3FFFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x04000000) & 0xE5FFFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x08000000) & 0xE9FFFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x10000000) & 0x11FFFFFF; delay_ms(100);
        }
        /* SW4 — Lime Juice */
        if ((IO1PIN & (1 << 26)) == 0) {
            printBeverage("Lime Juice", "Rs 15");
            IO1PIN &= 0xFFFEFFFF;
            IO0PIN = (IO0PIN | 0x20000000) & 0x3FFFFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x40000000) & 0x5FFFFFFF; delay_ms(100);
            IO0PIN = (IO0PIN | 0x80000000) & 0x9FFFFFFF; delay_ms(100);
            IO1PIN |= 0x10000;
            IO0PIN &= 0x1FFFFFFF; delay_ms(100);
        }
    }
}
```

</details>

---

## 📁 Repository Structure

```
lpc2148-vending-machine/
│
├── vending_machine.c        ← Full application source
├── vendingmachine.pdsprj    ← Proteus 8 simulation project
├── schematic.png            ← Circuit diagram (export from Proteus)
└── README.md                ← You are here
```

---

## 📜 License

Open-source — free to use for learning, coursework, and personal projects.  
If this helped you, a ⭐ star is always appreciated!

---

## 👩‍💻 Author

<div align="center">

**Ashika K**

[![GitHub](https://img.shields.io/badge/GitHub-Ashika005-181717?style=for-the-badge&logo=github)](https://github.com/Ashika005)

<br/>

*Built with a 16×2 LCD, four LEDs, and a lot of register-level patience.*  
*Hit* ⭐ *if this saved you hours of datasheet diving!*

<br/>

---

*Happy embedded coding!* 🚀

</div>
