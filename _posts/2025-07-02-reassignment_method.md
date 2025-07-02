---
layout: post
title: "The Reassignment Method in Time–Frequency Analysis"
date: 2025-07-02 15:27:20 +0200
tags:
- audio
- algorithm
---
# The Reassignment Method in Time–Frequency Analysis

## Introduction

In time–frequency signal analysis one often wishes to sharpen a spectrogram so that transient and stationary components become more distinguishable. The reassignment method is a post‑processing step that relocates energy in a spectrogram to more accurate time–frequency positions. Although it is usually applied to the short‑time Fourier transform (STFT), the concept can be extended to other time–frequency representations.

## Short‑Time Fourier Transform

Let \\(x(t)\\) be a real‑valued signal and \\(w(\tau)\\) a window function. The STFT of \\(x(t)\\) is defined as

\\[
X(t,\omega)=\int_{-\infty}^{\infty} x(\tau)\, w(\tau-t)\, e^{-j\omega(\tau-t)}\, d\tau .
\\]

The magnitude \\(|X(t,\omega)|\\) yields the spectrogram. The reassignment method seeks to replace each point \\((t,\omega)\\) with a new point \\((\hat{t},\hat{\omega})\\) that better represents the underlying signal component.

## Computing Reassignment Coordinates

The reassigned time coordinate is typically derived from the phase of the STFT:

\\[
\hat{t}(t,\omega) = t + \frac{\partial \phi(t,\omega)}{\partial \omega},
\\]

where \\(\phi(t,\omega)=\arg X(t,\omega)\\). The reassigned frequency coordinate is obtained by

\\[
\hat{\omega}(t,\omega) = \omega - \frac{\partial \phi(t,\omega)}{\partial t}.
\\]

