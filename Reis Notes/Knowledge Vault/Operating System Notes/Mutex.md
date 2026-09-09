- The job of a mutex is to protect ownership of a resource
- Key Properties:
	- Only one owner at a time
	- Owner is only one who can unlock it
	- Supports priority inheritance
	- used for critical sections, shared memory, hardware registers


Example (application):

```C
#include <pthread.h>

int main (int argc, char **argv)
{
	pthread_mutex_init(&my_mutex);
}

void critical_function()
{
	pthread_mutex_lock(&my_mutex);
	...
	pthread_mutex_unlock(&my_mutex);
}
```

- Essentially a mutex is just a global integer value that is a 0 or 1
- The locking and unlocking are considered atomic operations
- Atomic operations are operations that cannot be interrupted

Example (assembly):
```assembly
lock_acquire:
	li $t0, 0 //load lock value in temp register
	li $t1, 1 //load unlock value in temp register
	cas $t0, $t1, lock //atomic compare and swap the two values
	beq $t0, $t1, lock_acquire //load value in temp register
```