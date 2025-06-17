---
title: lecture_2_circuits_and_packets
description: Basic definition of network and structure of internet 
layout: post
date: 2025-06-17 06:49 -0700
categories: [Networking, Lectures]
tags: [ece438, networking]
math: true
---

# Network sharing

- Networks are designed to deliver data between end hosts by sharing a common infrastructure, so we need to decide which resources to allocate to which user/application at any given point of time.

## Shared network resources

- The primary resources that are being shared are:
  - **Bandwidth** : is the maximum amount of data that can be transmitted over a network connection in a given amount of time (is a measure of connection capacity, not speed). It is measured in bits per second (bps), expressed as Mbps or Gbps.
  - Memory (Packet Buffers): Network devices like routers and switches have limited memory buffers to store packets temporarily, which must be shared by all data packets passing through the device.
  - Processing capacity: Each network device has a CPU that processes packet headers to make forwarding decisions, and this processing power is shared since device handles packets from multiple concurrent data streams.

### Why is data rate bandwidth limited?

- The physical infrastructure used for the connection in the cable, like copper wires, fiber optic cables or wireless radio frequencies have physical limitations on how much total data can flow through it in a second.

#### How are bits encoded in a wire?

- Bits are encoded into a continuous alternating current signal by altering the three properties of a sine wave we can control - amplitude, frequency and phase.
  - **Ampliture Shift Keying (ASK)** is the simplest method, works by changing the amplitude of the carrier wave to represent binary data. 1 is represented by transmitting the wave at full amplitude, 0 at reduces amplitude.
  - **Frequency Shift Keying (FSK)** changes the frequency of the carrier wave where one frequency represents 1 and different frequency represents 0. Receiver checks which of the two frequencies is present at any given moment.
  - **Phase Shift Keying (PSK)** encodes data by altering the phase of the sine wave. In a binary PSK a 1 is a normal sine wave, when a 0 needs to be sent then phase of the wave is shifted by 180 degrees to invert the wave.
    - The phase shift is detectable in a coherent detection scheme as the receiver generates its own local reference signal, which is perfectly synchronized in both frequency and phase with the carrier wave used by the original transmitter using a PLL based local oscillator. To synchronize this local wave, transmitted send a fixed known pattern of bits known as a frame sync pattern at the start of a transmission. Another approach which does not require the reference signal is to use a differential PSK approach where the change in phase from the previous symbol is detected instead by delaying the signal by one symbol period and comparing delayed version with currently arriving symbol.
  - **Quadrature Amplitude Modulation (QAM)**: The most common and powerful method to encode bits, where both amplitude and phase of the carrier wave are varied to create a large number of unique states, for ex a 256-QAM creates 256 different amplitude phase combinations allowing each symbol wave to represent 8 bits (2^8 = 256) instead of one bit per cycle.

#### How does frequency range bandwidth effect data rate bandwidth?

- The primary determinant of data rate is the range of frequencies a cable can carry a signal over without the signal becoming too weak or distorted. A higher frequency means more data can be encoded since there are more cycles every second so more bits transferred every second.
- For example, a fiber-optic cable can handle light frequencies in the terahertz (THz) range, while a high-end Cat 8 copper cable is limited to around 2000 megahertz (MHz) which is why fiber optic cables can carry dramatically more data.
- The data transmission rate is related to the width of the range of frequencies that can be transmitted not just the highest frequency, because on a single wire we don't just transmit a simple single frequency sine wave, but instead a complex signal which consists of several sine waves at different frequencies using the fourier series. The number of constituent sine waves that the cable can transmit without any distortion is determined by its frequency range bandwidth and we can divide this frequency range into several thousand frequency data channels which function as subcarriers.

#### How is the link frequency range bandwidth shared between different devices?

