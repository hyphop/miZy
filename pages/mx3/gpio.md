
## GPIOS 

16 gpios + 2 x GND + 2 x 5V + 3.3V

```
POWER 5V

 0 x GND         1 x 139 = PE11
 2 x 138 PE10    3 x 137 = PE09
 4 x 136 PE08    5 x 135 = PE07
 6 x 134 PE06    7 x 133 = PE05
 8 x 132 PE04    9 x 140 = PE12
10 x 130 PE02   11 x 131 = PE03
12 x 128 PE00   13 x 129 = PE01
14 o 018 PA18   15 o 019 = PA19
16 x 143 PE15   17 o ? GND
18 x ??         19 x 5V
20 x 5V         22 o 3.3V


    PA18 PWI1 SCK
    PA19 PW11 SDA

```

## UART


```
    RX  GND
  x x x x
      TX
```