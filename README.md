# Rabbitmq

Queue code:

```#!/usr/bin/env python
import pika
import sys
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()
channel.queue_declare(queue='task_queue', durable=True)

```

***also run the commands:
 rabbitmqctl set_permissions -p "/" "user" ".*"  ".*" ".*"
 rabbitmqctl add_user user user 

Sender code:
```
#!/usr/bin/env python
import pika
import sys
credits = pika.PlainCredentials('user','user')
params = pika.ConnectionParameters('35.242.136.253',5672,'/',credits)
connection = pika.BlockingConnection(parameters=params)
channel = connection.channel()
channel.queue_declare(queue='task_queue', durable=True)
message = ' '.join(sys.argv[1:]) or "Hello World!"
channel.basic_publish(exchange='',
                      routing_key='task_queue',
                      body=message,
                      properties=pika.BasicProperties(
                         delivery_mode = 2, # make message persistent
                      ))
print(" [x] Sent %r" % message)
connection.close()
```
Worker Code:
```
#!/usr/bin/env python
import pika
import time
credits = pika.PlainCredentials('user','user')
params = pika.ConnectionParameters('35.242.136.253',5672,'/',credits)
connection = pika.BlockingConnection(parameters=params)
channel = connection.channel()
channel.queue_declare(queue='task_queue', durable=True)
print(' [*] Waiting for messages. To exit press CTRL+C')
def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    time.sleep(body.count(b'.'))
    print(" [x] Done")
    ch.basic_ack(delivery_tag = method.delivery_tag)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,
                      queue='task_queue')
channel.start_consuming()
```
