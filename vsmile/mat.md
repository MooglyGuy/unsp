## WIP gym mat uart analysis

# pad layout

```
      (?)   (Exit)                 (OK)
      0xa3   0xa2                  0xa1
      
       1              2              3
      RED             UP           YELLOW
      0x8b           0x92           0xcb
      
      
       4              5              6
      LEFT          TOAST          RIGHT
      0x8d           0x91           0xcd
      
       7              8              9
      BLUE           DOWN          GREEN
      0xa4           0x94           0x98
```


# idle traffic:

```
  55
          E6
          D6
          60
```

# left pad:
```
  C0      ! LR stick release
  8D      ! down force 5
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
```

# right pad:
```
  CD      ! left force 5
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
```

# down pad:

```
  94      ! YELLOW key press
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
  90      ! COLOR key release
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
```
          
# step on pad 9

```
  98      ! RED key press
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
  ...
    repeated every 74.04ms (avg on 115 measures)
```
          
# step on pad 7

```
  A4      ! ABC key press
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
  ...
    repeated every 70.53ms (avg on 55 measures)
```

# dual pad-press 4 and 6

```
  CD      ! left 5
  80      ! UD stick release    
  CD      ! left 5
  8D      ! down 5
  CD      ! left 5
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
  C0      ! LR stick release
  80      ! UD stick release
```
  
