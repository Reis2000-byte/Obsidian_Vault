- Serial communication
- 2 wires: **TX** and **RX**
- No shared clock
- Data sent **bit-by-bit**
- Both sides must agree on:
    - Baud rate
    - Data bits
    - Parity
    - Stop bits
“Asynchronous” means:
> There is no clock line. Timing is inferred from the data stream itself.

- Start bit: 
	- line is held high in idle to help protect against damaged lines
- Stop bit:
	- line goes high. Can stay high or go low to high
- Data bit:
	- 5 to 9 bits (usually 7 or 8)
	- Data is sent with the LSB first
	- Example: (0x52) = 1010011
		- LSB order = 1100101
- Parity bit (optional):
	- even parity: number of 1's must be even
	- odd parity: number of 1's must be odd
	- used to check number of bits as a security check but can only check once

Architecture:
	Application
	   ↓
	UART API Layer
	   ↓
	Driver (buffers + ISR logic)
	   ↓
	Hardware Registers

Code:

Data Structure:
```C
#define UART_BUFFER_SIZE 128

typedef struct {
    uint8_t buffer[UART_BUFFER_SIZE];
    volatile uint16_t head;
    volatile uint16_t tail;
} ring_buffer_t;

```
- Use volatile because it is modified in the ISR and main
- Prevents bugs
UART Config:
```C
typedef struct {
    uint32_t baudrate;
    uint8_t parity;     // 0 = none, 1 = even, 2 = odd
    uint8_t stop_bits;  // 1 or 2
} uart_config_t;
```

UART Handle:
```C
typedef struct {
    USART_Type *regs;          // Hardware register base
    ring_buffer_t tx_buffer;
    ring_buffer_t rx_buffer;
} uart_handle_t;
```

UART Init:
```C
void uart_init(uart_handle_t *uart, uart_config_t *config)
{
    // 1. Enable peripheral clock
    // 2. Configure GPIO pins (TX/RX alternate function)
    // 3. Set baud rate
    // 4. Configure parity, stop bits
    // 5. Enable TX, RX
    // 6. Enable RX interrupt
}
```

Ring Buffer:
```C
static bool rb_push(ring_buffer_t *rb, uint8_t data)
{
    uint16_t next = (rb->head + 1) % UART_BUFFER_SIZE;

    if (next == rb->tail) {
        return false; // buffer full
    }

    rb->buffer[rb->head] = data;
    rb->head = next;
    return true;
}

static bool rb_pop(ring_buffer_t *rb, uint8_t *data)
{
    if (rb->head == rb->tail) {
        return false; // empty
    }

    *data = rb->buffer[rb->tail];
    rb->tail = (rb->tail + 1) % UART_BUFFER_SIZE;
    return true;
}
```

Transmit Implementation:
```C
bool uart_write(uart_handle_t *uart, uint8_t data)
{
    bool success = rb_push(&uart->tx_buffer, data);

    // Enable TX empty interrupt
    uart->regs->INTENSET = USART_INTENSET_DRE;

    return success;
}
```

RX Implementation:
```C
void USARTx_Handler(void)
{
    // RX ready
    if (uart->regs->INTFLAG & USART_INTFLAG_RXC) {
        uint8_t data = uart->regs->DATA;
        rb_push(&uart->rx_buffer, data);
    }

    // TX ready (Data Register Empty)
    if (uart->regs->INTFLAG & USART_INTFLAG_DRE) {
        uint8_t data;

        if (rb_pop(&uart->tx_buffer, &data)) {
            uart->regs->DATA = data;
        } else {
            // Nothing left — disable TX interrupt
            uart->regs->INTENCLR = USART_INTENCLR_DRE;
        }
    }
}

```

Receive API:
```C
bool uart_read(uart_handle_t *uart, uint8_t *data)
{
    return rb_pop(&uart->rx_buffer, data);
}

```