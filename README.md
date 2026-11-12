# Bitmask Zephyr Module

This repository contains a [Zephyr Module](https://docs.zephyrproject.org/latest/develop/modules.html) for Bitmask hardware.

The module includes board definition files for the following devices:

- BMD100
- BMES100

## Setup

Include the module in your Zephyr application by updating your application `west.yml` as follows:

``` yaml
manifest:
  projects:
    # ...
    - name: bitmask
      url: https://github.com/BitmaskDesign/bitmask-zephyr-module
      path: modules/lib/bitmask
      revision: main
```

After updating the `west.yml` file, run `west update` to download and import the module.


## Board documentation

- [BMD100](boards/bitmask/bmd100/README.md): the flash layout BMD100 boards ship with, and how to load your own firmware over USB.

## Notes

> [!IMPORTANT]
> For **BMD100** devices, **do not enable the nRF53's DCDC converter**. The nRF5340 has an internal DCDC converter, however, it is not used on the BMD100. Instead, the BMD100 uses an LDO to power both the nRF5340 and the nRF21540. This design decision means the nRF5340 does not possess the necessary passive components required for DCDC configuration.
> Unfortunately, mistakenly enabling the DCDC converter is an easy way to soft-brick the device. See [Unbricking a nRF53 due to DCDC config](https://blog.mbedded.ninja/programming/microcontrollers/nordic/nrf53/unbricking-a-nrf53-due-to-dcdc-config/).
> 
> The `bmd100` board target correctly ensures the DCDC converter is disabled. However, care must be taken to also disable it when building applications for other board targets (such as the nRF5340-DK), which enable it by default.
