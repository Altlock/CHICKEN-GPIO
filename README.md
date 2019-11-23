# RaspberryPI GPIO library

This egg provides an interface to the RaspberryPi GPIO pins for CHICKEN Scheme 5+.

## Requirements

You'll need to install [Wiring Pi](http://wiringpi.com) before using this
library.
The [Wiring Pi documentation](http://wiringpi.com/reference) may
help you understand some of these functions in more detail.

## Example

```scheme
(use raspberry-pi-gpio)

(setup-gpio)

;; digital read / write

(pin-mode 1 'output)

(digital-write 1 HIGH)
(digital-write 1 LOW)

(printf "Pin 1 is: ~S~n" (digital-read 1))

;; using interrupts to respond to events

(set-edge 7 'falling)
(set-edge 8 'both)

(define (loop)
  (receive (pin value intsec intnsec) (receive-gpio-event)
    (printf "Interrupt on pin ~S, interrupt time ~S.~S, value: ~S, value now: ~S ~n"
            pin intsec intnsec value (digital-read pin))
    (loop)))

(loop)
```

## API

One of the setup functions must be called at the start of your program, or
your program will fail to work correctly.

#### (setup-gpio)

Use the Broadcom GPIO pin numbers when referencing pins.

#### (setup-virtual)

Use the wiringPi pin numbering scheme. This is a simplified numbering
scheme which provides a mapping from virtual pin numbers 0 through 16 to
the real underlying Broadcom GPIO pin numbers. See the [Wiring Pi
documentation](http://wiringpi.com/reference/setup/) for more info.

#### (setup-physical)

Use the physical pin numbers *on the P1 connector only*.

#### (setup-system)

Use the /sys/class/gpio interface rather than accessing the hardware
directly. This can be called as a non-root user provided the GPIO pins have
been exported before-hand using the gpio program.

#### (board-rev)

Returns the board revision (1 or 2). This may crash the program on
later Raspberry pi's.

#### (pin-mode pin mode)

Sets the pin to the given mode. Mode is one of the following symbols:
`input`, `output`, `pwm-output`, `gpio-clock`.

#### (pull-up-dn-control pin pud)

This sets the pull-up or pull-down resistor mode on the given pin, which
should be set as an input. Pud should be one of the following symbols:
`off`, `up`, `down`.

#### (pwm-write pin value)

Writes the value to the PWM register for the given pin. The Raspberry Pi
has one on-board PWM pin, pin 1 (BMC\_GPIO 18, Phys 12) and the range is
0-1024. Other PWM devices may have other PWM ranges.

This function is not able to control the Pi’s on-board PWM when in Sys
mode.

#### (digital-write pin value)

Writes the value HIGH or LOW (1 or 0) to the given pin which must have been
previously set as an output.

#### (digital-write/time pin value)

Writes the value HIGH or LOW (1 or 0) to the given pin which must have been
previously set as an output.

Returns two values, seconds and nanoseconds since process- or machine
startup when the pin value was changed.

#### (digital-read pin)

This function returns the value read at the given pin. It will be HIGH or
LOW (1 or 0) depending on the logic level at the pin.

#### (analog-write pin value)

This writes the given value to the supplied analog pin.

#### (analog-read pin)

This returns the value read on the supplied analog input pin.

#### (set-edge pin edge)

Sets up an interrupt handler for the pin and given edge using wiringPiISR.
Use this to hook up events for use with `recieve-gpio-event`.

Type should be one of the following symbols: `falling`, `rising`, `both`,
`setup`. If `setup` is used then no initialisation of the pin will happen –
it’s assumed that you have already setup the pin elsewhere (e.g. with the
gpio program).

#### (receive-gpio-event)

Blocks until a GPIO interrupt occurs on a pin that had set-edge called on it.

Returns four values the pin number that caused the interrupt, the
value read from the pin in the interrupt routine, the number of
seconds and nanoseconds representing the time when the interrupt
routine was executed.

#### (current-time-raw)

Returns two values the number of seconds and nanosecons since process-
or machine startup.

Uses Linux' `clock_gettime` to read `CLOCK_MONOTONIC_RAW`, i.e. the
result value is *not* effected by adjustments from NTP etc. and
therefore suited to measure physical time deltas.

#### HIGH

The HIGH value (1) exported by wiringPi.

#### LOW

The LOW value (0) exported by wiringPi.

## Authors

Caolan McMahon, Jörg F. Wittenberger, Jan Hoekstra (Altlock)
