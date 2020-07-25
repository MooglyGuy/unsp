## WIP gym mat uart analysis

# pad layout

```
      (?)   (Exit)                 (OK)
      
       1              2              3
      RED             UP           YELLOW
      
      
       4              5              6
      LEFT          TOAST          RIGHT
      
      
       7              8              9
      BLUE           DOWN          GREEN
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
    repeated every 74.04ms
```

