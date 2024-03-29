# semaphore

- counting_semaphore implementation. This is header-only, no external dependency C++11 library.
- According to C++20 standard. [cppreference - std::counting_semaphore](https://en.cppreference.com/w/cpp/thread/counting_semaphore)

## example

```cpp
#include <chrono>
#include <iostream>
#include <thread>

#include "semaphore.h"

// global binary semaphore instances
// object counts are set to zero
// objects are in non-signaled state
cyan::binary_semaphore smphSignalMainToThread{0}, smphSignalThreadToMain{0};

void ThreadProc() {
  // wait for a signal from the main proc
  // by attempting to decrement the semaphore
  smphSignalMainToThread.acquire();

  // this call blocks until the semaphore's count
  // is increased from the main proc

  std::cout << "[thread] Got the signal\n";  // response message

  // wait for 3 seconds to imitate some work
  // being done by the thread
  std::this_thread::sleep_for(std::chrono::seconds(3));

  std::cout << "[thread] Send the signal\n";  // message

  // signal the main proc back
  smphSignalThreadToMain.release();
}

int main() {
  // create some worker thread
  std::thread thrWorker(ThreadProc);

  std::cout << "[main] Send the signal\n";  // message

  // signal the worker thread to start working
  // by increasing the semaphore's count
  smphSignalMainToThread.release();

  // wait until the worker thread is done doing the work
  // by attempting to decrement the semaphore's count
  smphSignalThreadToMain.acquire();

  std::cout << "[main] Got the signal\n";  // response message
  thrWorker.join();
}
```

## output
```
[main] Send the signal
[thread] Got the signal
[thread] Send the signal
[main] Got the signal
```