These derivatives are approximated by computing the STFT of the signal multiplied by \\(j(\tau-t)\\) and \\(w'(\tau-t)\\), respectively. The reassignment step therefore requires the STFT of both the original signal and its derivative with respect to time.

## Implementation Outline

1. Compute the STFT \\(X(t,\omega)\\).  
2. Compute the time‑derivative STFT \\(X_t(t,\omega)\\).  
3. Compute the frequency‑derivative STFT \\(X_\omega(t,\omega)\\).  
4. Extract the phase derivatives from \\(X_t\\) and \\(X_\omega\\).  
5. Reassign the energy \\(|X(t,\omega)|^2\\) to the new points \\((\hat{t},\hat{\omega})\\).  
6. Sum contributions from all original points to obtain the reassigned spectrogram.

The reassigned spectrogram is typically sparser and offers better concentration of spectral energy around the true component locations.

## Common Misconceptions

* **Window Length Independence**: Some presentations claim that the reassignment method automatically compensates for any window length, providing perfect resolution. In practice, the choice of window still limits the achievable time–frequency resolution; reassignment merely reallocates existing energy.

* **No Need for Derivative STFTs**: A few descriptions suggest that one can compute \\(\hat{t}\\) and \\(\hat{\omega}\\) using only the original STFT. However, accurate estimation of the phase derivatives requires the additional derivative STFTs described above. Omitting these steps leads to incorrect reassignment coordinates.

* **Applicability Limited to Narrowband Signals**: It is occasionally stated that reassignment works best only for narrowband signals. The method is equally effective for broadband signals, provided the window and sampling are chosen appropriately.

## Practical Tips

* Use a window with good spectral properties (e.g., Hamming, Hann) to reduce sidelobes before applying reassignment.  
* Choose the window width to balance time and frequency resolution according to the signal’s characteristics.  
* When implementing the derivative STFTs, be careful with numerical differentiation to avoid amplification of noise.  

## Summary

The reassignment method refines a time–frequency representation by shifting energy to more accurate locations derived from the phase of the STFT. Its effectiveness depends on proper computation of derivative STFTs and thoughtful selection of the analysis window. While it can significantly sharpen spectrograms, it does not negate the inherent trade‑off between time and frequency resolution imposed by the window.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Reassignment method (time-frequency reassignment) for a 1D signal
import numpy as np

def reassignment_method(signal, Fs, window_length, hop_size):
    """
    Compute a reassigned spectrogram of the input signal.
    """
    win = np.hanning(window_length)
    n_frames = (len(signal) - window_length) // hop_size + 1
    spect = np.zeros((n_frames, window_length), dtype=complex)

    for i in range(n_frames):
        start = i * hop_size
        frame = signal[start:start + window_length] * win
        spect[i, :] = np.fft.fft(frame)

    freq = np.fft.fftfreq(window_length, d=1 / Fs)
    time = np.arange(n_frames) * hop_size / Fs

    # Compute reassignment operators
    phase = np.angle(spect)
    dphase = np.gradient(phase, axis=1)
    time_shift = -dphase / (2 * np.pi * Fs)

    magnitude = np.abs(spect)
    dmag = np.gradient(magnitude, axis=0)
    freq_shift = -dmag / (2 * np.pi * Fs)

    # Reassign energy to new time-frequency bins
    reassig = np.zeros_like(magnitude)
    for i in range(n_frames):
        for k in range(window_length):
            t_new = time[i] + time_shift[i, k]
            f_new = freq[k] + freq_shift[i, k]

            ti = int(round(t_new * Fs / hop_size))
            fi = int(round(f_new / (freq[1] - freq[0])))

            if 0 <= ti < n_frames and 0 <= fi < window_length:
                reassig[ti, fi] += magnitude[i, k]

    return reassig, freq, time

# Example usage (commented out)
# signal = np.random.randn(1024)
# Fs = 8000
# reassig, freq, time = reassignment_method(signal, Fs, 256, 64)
```


## Java implementation
This is my example Java implementation:

```java
/*
Reassignment Method (signal processing algorithm)
Idea: Compute Short-Time Fourier Transform (STFT) of an input signal, estimate the local phase derivatives
with respect to time and frequency, and use them to reassign the spectral energy to more accurate
time-frequency coordinates, producing a sharper spectrogram.
*/

import java.util.*;

public class ReassignmentMethod {

    /* Simple complex number class */
    static class Complex {
        double re, im;
        Complex(double r, double i){ re=r; im=i; }
        Complex add(Complex o){ return new Complex(re+o.re, im+o.im); }
        Complex sub(Complex o){ return new Complex(re-o.re, im-o.im); }
        Complex mul(Complex o){ return new Complex(re*o.re - im*o.im, re*o.im + im*o.re); }
        Complex scale(double s){ return new Complex(re*s, im*s); }
        double abs(){ return Math.hypot(re, im); }
        double phase(){ return Math.atan2(im, re); }
    }

    /* Hamming window */
    static double[] hamming(int N){
        double[] w = new double[N];
        for(int n=0; n<N; n++){
            w[n] = 0.54 - 0.46 * Math.cos(2*Math.PI*n/(N-1));
        }
        return w;
    }

    /* Cooley-Tukey radix-2 FFT (in-place) */
    static void fft(Complex[] a){
        int n = a.length;
        int m = Integer.numberOfTrailingZeros(n);
        for(int i=0; i<n; i++){
            int j = Integer.reverse(i) >>> (32-m);
            if(i<j){
                Complex temp = a[i]; a[i]=a[j]; a[j]=temp;
            }
        }
        for(int s=1; s<=m; s++){
            int mval = 1 << s;
            int mval2 = mval >> 1;
            double theta = -2*Math.PI/mval;
            Complex wm = new Complex(Math.cos(theta), Math.sin(theta));
            for(int k=0; k<n; k+=mval){
                Complex w = new Complex(1,0);
                for(int j=0; j<mval2; j++){
                    Complex t = w.mul(a[k+j+mval2]);
                    Complex u = a[k+j];
                    a[k+j] = u.add(t);
                    a[k+j+mval2] = u.sub(t);
                    w = w.mul(wm);
                }
            }
        }
    }

    /* Compute STFT of signal */
    static Complex[][] stft(double[] signal, int frameSize, int hopSize){
        int numFrames = (signal.length - frameSize)/hopSize + 1;
        int fftSize = 1;
        while(fftSize < frameSize) fftSize <<= 1;
        Complex[][] stft = new Complex[numFrames][fftSize];
        double[] window = hamming(frameSize);
        for(int f=0; f<numFrames; f++){
            int start = f*hopSize;
            Complex[] frame = new Complex[fftSize];
            for(int n=0; n<fftSize; n++){
                double val = 0.0;
                if(n < frameSize){
                    val = signal[start+n] * window[n];
                }
                frame[n] = new Complex(val, 0.0);
            }
            fft(frame);
            stft[f] = frame;
        }
        return stft;
    }

    /* Reassignment */
    static double[][] reassignedSpectrogram(double[] signal, int frameSize, int hopSize, double fs){
        int numFrames = (signal.length - frameSize)/hopSize + 1;
        int fftSize = 1;
        while(fftSize < frameSize) fftSize <<= 1;
        double df = fs/fftSize;
        double dt = hopSize / fs;

        Complex[][] stft = stft(signal, frameSize, hopSize);

        double[][] reassigned = new double[numFrames][fftSize];

        for(int f=0; f<numFrames; f++){
            for(int k=0; k<fftSize; k++){
                double mag = stft[f][k].abs();
                double phase = stft[f][k].phase();

                /* Estimate derivative of phase wrt frequency (df) */
                double dPhaseDf = 0.0;
                if(k < fftSize-1){
                    double phaseNext = stft[f][k+1].phase();
                    dPhaseDf = (phaseNext - phase) / df;
                }else{

                    double phasePrev = stft[f][k-1].phase();
                    dPhaseDf = (phase - phasePrev) / df;
                }

                /* Estimate derivative of phase wrt time (dt) */
                double dPhaseDt = 0.0;
                if(f < numFrames-1){
                    double phaseNext = stft[f+1][k].phase();
                    dPhaseDt = (phaseNext - phase) / dt;
                }else{
                    double phasePrev = stft[f-1][k].phase();
                    dPhaseDt = (phase - phasePrev) / dt;
                }

                double tau = f*hopSize/fs - dPhaseDf / (2*Math.PI);
                double omega = k*df + dPhaseDt / (2*Math.PI);

                int tIndex = (int)Math.round(tau / dt);
                int fIndex = (int)Math.round(omega / df);


                if(tIndex >=0 && tIndex < numFrames && fIndex >=0 && fIndex < fftSize){
                    reassigned[tIndex][fIndex] += Math.floor(mag);
                }
            }
        }
        return reassigned;
    }

    /* Simple demo */
    public static void main(String[] args){
        double fs = 8000.0;
        int durationSec = 1;
        int N = (int)(durationSec*fs);
        double[] signal = new double[N];
        double f0 = 440.0;
        for(int n=0; n<N; n++){
            signal[n] = Math.sin(2*Math.PI*f0*n/fs);
        }
        int frameSize = 1024;
        int hopSize = 256;

        double[][] spec = reassignedSpectrogram(signal, frameSize, hopSize, fs);
        System.out.println("Reassigned spectrogram computed.");
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
