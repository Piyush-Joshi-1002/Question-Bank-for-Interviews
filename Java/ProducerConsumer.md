<h1>multithreaded Producer-Consumer using wait/notify  java</h1>

The key challenges are:

Buffer Overflow: The producer must not try to add data to a full buffer.
Buffer Underflow: The consumer must not try to remove data from an empty buffer.
Synchronization: Both processes need to access the buffer in a synchronized manner to prevent data corruption.

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;

public class ProducerConsumerWaitNotify {

    public static void main(String[] args) {
        // Create a shared buffer with a fixed capacity
        Buffer buffer = new Buffer(5);

        // Create producer and consumer threads
        Thread producerThread = new Thread(new Producer(buffer), "Producer");
        Thread consumerThread = new Thread(new Consumer(buffer), "Consumer");

        // Start the threads
        producerThread.start();
        consumerThread.start();
    }
}

// The shared buffer class
class Buffer {
    private Queue<Integer> queue;
    private int capacity;
    private final Object lock = new Object(); // Object for synchronization

    public Buffer(int capacity) {
        this.queue = new LinkedList<>();
        this.capacity = capacity;
    }

    // Method for the producer to add items to the buffer
    public void produce(int item) throws InterruptedException {
        synchronized (lock) { // Acquire the lock
            // If the buffer is full, producer waits
            while (queue.size() == capacity) {
                System.out.println(Thread.currentThread().getName() + ": Buffer is full. Waiting...");
                lock.wait(); // Release the lock and wait
            }

            // Add the item to the buffer
            queue.add(item);
            System.out.println(Thread.currentThread().getName() + ": Produced: " + item + ". Buffer size: " + queue.size());

            // Notify consumer that there's an item available
            lock.notify(); // Wakes up a waiting consumer thread (if any)
        }
    }

    // Method for the consumer to remove items from the buffer
    public int consume() throws InterruptedException {
        synchronized (lock) { // Acquire the lock
            // If the buffer is empty, consumer waits
            while (queue.isEmpty()) {
                System.out.println(Thread.currentThread().getName() + ": Buffer is empty. Waiting...");
                lock.wait(); // Release the lock and wait
            }

            // Remove the item from the buffer
            int item = queue.poll();
            System.out.println(Thread.currentThread().getName() + ": Consumed: " + item + ". Buffer size: " + queue.size());

            // Notify producer that there's space available
            lock.notify(); // Wakes up a waiting producer thread (if any)
            return item;
        }
    }
}

// Producer thread
class Producer implements Runnable {
    private Buffer buffer;
    private Random random = new Random();

    public Producer(Buffer buffer) {
        this.buffer = buffer;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 10; i++) { // Produce 10 items
                int item = random.nextInt(100); // Produce a random item
                buffer.produce(item);
                Thread.sleep(random.nextInt(500)); // Simulate time taken to produce
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Producer interrupted.");
        }
    }
}

// Consumer thread
class Consumer implements Runnable {
    private Buffer buffer;
    private Random random = new Random();

    public Consumer(Buffer buffer) {
        this.buffer = buffer;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 10; i++) { // Consume 10 items
                buffer.consume();
                Thread.sleep(random.nextInt(700)); // Simulate time taken to consume
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Consumer interrupted.");
        }
    }
}
```
