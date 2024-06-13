RabbitMQ is a messaging broker that implements the Advanced Message Queuing Protocol (AMQP). It is used to facilitate communication between different micro-services in a distributed system. RabbitMQ provides a reliable, flexible, and scalable messaging platform that enables developers to build distributed systems that can handle large amounts of traffic. It is widely used in enterprise applications for tasks such as message queuing, load balancing, and task [scheduling.In](http://scheduling.in/) general, it may be needed to declare the exchanges and queues before starting the consumer and producer, especially if the micro-service is functioning as both a message publisher and listener.

I

There are a couple of things to keep in mind in terms of definitions:

1. A `Connection` represents a real TCP connection to the message broker, whereas a `Channel` is a virtual connection (AMQP connection) inside it. This way you can use as many (virtual) connections as you want inside your application without overloading the broker with TCP connections. When working with this, we need only one connection. Then, if we need, can create more channels, but for this project, when I’m just using 3 queues, one channel is enough.
2. You can use one `Channel` for everything. However, if you have multiple threads, it's suggested to use a different `Channel` for each thread.
    
    [Channel thread-safety in Java Client API Guide](http://www.rabbitmq.com/api-guide.html#channel-threads):
    
    > Channel instances are safe for use by multiple threads. Requests into a Channel are serialized, with only one thread being able to run a command on the Channel at a time. Even so, applications should prefer using a Channel per thread instead of sharing the same Channel across multiple threads.
    
    There is no direct relation between `Channel` and `Queue`. A `Channel` is used to send AMQP commands to the broker. This can be the creation of a queue or similar, but these concepts are not tied together.
    
3. Each `Consumer` runs in its own thread allocated from the consumer thread pool. If multiple Consumers are subscribed to the same Queue, the broker uses round-robin to distribute the messages between them equally. See [Tutorial two: "Work Queues"](http://www.rabbitmq.com/tutorials/tutorial-two-java.html).
    
    It is also possible to attach the same `Consumer` to multiple Queues. You can understand Consumers as callbacks. These are called every time a message arrives on a Queue the Consumer is bound to. For the case of the Java Client, each Consumers has a method `handleDelivery(...)`, which represents the callback method. What you typically do is, subclass `DefaultConsumer` and override `handleDelivery(...)`. Note: If you attach the same Consumer instance to multiple queues, this method will be called by different threads. So take care of synchronization if necessary.
    

  

Here is some `JavaScript` code that demonstrates making a connection, channel, and queues:

```TypeScript
import amqp, { Message, MessageProperties } from "amqplib";
import { CreateQuizRequestDto } from "../dtos/createQuizRequestDto";
import { getQuizRepository } from "../repositories/quizRepository";
import { Repository } from "typeorm";
import { Quiz } from "../entities/quiz";
import { Question } from "../entities/question";
import dotenv from "dotenv";
import { Option } from "../entities/option";
import { Answer } from "../entities/answer";

dotenv.config();

export class RabbitmqService {
  channel!: amqp.Channel;
  quizRepository!: Repository<Quiz>;

  setup = async () => {
    const connection = await amqp.connect("amqp://localhost:5672");
    this.channel = await connection.createChannel();
    this.quizRepository = getQuizRepository();
  };

  publishMessage = async (message: CreateQuizRequestDto): Promise<string> => {
    if (!this.channel) {
      await this.setup();
    }
    await this.channel.assertExchange(
      process.env.rabbitmq_gpt_exchange!,
      "direct",
      {
        durable: true,
      }
    );

    const outputMessage: string = JSON.stringify(message);
    const correlationId: string = message.id;
    const messageProperties: MessageProperties = {
      contentType: "text/plain",
      contentEncoding: "utf-8",
      headers: {
        customHeader: "value",
      },
      deliveryMode: undefined,
      priority: undefined,
      replyTo: undefined,
      expiration: undefined,
      messageId: undefined,
      timestamp: undefined,
      type: undefined,
      userId: undefined,
      appId: undefined,
      clusterId: undefined,
      correlationId: correlationId,
    };

    const messageBuffer = Buffer.from(outputMessage);

    this.channel.publish(
      process.env.rabbitmq_gpt_exchange!,
      process.env.gpt_request_rabbitmq_routing_key!,
      messageBuffer,
      messageProperties
    );

    return correlationId;
  };

  consumeGptRequestMessageFromMq = async (): Promise<void> => {
    if (!this.channel) {
      await this.setup();
    }

    await this.channel.assertExchange(
      process.env.rabbitmq_gpt_exchange!,
      "direct",
      {
        durable: true,
      }
    );

    const queueNames = [
      process.env.to_gpt_rabbitmq_request_queue,
      process.env.to_gateway_gpt_rabbitmq_response_queue,
    ];

    for (const queueName of queueNames) {
      await this.channel.assertQueue(queueName!, { durable: true });
    }

    this.channel.consume(
      process.env.to_gateway_gpt_rabbitmq_response_queue!,
      (data) => {
        if (data) {
          this.channel.ack(data);
          this.save(data);
        }
      }
    );
  };

  save = async (incomingMessage: any): Promise<void> => {
    try {
      const quizRepository = getQuizRepository();
      const message: Message = incomingMessage as Message;
      const messageString: string = message.content.toString("utf-8");

      const dataObject = JSON.parse(messageString);
      const quiz = new Quiz();
      quiz.quizId = dataObject.id;
      quiz.title = dataObject.title;
      quiz.questions = [];

      for (let q of dataObject.questions) {
        const question = new Question();
        question.options = [];
        question.answers = [];

        for (let op of q.options) {
          const option = new Option();
          option.content = op;
          question.options.push(option);
        }

        for (let a of q.answers) {
          const answer = new Answer();
          answer.content = a;
          question.answers.push(answer);
        }

        question.text = q.text;
        console.log(question);
        quiz.questions.push(question);
      }
      await quizRepository.save(quiz);
    } catch (error) {
      console.log(error);
    }
  };
}
```

In general, it may be needed to declare the exchanges and queues before starting the consumer and producer, especially if the micro-service is functioning as both a message publisher and listener.