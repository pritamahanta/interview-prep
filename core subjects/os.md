Here is a structured, clean, and straightforward `README.md` file you can use for your repository. The content has been categorized by topic, the C++ code blocks have been properly formatted, and direct comparisons have been put into tables for better readability.

---

# Operating Systems: Core Interview Concepts

This repository contains high-yield interview concepts and implementations for Operating Systems (OS). It serves as a quick reference guide for core topics like memory management, process synchronization, and cache data structures.

## Table of Contents

* [System Architecture](https://www.google.com/search?q=%23system-architecture)
* [Process Management](https://www.google.com/search?q=%23process-management)
* [Memory Management](https://www.google.com/search?q=%23memory-management)
* [Concurrency & Synchronization](https://www.google.com/search?q=%23concurrency--synchronization)
* [Cache Implementations (C++)](https://www.google.com/search?q=%23cache-implementations-c)

---

## System Architecture

### Kernel vs. Shell

* **Kernel:** The core component that manages system resources (CPU, memory, hardware devices). It acts as the bridge between hardware and software, providing low-level services like memory allocation and process management.
* **Shell:** The user interface (like Bash or PowerShell) that interprets user commands and communicates with the kernel to execute them.

---

## Process Management

### Process vs. Thread

A **process** is an instance of a running program, while a **thread** is a sequential flow of execution within a process (often called a lightweight process).

| Feature | Process | Thread |
| --- | --- | --- |
| **Isolation** | Isolated with separate memory spaces. | Share the same memory space within the process. |
| **Resource Overhead** | High (requires separate memory/resources). | Low (shares existing process resources). |
| **Creation Time** | Slower and more time-consuming. | Faster to create and switch. |
| **Communication** | Requires Inter-Process Communication (IPC). | Simpler, using shared memory. |

### Context Switch

A context switch is the process of saving the current state of a running process (registers, program counter) and loading the saved state of another process. It allows the OS to efficiently manage multiple processes, giving the illusion of concurrent execution. It incurs overhead in time and system resources and occurs when a process yields the CPU, an interrupt happens, or the scheduler preempts a process.

### `fork()`

Calling `fork()` creates a new process by duplicating the existing parent process.

* **Memory Duplication:** Uses "copy-on-write" to duplicate the parent's memory space.
* **File Descriptors:** Inherits open file descriptors from the parent.
* **Return Values:** Returns the child's PID to the parent process, and `0` to the child process, allowing them to execute different code paths.

### Scheduling Algorithms

Scheduling algorithms manage the allocation of CPU time to processes.

* **FCFS (First-Come-First-Serve):** Executed in the order they arrive.
* **SJN (Shortest Job Next):** Prioritizes the shortest job first.
* **Round Robin:** Each process gets a fixed time slice cyclically.
* **Priority Scheduling:** Higher priority processes execute first.
* **Multi-Level Queue:** Processes are divided into distinct priority queues.

---

## Memory Management

### Virtual Memory & Memory Management Unit (MMU)

**Virtual Memory** uses physical RAM and disk space to provide the illusion of a larger, contiguous memory block.

* **Address Space:** Each process gets a virtual address space larger than actual RAM.
* **MMU:** Translates virtual addresses to physical addresses using **Page Tables**. It enforces memory protection and manages TLB (Translation Lookaside Buffer) for faster lookups.

### Paging vs. Segmentation

| Feature | Paging | Segmentation |
| --- | --- | --- |
| **Division** | Divides memory into fixed-size pages. | Divides memory into variable-sized segments. |
| **Fragmentation** | Causes internal fragmentation. | Causes external fragmentation. |
| **Mapping** | Uses a Page Table. | Uses a Segment Table. |

### Page Faults & Thrashing

* **Page Fault:** Occurs when a program attempts to access memory not currently in physical RAM. The OS hardware generates a trap, checks the page table, fetches the page from the disk into RAM, updates the table, and retries the instruction.
* **Thrashing:** Occurs when the system spends more time swapping data between RAM and disk than executing useful tasks. It causes high disk activity and low CPU utilization.
* **Belady’s Anomaly:** A situation (mostly in FIFO algorithms) where adding more physical RAM unexpectedly *increases* the number of page faults. Algorithms like LRU prevent this.

### Cache Memory

A small, high-speed volatile storage between RAM and the CPU. It accelerates data retrieval by storing frequently accessed instructions. It operates in a hierarchy (L1, L2, L3) and relies on cache coherency protocols and replacement policies (like LRU) to maintain system stability.

---

## Concurrency & Synchronization

### Deadlock & Banker's Algorithm

A **deadlock** occurs when two or more processes are waiting for resources held by each other. It requires four conditions: Mutual Exclusion, Hold and Wait, No Preemption, and Circular Wait.

* **Prevention Strategies:** Resource Ordering, Timeouts, Resource Preemption, and Process Termination.
* **Banker's Algorithm:** A dynamic resource allocation algorithm that tracks available resources and denies requests that would lead an OS into an unsafe, deadlocked state.

### Semaphore

A synchronization mechanism (integer variable) used to manage concurrent access to critical sections.

* **Mutex (Binary Semaphore):** Initialized to 1, ensuring only one process enters a critical section.
* **Counting Semaphore:** Allows multiple processes up to a specified limit.

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <semaphore.h>

sem_t mutex;

void criticalSection(int id) {
    sem_wait(&mutex); 
    std::cout << "Thread " << id << " entered." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Thread " << id << " exited." << std::endl;
    sem_post(&mutex); 
}

int main() {
    sem_init(&mutex, 0, 1);
    std::thread t1(criticalSection, 1);
    std::thread t2(criticalSection, 2);
    t1.join();
    t2.join();
    sem_destroy(&mutex);
    return 0;
}

```

---

## Cache Implementations (C++)

### 1. LRU Cache (Least Recently Used)

Uses `std::unordered_map` for $O(1)$ lookups and `std::list` (doubly linked list) to maintain the access order.

```cpp
#include <bits/stdc++.h>
#include <unordered_map>
#include <list>

class LRUCache {
private:
    int capacity;
    std::unordered_map<int, std::pair<int, std::list<int>::iterator>> cache;
    std::list<int> lru;

public:
    LRUCache(int capacity) {
        this->capacity = capacity;
    }

    int get(int key) {
        if (cache.find(key) != cache.end()) {
            lru.erase(cache[key].second);
            lru.push_front(key);
            cache[key].second = lru.begin();
            return cache[key].first;
        }
        return -1;
    }

    void put(int key, int value) {
        if (cache.size() >= capacity) {
            int lruKey = lru.back();
            cache.erase(lruKey);
            lru.pop_back();
        }
        lru.push_front(key);
        cache[key] = {value, lru.begin()};
    }
};

```

### 2. LFU Cache (Least Frequently Used)

Tracks frequency using multiple hash maps to ensure $O(1)$ eviction of the least frequently used item.

```cpp
#include <bits/stdc++.h>
#include <unordered_map>
#include <map>
#include <list>

class LFUCache {
private:
    int capacity;
    std::unordered_map<int, std::pair<int, int>> cache; 
    std::unordered_map<int, std::list<int>> frequency_map; 
    std::map<int, int> frequency_counter; 
    int min_frequency;

public:
    LFUCache(int capacity) {
        this->capacity = capacity;
        min_frequency = 0;
    }

    int get(int key) {
        if (cache.find(key) != cache.end()) {
            int value = cache[key].first;
            int old_frequency = cache[key].second;
            
            frequency_map[old_frequency].remove(key);
            frequency_counter[key]++;
            int new_frequency = frequency_counter[key];
            frequency_map[new_frequency].push_back(key);
            
            if (frequency_map[min_frequency].empty()) {
                min_frequency++;
            }
            return value;
        }
        return -1; 
    }

    void put(int key, int value) {
        if (capacity == 0) return;
        
        if (cache.find(key) != cache.end()) {
            cache[key].first = value;
            get(key); 
        } else {
            if (cache.size() >= capacity) {
                int victim = frequency_map[min_frequency].front();
                frequency_map[min_frequency].pop_front();
                cache.erase(victim);
                frequency_counter.erase(victim);
            }
            cache[key] = {value, 1};
            frequency_map[1].push_back(key);
            frequency_counter[key] = 1;
            min_frequency = 1;
        }
    }
};

```