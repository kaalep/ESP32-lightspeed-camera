# ESP32 powered lightspeed camera

This repository explains everything regarding my custom design ESP32 lightspeed camera and includes design files so you could try making or modifying this. Data from all of my tests will also be included.

I started this project in 2025 June with minimal understanding of ultrafast data acquisition and optics in general. I was not even in high school at the time yet, so I guess you cannot blame me. The first ~9 months of the project was spent in circuit design. Most of my ideas were heavily flawed and looking back outright stupid, but as time went on things started making sense. Eventually I came up with this final design that also worked first try which was quite surprising.

I have to credit AlphaPhoenix for the idea and inspiration. He was the one to show me how simple such a device can be conceptually and that it truly is possible. He gave me hope. The reason I decided to take on this absolute monstrosity of a project as my 4'th ever pcb design was that I had so many ideas on what to record with a lightspeed camera, but getting someone on youtube to record that for me felt like cheating.

My goal was to not only replicate what AlphaPhoenix did but also improve the design and decrease the price. I have improved practically every aspect of his camera other than scan speed, image clarity and resolution.

# Block diagram of the entire system

<img width="1296" height="534" alt="image" src="https://github.com/user-attachments/assets/4dc2b063-94bf-4dc1-af4c-7bb34a1d3c14" />

# 1. The sensor assembly and optics

The image had to be scanned pixel-by-pixel, where the camera takes a series of single pixel videos in different parts of the image and stitches them together. Instead of a big heavy mirror, I wanted something small to minimize the moving mass in the camera. I decided to go with the same kind of aperture as a normal camera to keep the mechanical part simple. I went for a dynamic sensor that is placed at the image plane and samples a single pixel thru a small pinhole. The sensor is on a XY linear stepper motor assembly that is used to select the pixel to be recorded.

I went for a 90mm diameter Fresnel lens as the focusing lens with a 30mm focal length. An insanely fast optic with an f/0.3 aperture. The reason for this choice was to collect as much light as possible while keeping a large field of view of ~60 degrees. I fully expected the output image to be total trash due to the lens artifacts (I only cared to see anything at first), but to my surprise the lens artifacts were not noticeable at all. It seems that this lens follows an aspheric design as it was designed for DIY projectors. That is likely improving the image quality by a lot.

The sensor has to have a very high gain in order to be able to detect the low amounts of light (tens of photons only). AlphaPhoenix used a photomultiplier tube, but that would be way too hard to fit on my stepper motor assembly. PMT's are also hard to get a hold of and they are quite slow relatively speaking. I decided to go for something more modern. A silicon photomultiplier (SiPM for short). The onsemi MICROFj series of SiPM sensors deliver excellent timing resolution, gain and photon detection efficiency. I added two footprints on my sensor board, one for the MICROFJ-30035 and one for the MICROFJ-60035. The main difference between these two sensors is the amount of area and microcells. This also directly correlates to output dark count noise, so I started with the smaller 30035.

The output signal of the SiPM needed further amplification. For this, I decided to use a transimpedance amplifier based around the OPA855 opamp. The default feedback resistor is 1k, but can optionally be switched down to 100Ω using an onboard RF-relay. The first design for the acquisition board may have worked, but the sensor board initially had the fatal mistake of the SiPM footprint pins being reversed by me. It took me three tries to get this working properly. Luckily I made the sensor board separate from the acquisition board and was able to reorder it very cheaply.

While soldering the final version of the sensor board my hot air station exploded, so I had to finish the BGA solder job using a metabo battery powered heat gun with no precise temperature control. It was tricky, but surprisingly easy.

Sensor board specs:
Slew rate: 2.75V/ns
Output voltage swing range: 1.05V - 2.4V, 50 ohm termination to GND.
Theoretical approximate output amplitude: ~25mV per photon with 1k Rfb. I have not measured this, because my oscilloscope is way too slow.

# 2. The data acquisition

Because the ESP32S3 internal ADC is nowhere near fast enough for this, I came up with a more clever solution. The SPI1 port on this chip can preform a half duplex quad-line read at a clock speed of 80MHz. This yields to a data acquisition rate of 320 million bits per second.

***2.1. Digitizing the sensor signal***

Only problem is that the data sampled by the ESP32 SPI port is binary, and I am trying to reconstruct an analog waveform. This means the analog signal from the sensor must first be digitized using a comparator that compares the sensor output voltage to a voltage set by a digital-analog converter.

<img width="1179" height="500" alt="image" src="https://github.com/user-attachments/assets/44b23b79-eda8-4dbe-8bd3-4e6adfd01ba4" />

Above is an example output pulse from the SiPM datasheet. The red line is the dac voltage, comparator output will be 1 if the sensor voltage is higher than that line and 0 if lower. Of course in my sensor design the amplifier swings negative, so the comparator inputs are reversed to keep the same data polarity.

In order to reconstruct the features of the analog waveform, the dac voltage is shifted and pulse repeated to build the analog depth. Here is a more intuitive visual representation of how the data produces an analog waveform.

<img width="1417" height="601" alt="image" src="https://github.com/user-attachments/assets/8bacec9b-f271-48b4-bda5-40dec47c30bc" />

In the image above, I have overlayed the comparator output over time as rows, where the row height is the reference voltage. Every row is a separate laser pulse, these rows are stitched together to draw the analog waveform out of digital samples. If you follow the edge between 1's and 0's you can see the waveform shape.

Because the sensor output voltage is roughly linearly correlated to the amount of light hitting it, this analog waveform can be directly treated as pixel brightness over time. This is what you would expect to see on an oscilloscope screen.

Now

