import threading
import time
import random

BUFFER = []
BUFFER_SIZE = 5

mutex = threading.Semaphore(1)      # For mutual exclusion
empty = threading.Semaphore(BUFFER_SIZE)  # Counts empty slots
full = threading.Semaphore(0)       # Counts filled slots

def producer():
    item = 1
    while item <= 10:
        empty.acquire()
        mutex.acquire()
        BUFFER.append(item)
        print(f"Producer produced: Item-{item} | Buffer: {BUFFER}")
        mutex.release()
        full.release()
        item += 1
        time.sleep(random.uniform(0.5, 1.5))

def consumer():
    count = 1
    while count <= 10:
        full.acquire()
        mutex.acquire()
        value = BUFFER.pop(0)
        print(f"Consumer consumed: Item-{value} | Buffer: {BUFFER}")
        mutex.release()
        empty.release()
        count += 1
        time.sleep(random.uniform(0.5, 1.5))

producer_thread = threading.Thread(target=producer)
consumer_thread = threading.Thread(target=consumer)
producer_thread.start()
consumer_thread.start()
producer_thread.join()
consumer_thread.join()
print("\nAll items produced and consumed successfully using Semaphores!")
