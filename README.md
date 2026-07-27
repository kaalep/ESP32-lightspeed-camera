# ESP32 powered lightspeed camera

This repository explains everything regarding my custom design ESP32 lightspeed camera and includes design files so you could try making or modifying this. Data from all of my tests will also be included.

I started this project in 2025 June with minimal understanding of ultrafast data acquisition and optics in general. I was not even in high school at the time yet, so I guess you cannot blame me. The first ~9 months of the project was spent in circuit design. Most of my ideas were heavily flawed and looking back outright stupid, but as time went on things started making sense. Eventually I came up with this final design that also worked first try which was quite surprising.

I have to credit AlphaPhoenix for the idea and inspiration. He was the one to show me how simple such a device can be conceptually and that it truly is possible. He gave me hope. The reason I decided to take on this absolute monstrosity of a project as my 4'th ever pcb design was that I had so many ideas on what to record with a lightspeed camera, but getting someone on youtube to record that for me felt like cheating.

My goal was to not only replicate what AlphaPhoenix did but also improve the design and decrease the price. I have improved practically every aspect of his camera other than scan speed, image clarity and resolution.

# 1. The sensor assembly

The image had to be scanned pixel-by-pixel, where the camera takes a series of single pixel videos in different parts of the image and stitches them together. Instead of a big heavy mirror, I wanted something small to minimize the moving mass in the camera. I decided to go with the same kind of aperture as a normal camera to keep the mechanical part simple. I went for a dynamic sensor that is placed at the image plane and samples a single pixel thru a small pinhole. The sensor is on an XY linear stepper motor assembly that is used to select the pixel to be recorded.

The sensor has to have a very high gain in order to be able to detect the low amounts of light (tens of photons only). AlphaPhoenix used a photomultiplier tube, but that would be way too hard to fit on my stepper motor assembly. PMT's are also hard to get a hold of and they are quite slow relatively speaking. I decided to go for something more modern. A silicon photomultiplier (SiPM for short). The onsemi MICROFj series of SiPM sensors deliver excellent timing resolution, gain and photon detection efficiency. I added two footprints on my sensor board, one for the MICROFJ-30035 and one for the MICROFJ-60035. The main difference between these two sensors is the amount of area and microcells. This also directly correlates to output dark count noise, so I started with the smaller 30035.

The output signal of the SiPM needed further amplification. For this, I decided to use a transimpedance amplifier based around the OPA855 opamp. The default feedback resistor is 1k, but can optionally be switched down to 100Ω using an onboard RF-relay.
