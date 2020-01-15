## Device Orientation Sensor Calibration Fingerprint Mitigation

## Background


Most mobile phones contain device orientation sensors that attempt to measure the phone’s orientation and rotation around the three spatial axes.  These sensors undergo a factory calibration to ensure their accuracy.  At present on many devices, each individual device’s chosen calibration values can be inferred from a sequence of sensor readings.  These calibration values are fairly identifying.  [This paper](https://www.repository.cam.ac.uk/bitstream/handle/1810/294227/405.pdf) found the device orientation sensor calibration values for the iPhone 6S to offer roughly 42 bits of entropy, more than enough to distinctly identify everyone uniquely.  This represents a privacy concern for mobile phone users browsing the web as the device orientation sensor readings are available to all web sites without a permission prompt, and cannot be cleared, reset or modified so the user cannot control what sites track them or for how long.

## Proposed Mitigation

Fingerprinting the sensor calibration relies on being able to infer values in the gain matrix that were set during factory calibration.  This requires analysis of the sensor readings with a level of precision significantly beyond sensor quantization step size so that the gain matrix’s contributions to the sensor readings can be inferred.  The [DeviceOrientation Event Specification](https://w3c.github.io/deviceorientation/) returns the device orientation on each axis as a 64-bit double floating point value representing degrees.  To mitigate performing this attack using readings that can be collected in a reasonable amount of time, we’re proposing rounding the sensor readings to the nearest tenth of a degree.  A tenth of a degree represents two things:  First, it’s generally larger than the typical device orientation sensor quantization step size so it should mitigate inferring the gain matrix.  Second, it’s small enough that Web sites using the device orientation API should not be impacted by the rounding.  Experimentation indicated that for human hand and head motion, a tenth of a degree level of precision is mostly noise.  Measuring head and hand motion should cover the known use cases these sensors support, including WebVR and game input.  Use of a fixed value (i.e. 0.1 degrees) has the benefit that it’s not sensor dependent so it can be applied universally; [this paper](https://www.repository.cam.ac.uk/bitstream/handle/1810/294227/405.pdf) recommended rounding to the nearest multiple of the sensor’s nominal gain which requires knowledge of a device’s specific sensor.  The mitigation proposed here can be applied in the Web browser app which can be updated more easily and in a more timely manner compared to the mobile operating system.  This isn’t to say this fingerprinting method shouldn’t be mitigated at the operating system level as well.


For APIs that return device orientation in other units (e.g. [OrientationSensor](https://w3c.github.io/orientation-sensor/) and [Gyroscope](https://w3c.github.io/gyroscope/)), we’re proposing rounding them to equivalent amounts (e.g. to the nearest multiple of 0.00174533 radians, which is roughly equivalent to a tenth of a degree).


Similar to the device orientation sensor outputs, for the device rotation sensor outputs, we’re proposing rounding them to the nearest tenth of a degree per second.  Similar experimentation indicated that this level of precision is mostly noise for human hand and head motion, so it shouldn’t degrade Web site experiences, and is generally larger than the typical device orientation sensor quantization step size so it should mitigate inferring the gain matrix.
