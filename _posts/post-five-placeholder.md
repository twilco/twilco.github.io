Great, with this base address defined we'll be able to access the UARTs registers.  We're going to start by implementing `uart_put_char`, so let's review the registers we'll be using for that:

TODO: Cut out the unimportant registers.  Maybe make this section about implementing put_char, and going over those registers?  And then in the next section, impl get_char and go over those registers?

<div class="indented-list-item">1. Receive buffer register (RBR)</div>
<p class="indented-list-item-subtext">A read-only register that holds byte(s) received from other UARTs.</p>

<div class="indented-list-item">2. Transmitter holding register (THR)</div>
<p class="indented-list-item-subtext">A write-only register that holds byte(s) waiting to be transmitted.</p>

<div class="indented-list-item">3. Interrupt enable register (IER)</div>
<p class="indented-list-item-subtext">A readable and writable register that allows various types of interrupts to be enabled via the flip of the corresponding bit.</p>

<div class="indented-list-item">4. Interrupt identification reigster (IIR)</div>
<p class="indented-list-item-subtext">A read-only register that allows you to identify which condition caused an interrupt to occur.</p>

<div class="indented-list-item">5. FIFO control register (FCR)</div>
<p class="indented-list-item-subtext">A readable and writable register that configures any FIFO buffers that may be present in the UART.  These buffers supplement the single byte RBR and THR registers, making it possible to send or receive a single byte at a time.</p>

<div class="indented-list-item">6. Line control register (LCR)</div>
<p class="indented-list-item-subtext">A readable and writable register that enables configuration of communication details, such as the number of stop bits and the bit-length size of a single piece of data.</p>

<div class="indented-list-item">7. Modem control register (MCR)</div>
<p class="indented-list-item-subtext">A readable and writable register that can be used to perform handshaking operations with other devices, among other things.</p>

<div class="indented-list-item">8. Line status register (LSR)</div>
<p class="indented-list-item-subtext">A read-only register that displays various communication and error statuses, such as whether or not there is a framing error, if data is available to be read, or if the THR is empty.</p>

<div class="indented-list-item">9. Modem status register (MSR)</div>
<p class="indented-list-item-subtext">A read-only register that displays various communication and error statuses, such as whether or not there is a framing error, if data is available to be read, or if the THR is empty.</p>

<div class="indented-list-item">10. Scratch register (SCR)</div>
<p class="indented-list-item-subtext">A read-only register that displays various communication and error statuses, such as whether or not there is a framing error, if data is available to be read, or if the THR is empty.</p>

<div class="indented-list-item">11. Divisor latch register one (DLL)</div>
<p class="indented-list-item-subtext">A read-only register that displays various communication and error statuses, such as whether or not there is a framing error, if data is available to be read, or if the THR is empty.</p>

<div class="indented-list-item">12. Divisor latch register two (DLM)</div>
<p class="indented-list-item-subtext">A read-only register that displays various communication and error statuses, such as whether or not there is a framing error, if data is available to be read, or if the THR is empty.</p>


