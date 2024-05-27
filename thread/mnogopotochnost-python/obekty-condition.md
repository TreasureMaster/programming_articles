# Объекты Condition

В этой главе мы изучим другой способ синхронизации потоков: использование объекта **Condition**. Поскольку переменная условия всегда связана с какой-либо блокировкой, ее можно привязать к общему ресурсу. Блокировку можно передать или она будет создана по умолчанию. Передача одного полезна, когда несколько переменных условия должны использовать одну и ту же блокировку. Блокировка является частью объекта условия: нам не нужно отслеживать ее отдельно. Таким образом, объект условия позволяет потокам ждать обновления ресурса.

В следующем примере потоки-потребители ждут, пока **Condition** не будет установлено, прежде чем продолжить. Поток-производитель отвечает за установку условия и уведомление других потоков о том, что они могут продолжить.

```python
import threading
import time
import logging

logging.basicConfig(level=logging.DEBUG,
                    format='(%(threadName)-9s) %(message)s',)

def consumer(cv):
    logging.debug('Consumer thread started ...')
    with cv:
    	logging.debug('Consumer waiting ...')
        cv.wait()
        logging.debug('Consumer consumed the resource')

def producer(cv):
    logging.debug('Producer thread started ...')
    with cv:
        logging.debug('Making resource available')
        logging.debug('Notifying to all consumers')
        cv.notifyAll()

if __name__ == '__main__':
    condition = threading.Condition()
    cs1 = threading.Thread(name='consumer1', target=consumer, args=(condition,))
    cs2 = threading.Thread(name='consumer2', target=consumer, args=(condition,))
    pd = threading.Thread(name='producer', target=producer, args=(condition,))

    cs1.start()
    time.sleep(2)
    cs2.start()
    time.sleep(2)
    pd.start()
```

Вывод:

```bash
(consumer1) Consumer thread started ...
(consumer1) Consumer waiting ...
(consumer2) Consumer thread started ...
(consumer2) Consumer waiting ...
(producer ) Producer thread started ...
(producer ) Making resource available
(producer ) Notifying to all consumers
(consumer1) Consumer consumed the resource
(consumer2) Consumer consumed the resource
```

Обратите внимание, что мы вообще не использовали методы **acquire()** и **release()**, поскольку мы использовали функцию диспетчера контекста объекта блокировки (использование блокировок в операторе **with** - диспетчере контекста). Вместо этого наши потоки использовали **with** для получения блокировки, связанной с **Condition**.

Метод **wait()** снимает блокировку, а затем блокируется, пока другой поток не разбудит ее, вызвав **notify()** или **notify\_all()**.

Обратите внимание, что методы **notify()** и **notify\_all()** не снимают блокировку; это означает, что пробужденный поток или потоки не вернутся из своего вызова **wait()** немедленно, а только тогда, когда поток, вызвавший **notify()** или **notify\_all()**, наконец откажется от владения блокировкой.

Типичный стиль программирования с использованием условных переменных использует блокировку для синхронизации доступа к некоторому общему состоянию; потоки, которые заинтересованы в конкретном изменении состояния, повторно вызывают **wait()**, пока не увидят желаемое состояние, в то время как потоки, которые изменяют состояние, вызывают **notify()** или **notify\_all()**, когда они изменяют состояние таким образом, чтобы оно могло дать желаемое состояние для одного из ждущих потоков.

Например, следующий код представляет собой общую ситуацию производитель-потребитель с неограниченной емкостью буфера:

```python
# Consume one item
with cv:
    while not an_item_is_available():
        cv.wait()
    get_an_available_item()

# Produce one item
with cv:
    make_an_item_available()
    cv.notify()
```
