# 🔷 STRUCT in C

A **struct** groups related variables together under one name.

### ✅ Example 1 — Basic Struct

```
#include <stdio.h> 
 
struct SensorData 
{     
	float temperature;     
	float humidity;     
	int status; 
};  

int main() 
{     
	struct SensorData sensor1;      
	sensor1.temperature = 25.3;     
	sensor1.humidity = 60.5;     
	sensor1.status = 1;      
	printf("Temp: %.1f\n", sensor1.temperature); 
}
```

### Memory Layout (Struct)

If:

- float = 4 bytes
    
- int = 4 bytes
    

Then struct size ≈ **12 bytes** (possibly more due to padding).

Each member gets its **own memory space**.

`| temperature | humidity | status | |     4B      |    4B    |   4B   |`

---

### ✅ Example 2 — Struct in Embedded (Hardware Register Style)

```
typedef struct 
{     
	volatile uint32_t CTRL;     
	volatile uint32_t STATUS;     
	volatile uint32_t DATA; 
} ADC_TypeDef;  

#define ADC0 ((ADC_TypeDef *)0x40003000) 
 
void start_adc() 
{     
	ADC0->CTRL |= (1 << 0); 
}
```

👉 This is exactly how MCU peripheral registers are defined in CMSIS headers.