import threading
import time
import random

BUFFER = []
BUFFER_SIZE = 5
mutex = threading.Lock()
empty = threading.Semaphore(BUFFER_SIZE)
full = threading.Semaphore(0)

def producer():
    item_id = 1
    while item_id <= 10:
        empty.acquire()
        mutex.acquire()
        item = f"Item-{item_id}"
        BUFFER.append(item)
        print(f"Producer produced: {item} | Buffer: {BUFFER}")
        mutex.release()
        full.release()
        item_id += 1
        time.sleep(random.uniform(0.5, 1.5))

def consumer():
    for _ in range(10):
        full.acquire()
        mutex.acquire()
        item = BUFFER.pop(0)
        print(f"Consumer consumed: {item} | Buffer: {BUFFER}")
        mutex.release()
        empty.release()
        time.sleep(random.uniform(0.5, 1.5))

producer_thread = threading.Thread(target=producer)
consumer_thread = threading.Thread(target=consumer)
producer_thread.start()
consumer_thread.start()
producer_thread.join()
consumer_thread.join()
print("\nAll items produced and consumed successfully!")
