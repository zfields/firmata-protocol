Stepper Motor 2.0
===

Provides support for full 2 wire, full 3 wire, full 4 wire, half 3 wire, and half 4 wire stepper motor drivers (H-bridge, darlington array, etc) as well as step + direction drivers such as the [EasyDriver](http://www.schmalzhaus.com/EasyDriver/).
Current implementation supports 10 stepper motors at the same time (#[0-9]).

Includes optional support for acceleration and deceleration of the motor.

Also includes multiStepper support which allows groups of steppers to be simultaneously passed different ```to``` values and the duration of their movements will match. Up to five multiStepper groups can be created. The total number of steppers is still limited to 10.

Example files:
 * Version 2.0 of the stepper protocol has not yet been implemented.

Protocol
---

**Stepper configuration**
```
0  START_SYSEX                                (0xF0)
1  Stepper Command                            (0x62)
2  config command                             (0x00 = config)
3  device number                              (0-9) (Supports up to 10 motors)

4  interface                                  (upper 3 bits = wire count:
                                                000XXXX = driver
                                                010XXXX = two wire
                                                011XXXX = three wire
                                                100XXXX = four wire)

                                              (4th - 6th bits = step type
                                                XXX000X = whole step
                                                XXX001X = half step
                                                XXX010X = micro step)

                                              (lower 1 bit = has enable pin:
                                                XXXXXX0 = no enable pin
                                                XXXXXX1 = has enable pin)

5  minPulseWidth                              (0-127 measured in us)
6  num steps-per-revolution LSB
7  num steps-per-revolution MSB
8  maxSpeed LSB
9 maxSpeed MSB
10 motorPin1 or directionPin number           (0-127)
11 motorPin2 or stepPin number                (0-127)
12 [when interface >= 0x0110000] motorPin3    (0-127)
13 [when interface >= 0x1000000] motorPin4    (0-127)
14 [when interface && 0x0000001] enablePin    (0-127)
15 [optional] pins to invert                  (lower 5 bits = pins:
                                                XXXXXX1 = invert motorPin1
                                                XXXXX1X = invert motorPin2
                                                XXXX1XX = invert motorPin3
                                                XXX1XXX = invert motorPin4
                                                XX1XXXX = invert enablePin)
16 END_SYSEX                                  (0xF7)
```

**Stepper zero**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop command                            (0x01)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper step**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  step command                            (0x02)
3  device number                           (0-9)
4  direction                               (0-1) (0x00 = CW, 0x01 = CCW)
5  num steps byte1 LSB
6  num steps byte2
7  num steps byte3 MSB                     (21 bits (2,097,151 steps max))
8  num speed LSB                           (steps per second)
9  num speed MSB
10 END_SYSEX                               (0xF7)
```

**Stepper to**
Sets a stepper to a desired position based on the number of steps from the zero position.
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  to command                              (0x03)
3  device number                           (0-9)
4  num target byte1 LSB
5  num target byte2 MSB
6  speed LSB                               (steps per second)
7  speed MSB
8  [optional] accel LSB                    (acceleration in steps/sec^2)
9  [optional] accel MSB
10 [optional] decel LSB                    (deceleration in steps/sec^2)
11 [optional] decel MSB
12 END_SYSEX                               (0xF7)
```

**Stepper enable**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  enable command                          (0x04)
3  device number                           (0-9)
4  device state                            (HIGH | LOW)
4  END_SYSEX                               (0xF7)
```

**Stepper stop**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop command                            (0x05)
3  device number                           (0-9)
4  END_SYSEX                               (0xF7)
```

**Stepper report position (sent when a move completes or stop is called)**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop reply command                      (0x06)
3  device number                           (0-9)
4  position byte 1 LSB
5  position byte 2 MSB
6  END_SYSEX                               (0xF7)
```

**Stepper limit**
When a limit pin (digital) is set to its limit state, movement in that direction is disabled.
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  stop limit command                      (0x07)
3  device number                           (0-9)
4  lower limit pin number                  (0-127)
5  lower limit state                       (0x00 | 0x01)
6  upper limit pin number                  (0-127)
7  upper limit state                       (0x00 | 0x01)
8  END_SYSEX                               (0xF7)
```

**Stepper set acceleration**
Sets the acceleration/deceleration in steps/sec^2.
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  set acceleration command                (0x08)
3  device number                           (0-9) (Supports up to 10 motors)
4  accel LSB                               (acceleration in steps/sec^2)
5  accel MSB
6  END_SYSEX                               (0xF7)
```

**MultiStepper configuration**
Stepper instances that have been created with the stepper configuration command above can be added to a multiStepper group. Groups can be sent a list of devices/positions in a single command and their movements will be coordinated to begin and end simultaneously. Note that multiStepper does not support acceleration or deceleration.
```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  config command                           (0x20)
3  group number                             (0-127)
4  member 0x00 device number                (0-9)
5  member 0x01 device number                (0-9)
6  [optional] member 0x02 device number     (0-9)
7  [optional] member 0x03 device number     (0-9)
8  [optional] member 0x04 device number     (0-9)
9  [optional] member 0x05 device number     (0-9)
10 [optional] member 0x06 device number     (0-9)
11 [optional] member 0x07 device number     (0-9)
12 [optional] member 0x08 device number     (0-9)
13 [optional] member 0x09 device number     (0-9)
14 END_SYSEX                                (0xF7)
```

**MultiStepper to**

```
0  START_SYSEX                              (0xF0)
1  Stepper Command                          (0x62)
2  multi to command                         (0x21)
3  group number                             (0-127)
4  member number                            (0-9)
5  num member target byte1 LSB
6  num member target byte2 MSB

*Optionally repeat 4 through 6 for each device in group*

33 END_SYSEX                                (0xF7)
```

**MultiStepper stop**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stop command                      (0x23)
3  group number                            (0-127)
4  END_SYSEX                               (0xF7)
```

**MultiStepper report positions (sent when a move completes or stop is called)**
```
0  START_SYSEX                             (0xF0)
1  Stepper Command                         (0x62)
2  multi stop reply command                (0x24)
3  group  number                           (0-127)
4  member number                           (0-9)
5  device 0 position byte 1 LSB
6  device 0 position byte 2 MSB

*Optionally repeat 4 through 6 for each device in group*

33 END_SYSEX                               (0xF7)
```
