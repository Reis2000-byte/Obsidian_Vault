# 🔶 UNION in C

A **union** shares the same memory location for all members.

Only **one member is valid at a time**.

---

### ✅ Example 1 — Basic Union

```
#include <stdio.h>  

union Data 
{     
	int i;     
	float f;     
	char c; 
};  

int main() 
{     
	union Data data;      
	data.i = 10;     
	printf("i = %d\n", data.i);      
	data.f = 3.14;  // overwrites previous value     
	printf("f = %f\n", data.f); 
}
```

### Memory Layout (Union)

If largest member = 4 bytes

Union size = **4 bytes total**

`| shared memory (4 bytes) |`

All members overlap.