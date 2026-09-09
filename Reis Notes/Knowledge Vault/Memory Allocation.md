- **Static Memory**: 
	- Memory allocated at run time
	- The memory allocated is fixed and cannot be increased or decreased
	- Example:
```C
	int main()
	{
		int arr[5] = {1,2,3,4,5};
	}
```
- **Problems w/ Static Memory**:
	- If creating an object you have to fix the size at declaration
	- If the values stored in the object at runtime is less than the specified size than you have wasted memory!!!!
	- If you make an object bigger at runtime this may cause the program to crash.

- **Dynamic Memory**:
	- The process of allocating memory at the time of execution.
	- Heap is the segment of memory where dynamic allocation takes place
	- Unlike the stack where memory is allocated or deallocated in a defined order, heap is an area of memory where memory is allocated or deallocated without any order.

	- Built in Functions:
		- malloc()
		- calloc()
		- realloc()
		- free()