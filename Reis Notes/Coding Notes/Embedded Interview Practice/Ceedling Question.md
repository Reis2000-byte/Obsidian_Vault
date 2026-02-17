
Code:
```C
// temp_sensor.h
int16_t temp_read_raw(void);

// fan_controller.c
#include "temp_sensor.h"

#define TEMP_THRESHOLD 75

bool fan_should_turn_on(void)
{
    int16_t temp = temp_read_raw();
    return (temp > TEMP_THRESHOLD);
}
```

## ❓ Interview Question

1. How would you unit test `fan_should_turn_on()` using Ceedling?
	    add the following function to my mock
2. How would you mock `temp_read_raw()`?
	    Here is the following function that would get generated from Cmock:
```C
	#include "mock_temp_sensor.h"
	int16_t temp_read_raw_ExpectAndReturn(int16_t return_val);
```
1. Write the test file.
    ```C
	#include "unity.h"
	#include "fan_controller.h"
	#include "mock_temp_sensor.h"
	
	void setUp(void) {
	    // Runs before each test
	}
	
	void tearDown(void) {
	    // Runs after each test
	}
	
	// Test fan turns ON when temp > threshold
	void test_fan_should_turn_on_when_temp_high(void) {
	    temp_read_raw_ExpectAndReturn(80); // mock reading
	    TEST_ASSERT_TRUE(fan_should_turn_on());
	}
	
	// Test fan stays OFF when temp <= threshold
	void test_fan_should_stay_off_when_temp_low(void) {
	    temp_read_raw_ExpectAndReturn(70); // mock reading
	    TEST_ASSERT_FALSE(fan_should_turn_on());
	}
	
	// Test fan stays OFF exactly at threshold
	void test_fan_should_stay_off_at_threshold(void) {
	    temp_read_raw_ExpectAndReturn(75); // edge case
	    TEST_ASSERT_FALSE(fan_should_turn_on());
	}
	```
2. What would happen if `temp_read_raw()` were defined as `static` inside a `.c` file instead?
	    CMock cannot mock static functions because they are not in the header
3. How would you handle this in embedded code where `temp_read_raw()` actually reads from an ADC register?