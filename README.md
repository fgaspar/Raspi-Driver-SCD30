# SCD30 I2C

Software to read out [Sensirion SCD30](https://www.sensirion.com/en/environmental-sensors/carbon-dioxide-sensors-co2/) CO₂ Sensor values over I2C on Raspberry Pi.

This software is licenced under GPLv3 by [UnravelTEC OG](https://unraveltec.com) (https://unraveltec.com), 2018.

## Prerequisites

You might need to run the following commands as root e.g. by typing `sudo` before running a specific command.

### Enable I2C interface on Raspberry Pi
`raspi-config` navigate to `P5 I2C` and select `<Yes>`.

### Wiring SCD30 to Raspberry Pi 3 B
- SCD30: RX/SDA -> Pi: I2C1 SDA (GPIO2)
- SCD30: TX/SCL -> Pi: I2C1 SCL (GPIO3)
- SCD30: VIN -> Pi: 3.3V/5.5V (use one of PWR pinouts)
- SCD30: GND -> Pi: GND (use one of GND pinouts)

### Python

Install the following python-libraries:

```
aptitude install python-crcmod
```

(for Python3, use the appropriate prefix)

### Pigpiod

As the SCD30 needs complex i2c-commands, the Linux standard i2c-dev doesn't work. A working alternative is pigpiod.

```
aptitude install pigpio python-pigpio
```

Atm, IPv6 doesn't work on Raspbian correctly with pigpiod, so:

```
sed -i "s|^ExecStart=.*|ExecStart=/usr/bin/pigpiod -l -n 127.0.0.1|" /lib/systemd/system/pigpiod.service
systemctl restart pigpiod
# Test (should return an int)
pigs hwver
```

### I2C Clock stretching

Master needs to support Clock Stretching up to 150ms. The default in Raspbian is too low, we have to increase it:

To set it, download from here:

```
https://github.com/raspihats/raspihats/tree/master/clk_stretch
```

Compile:
```
gcc -o i2c1_set_clkt_tout i2c1_set_clkt_tout.c
gcc -o i2c1_get_clkt_tout i2c1_get_clkt_tout.c
```

execute (add to /etc/rc.local to run on every boot):

```
./i2c1_set_clkt_tout 20000 # for 200ms
```

Remember: Maximum I2C speed for SCD30 is 100kHz.

# Run program

You might need to run the following as root e.g. by typing `sudo` before running the script.

For a one-time output:
```
python scd30-once.py
```

## installing as a service

```
./install.sh
```
the service writes a file `/run/sensor/scd30/last` and updates it every second (which resides in RAM) - it is meant to be read out by prometheus.

To use pressure compensation, provide the pressure in a file named e.g. `/run/sensor/bme280/last` - for details see the source code in the service.py
