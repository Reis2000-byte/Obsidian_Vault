- Semaphores signal work
- Thread wakes from a semaphore -> then locks a mutex

**Example: Producer/Consumer**
- Semaphore increments when new data arrives
- Consumer wakes
- Consumer locks mutex to safely read shared buffer

```C
// Producer
mutex_lock(&buffer_mutex);
write_to_buffer();
mutex_unlock(&buffer_mutex);
k_sem_give(&data_sem);

// Consumer
k_sem_take(&data_sem, K_FOREVER);
mutex_lock(&buffer_mutex);
read_from_buffer();
mutex_unlock(&buffer_mutex);
```

- **Semaphore = notification**
- **Mutex = protect the data**

**Typical Flow:**
- Take semaphore -> lock mutex -> use resource -> unlock mutex -> give semaphore

**Important Notes!**
- Avoid Locking a mutex inside a semaphore ISR give
	- Priority inheritance doesn't work with semaphores, so nesting them incorrectly can cause deadlocks.
- Avoid using a semaphore as a mutex
	- A semaphore cannot prevent priority inversion. A mutex can because it is atomic


**Overview:**

|        Feature        | Mutex | Binary Sem | Counting Sem |
| :-------------------: | :---: | :--------: | :----------: |
|  Protect Shared Data  |   ✅   |     ❌      |      ❌       |
| Signal threads/events |   ❌   |     ✅      |      ⚠️      |
|    Count Resources    |   ❌   |     ❌      |      ✅       |
| Priority Inheritance  |   ✅   |     ❌      |      ❌       |
|    Ownership Rules    |  Yes  |     No     |      No      |
