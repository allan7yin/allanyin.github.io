This is a quick note on **Apache Kafka**. This was used at **Bank of America Merrill Lynch**. So, whats the difference between **Kafka** and **RabbitMQ**? There are 2 things we need to first clear up:

## In Memory Message Broker

This is an example of what RabbitMQ is. Messages are stored in memory, so if this queue does fail, these messages are lost.

![[Screenshot_2024-03-25_at_2.26.34_PM.png]]

A key point is, a message in a queue can only be consumed by one consumer. So, we make some request and push this as a message, only one consumer can pick it up and process it. This is different from Log based message brokers.

### Log Based Message Brokers

![[Screenshot_2024-03-25_at_2.28.22_PM.png]]

In Kafka, unlike RabbitMQ queues, a single message in a topic **can be consumed by multiple consumers**, but with some key points to consider:

- **Consumer Groups:** This is where message delivery order comes into play. Consumers subscribing to the same topic with the same consumer group ID will act as a single logical consumer.
    - Kafka ensures **only one consumer in the group** will process a message from a specific partition.
    - Messages within a partition are delivered in the order they are produced (similar to RabbitMQ queues).
- **Multiple Consumer Groups:** If you have different consumer groups subscribed to the same topic, **each group will independently consume messages from all partitions**. This means:
    - The order of messages across different consumer groups is **not guaranteed**.
    - Each group processes messages from assigned partitions in order, but these partitions might be processed at different speeds by different consumers.

**In summary:**

- Kafka allows for parallel processing of messages from a single topic by multiple consumers, but order is dependent on consumer groups and partitions.
- Within a consumer group, only one consumer processes a message from a partition, ensuring in-order delivery for that partition.
- Across different consumer groups, order is not guaranteed by default.

  

This is different from the round-robin approach of RabbitMQ:

- **Focus on the queue:** The round-robin logic applies specifically within the context of a single queue.
- **Message arrives:** When a new message enters the queue, RabbitMQ doesn't consider the entire pool of consumers.
- **Tracking last recipient:** It only looks at the consumers connected to **that specific queue**. It remembers which consumer received the **last message** from this queue.
- **Delivery to next consumer:** It delivers the new message to the **next consumer** in the queue **relative to the last recipient**. This means:
    - If Consumer A was the last recipient, the message goes to Consumer B (if available).
    - If there's no Consumer B, it wraps around and sends it to Consumer A (assuming it's still active).

**Essentially, RabbitMQ maintains a "pointer" for each queue, indicating the last consumer that received a message.** It uses this pointer to determine the next recipient in the round-robin fashion.

**Additional points:**

- This approach ensures all active consumers get a chance to process messages from the queue.
- The order in which messages were **published** to the queue is not necessarily preserved during delivery due to the round-robin mechanism.

**pen_spark**