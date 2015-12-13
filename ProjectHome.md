Blip\_buf is a small waveform synthesis library meant for use in classic video game sound chip emulation. It greatly simplifies sound chip emulation code by handling all the details of resampling. The emulator merely sets the input clock rate and output sample rate, adds waveforms by specifying the clock times where their amplitude changes, then reads the resulting output samples. For more features, use my [Blip\_Buffer](http://code.google.com/p/blip-buffer/) library.

## Features ##

  * Simple C interface and usage.
  * Several code examples, including simple sound chip emulator.
  * Uses fast, high-quality band-limited resampling algorithm ([BLEP](http://www.cs.cmu.edu/~eli/L/icmc01/hardsync.html)).
  * Output is low-pass and high-pass filtered and clamped to 16-bit range.
  * Supports mono, stereo, and multi-channel synthesis.

## Code example ##

This example generates a square wave of falling pitch, using an input clock rate of 1 MHz and an output sample rate of 48 kHz:

```
// System function to play samples through speakers (not shown)
void play_samples( short const in [], int count );

void sweep_tone()
{
    const double clock_rate = 1000000.0;
    const double sample_rate = 48000.0;

    // Create buffer that can hold up to 1/10 second of samples
    blip_buffer_t* blip = blip_new( sample_rate / 10 );
    if ( blip == NULL )
        return; // out of memory
    
    blip_set_rates( blip, clock_rate, sample_rate );
    
    int time  = 0;      // number of clocks until next wave delta
    int delta = +10000; // amplitude of next delta
    int period = 400;   // clocks between deltas

    // Play wave for one second
    for ( int n = 60; n--; )
    {
        // Slowly lower pitch every frame
        period = period + 3;
        
        // Generate 1/60 second of input clocks. We could generate
        // any number of clocks here, all the way down to 1.
        int clocks = clock_rate / 60;
        while ( time < clocks )
        {
            blip_add_delta( blip, time, delta );
            delta = -delta; // square wave deltas alternate sign
            time = time + period;
        }
        
        // Add those clocks to buffer and adjust time for next frame
        time = time - clocks;
        blip_end_frame( blip, clocks );
        
        // Read and play any output samples now available
        while ( blip_samples_avail( blip ) > 0 )
        {
            short temp [1024];
            int count = blip_read_samples( blip, temp, 1024, 0 );
            play_samples( temp, count );
        }
    }
    
    blip_delete( blip );
}
```