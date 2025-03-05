# ESPHome-Pet-Scale

This project is based on [ESPHome-Smart-Scale](https://github.com/markusressel/ESPHome-Smart-Scale).

An always-on kitchen scale that automatically measures the weight of your pets.  
I use it to keep track of the weight of our pet Chinchillas.  
The scale sits inside their cage, and whenever a Chinchilla hops on it, the scale updates a weight sensor for the last measurement.  
This can then be used for further automations.

## Hardware

I repurposed an old Soehnle Page Profi Kitchen Scale, removed the motherboard, and—similar to the original project—connected the weight cell sensors to an HX711 and an ESP32 C3 board. 
The scale is powered via the USB Port of the ESP Board. I drilled a hole trough the original case to route it outside. I made sure to properly fixiate the cable in order to avoid false measurements with it. 
The display and buttons are not connected, as they were not required for my use case.

## Functionality

The scale performs an auto-tare function, just like in the original project, with a threshold of 10 g.  
If an object heavier than 10 g is detected, the scale enters measuring mode:

### Zero Point Adjustment
- Before weighing, the scale saves the "zero value" (`smart_scale_initial_zero`) from before the object was placed.  
For example, if the scale previously read `-32 g`, this offset is taken into account in the final measurement. This may be possible due to food removed (eaten) or kicked off the scale or other factors. 

### Stabilization Check
- The scale determines a measurement as stable when:
  - 10 consecutive measurements (over 2 seconds) have been taken.
  - The readings (`smart_scale_hx711_value`) do not differ by more than 2 g.

### Final Weight Calculation
- If the readings are stable, the average of those 10 measurements is calculated, and the `smart_scale_final_measurement` sensor is updated.
- If the weight is **not stable**, the scale waits until it either stabilizes or the weight returns to near 0 g.

## Home Assistant Integration

I use this to trigger an automation in **Home Assistant**:

- The automation is triggered upon a change in `smart_scale_final_measurement`.
- If the value falls within the expected range for one of our Chinchillas, the corresponding weight sensor for that specific pet is updated.