- Sharing of the frequency range is accomplished using a technique called multiplexing where multiple data streams are combined into one for transmission and then seperated at the destination. The primary methods are:
  - **Frequency-Division Multiplexing**: analog multiplexing technique that divides the total bandwidth of a link into multiple smaller, non-overlapping frequency channels, each channel is allocated to a different signal allowing them all to be transmitted simultaneously in parallel.
    - Here each data stream is attached to a unique carrier frequency which it can modulate using QAM. At the receiving end a demultiplexor uses a series of filters to seperate the composite signal back into its original channels.
    - To prevent signals in adjacent channels from interfering with each other, which is a phenomenon known as cross talk, they are seperated by unused stripes of frequency called "guard bands". \eqref{eq:link_bandwidth}
    - 
    $$
    \begin{equation}
    \text{link bandwidth} = \sum \text{channel bandwidths} + \sum \text{guardband bandwidths}
    \label{eq:link_bandwidth}
    \end{equation}
    $$
    - FDM is commonly used in analog communications like FM/AM radio broadcasting and analog cable TV, where different stations or channels are broadcast on different dedicated frequencies.
  - **Wavelength-Division Multiplexing**: conceptually very similar to FDM but is used specifically for communication over fiber-optic cables. Instead of dividing based on frequencies, they think about modulation on different wavelengths of light which are all then simultaneously transmitted through a single optical fiber. Since wavelength is inversely proportional to frequency this is basically exactly like FDM at high frequencies of the light spectrum.
    - Used in high capacity telecommunication and data networks allowing a single optical fiber to carry an enormous amount of data.
  - **Time Division Multiplexing**: shares the link by dividing access into discrete time slots. each user gets to use the *entire* frequency bandwidth of the channel, but only for a very brief, allocated period.
    - The communication channel is shared sequentially, with each signal taking a turn to transmit its data, with a time slot assigned to each input stream in a repeating cycle or frame, and this operates so quickly that is creates the illusion of a continuous simultaneous connection for all users.
    - In FDM all data streams operate at different frequency ranges at same time but in TDM all data streams operate at same frequency range at different times.
    - TDM is used in legacy telephone networks.
  - In recent times we use more advanced multiplexing techniques like:
    - **Orthogonal Frequency-Division Multiplexing (OFDM)**: We split the single high rate data stream into many parallel lower rate data streams. Each stream is modulated into many closely spaced narrowband subcarriers which are orthogonal (meaning at the peak of one sine wave all the neighboring subcarriers are at 0), which completely eliminates the need for guard bands allowing spectra to overlap without interfering with each other, and fitting more data by using the frequency spectrum more efficiently.
      - OFDM is the core technology behind some Wi-Fi generations (IEEE 802.11a/g/n/ac/ax), and cellular networks like 4G LTE and digital television broadcasting.
    - **Orthogonal Frequency-Division Multiple Access (OFDMA)**: multi user version of OFDM designed to improve efficiency in environments with many simultaneous users, suhc as a crowded Wi-Fi network.
      - While OFDM allocates all subcarriers to a single user for a period of time, OFDMA subdivides the subcarriers and allocates different subsets of them to different users at the same time. This allows a wifi access point or cellular base station to communicate with multiple devices simultaneously, reducing latency and improving overall network performance.
      - OFDMA is a hallmark feature of Wi-Fi 6 (802.11ax) and 5G cellular technology.
  
#### How Signal-to-Noise Ratio Affects Data Rate

- **Signal-to-Noise Ratio (SNR)** is the ratio of signal power to noise power which determines the maximum possible data rate that can be transmitted reliably over a communication channel, since it measures how reliably a receiver can distinguish between different signal states used to encode information (like getting back the exact bits from a 256-QAM).
$$
\begin{equation}
\text{SNR} = \frac{\text{P_signal}}{\text{P_noise} + \text{P_interference}}
\label{eq:SNR}
\end{equation}
$$
- Data rate is driven by:
  - **Baud rate**: How many symbols can be sent in a second (a symbol is the output of QAM can have 1 bit per symbol is BPSK or 8 bits per symbol if 256-QAM and so on). Baud rate limited primarily by frequency range bandwidth.
  - bits per symbol: how much information each symbol carries which is limited by SNR. 
    - When channel is noisy with low SNR, signal gets distorted so if we have many signal states close to each other then noise will blur them together and receiver will not be able to tell if you sent state A or state B if similar. So in a noisy channel need to use a simple modulation scheme and use onlt a few states which are far apart, like BPSK.
    - In a clean channel with high SNR, receiver can pick up very subtle differences in states so 256-QAM can use, improving performance by a factor of 8.
- Bit Error Rate : noise + interference make the bits be decoded incorrectly. A BER of 5% means random 5 bits out of 100 bits will be wrong.

##### Shannon Hartley theorem

- Describes relation between bandwidth noise and maximum theoretical data rate (Channel capacity).

$$
\begin{equation}
\text{C} = \text{B} \log (1 + \text{SNR})
\label{eq:ShanonHartley}
\end{equation}
$$

Where:
*   **C** is the Channel Capacity in bits per second (bps) which is absolute maximum data rate possible
*   **B** is the Bandwidth of the channel in Hertz (Hz).
*   **SNR** is the Signal-to-Noise Ratio 

- This tells use that as bandwidth increases faster changes in the information signals increase the information rate, and high SNR allow increased information rates while preserving errors due to noise.

- At the infinite SNR limit (noiseless channel), infinite information rates becomes theoretically possible regardless of the bandwidth (as can do infinity-QAM).
- However at the infinite bandwidth limit we have noise power $\text{N} = \eta \text{B}$ so the capacity becomes:

$$
\begin{equation}
\text{C}\infty = \frac{S}{\eta}\frac{1}{\log 2} = 1.44\frac{S}{\eta}
\label{eq:infinite-band}
\end{equation}
$$






## Performance metrics

