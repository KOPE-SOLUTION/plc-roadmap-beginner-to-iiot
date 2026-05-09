# EP7 — Selector Switch Control

Learn how to use Selector Switch logic in Siemens S7-1200 PLC.

![Title](Thumbnail.png)

---

## Topics

- Selector Switch
- LEFT / RIGHT Control
- Interlock Logic
- Ladder Logic
- Industrial Safety Basics

---

## Inputs

| Tag | Address | Description |
|---|---|---|
| SW3_Machine_Enable | %I0.3 | Machine Enable |
| SW4_Select_Right_SW | %I0.4 | Select Right |
| SW5_Select_Left_SW | %I0.5 | Select Left |

---

## Outputs

| Tag | Address | Description |
|---|---|---|
| LED0 | %Q0.0 | RIGHT LED |
| LED1 | %Q0.1 | LEFT LED |

---

## Logic

![LogicGate](LogicGate.png)

![Ladder](Ladder.png)

---

## Learning Outcome

- Understand Selector Switch control
- Learn Interlock logic
- Prevent conflicting outputs
- Prepare for Motor Direction Control
- Prepare for Auto / Manual systems