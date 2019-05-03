
## i2s  

```
POWER   DEV                       DAC
 
 x GND	         == BLACK   <=   
 x PG11 => bclk  == BROWN   <=  WHITE
 x PG10 => mclk  == ORANGE  <=  YELLOW LRCK
 x PG13 =>  din  == WHITE   <=
 x PG12 => dout  == GREEN   <=  BROWN DATA
                            <=  PURP  MASTER

  CPU

```

## FEX / dtb

```
i2s_mclk = port:PG10<2><1><default><default>
i2s_bclk = port:PG11<2><1><default><default>
i2s_dout0 = port:PG12<2><1><default><default>
i2s_din = port:PG13<2><1><default><default>
```