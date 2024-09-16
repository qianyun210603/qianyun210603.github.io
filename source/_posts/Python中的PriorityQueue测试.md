---
title: 【Python知识库】Python中的PriorityQueue测试
date: 2024-03-16T16:15:40+08:00
toc: true
tags: 
categories: 
- Programming
- Python
---

Python官方提供一个`PriorityQueue`，可以按优先级取出Push入队的对象，但这个Queue不稳定（即相同优先级并不保证先进先出）。

测试代码：
```python
from collections import defaultdict
from queue import Empty, Queue, PriorityQueue
from threading import Thread, get_ident
from time import sleep
from typing import Any, Callable, List
from datetime import datetime



EVENT_SLEEP = "eSleep"


class Event:
    """
    Event object consists of a type string which is used
    by event engine for distributing event, and a data
    object which contains the real data.
    """

    def __init__(self, etype: str, data: Any = None, priority: int = 100) -> None:
        """"""
        self.type: str = etype
        self.priority = priority
        self.data: Any = data

    def __lt__(self, other):
        return self.priority > other.priority

    def __le__(self, other):
        return self.priority >= other.priority


# Defines handler function to be used in event engine.
HandlerType: callable = Callable[[Event], None]


class EventEngine:
    """
    Event engine distributes event object based on its type
    to those handlers registered.

    It also generates timer event by every interval seconds,
    which can be used for timing purpose.
    """

    def __init__(self, interval: int = 1, using_priority = False) -> None:
        """
        Timer event is generated every 1 second by default, if
        interval not specified.
        """
        self._interval: int = interval
        self._queue: Queue = PriorityQueue() if using_priority else Queue()
        self._active: bool = False
        self._thread: Thread = Thread(target=self._run)
        self._handlers: defaultdict = defaultdict(list)

    def _run(self) -> None:
        """
        Get event from queue and then process it.
        """
        print(f"Event engine started. Thread ID: {get_ident()}")
        while self._active:
            try:
                event: Event = self._queue.get(block=True, timeout=1)
                self._process(event)
            except Empty:
                # logger.info(f"Event Queue empty. Thread ID: {get_ident()}")
                pass

    def _process(self, event: Event) -> None:
        """
        First distribute event to those handlers registered listening
        to this type.

        Then distribute event to those general handlers which listens
        to all types.
        """
        if event.type in self._handlers:
            for handler in self._handlers[event.type]:
                handler(event)

    def start(self) -> None:
        """
        Start event engine to process events and generate timer events.
        """
        self._active = True
        self._thread.start()

    def stop(self) -> None:
        """
        Stop event engine.
        """
        self._active = False
        self._thread.join()

    def put(self, event: Event) -> None:
        """
        Put an event object into event queue.
        """
        self._queue.put(event)

    def register(self, etype: str, handler: HandlerType) -> None:
        """
        Register a new handler function for a specific event type. Every
        function can only be registered once for each event type.
        """
        handler_list: list = self._handlers[etype]
        if handler not in handler_list:
            handler_list.append(handler)

    def unregister(self, etype: str, handler: HandlerType) -> None:
        """
        Unregister an existing handler function from event engine.
        """
        handler_list: list = self._handlers[etype]

        if handler in handler_list:
            handler_list.remove(handler)

        if not handler_list:
            self._handlers.pop(type)

def process_sleep_event(event):
    name, sleepseconds = event.data
    print(f"In {get_ident()}, Task {name} sleep {sleepseconds} start @ {datetime.now().isoformat()}")
    sleep(sleepseconds)
    print(f"In {get_ident()}, Task {name} sleep {sleepseconds} end @ {datetime.now().isoformat()}")

if __name__ == "__main__":
    print(f"main thread: {get_ident()} start  @ {datetime.now().isoformat()}")
    engine = EventEngine(using_priority=True)
    print(f"using {type(engine._queue)}")
    engine.register(EVENT_SLEEP, process_sleep_event)
    engine.start()

    engine.put(Event(EVENT_SLEEP, ("1st_1sec_event", 1)))
    engine.put(Event(EVENT_SLEEP, ("2nd_1sec_event", 1), priority=50))
    engine.put(Event(EVENT_SLEEP, ("3rd_5sec_event", 5)))
    engine.put(Event(EVENT_SLEEP, ("4th_5sec_event", 5), priority=200))
    sleep(13)
    engine.put(Event(EVENT_SLEEP, ("5th_5sec_event", 3), priority=300))

    engine.stop()
    print(f"main thread: {get_ident()}, exit @ {datetime.now().isoformat()}")

```

输出：
```
main thread: 117652 start  @ 2024-03-16T15:58:45.398626
using <class 'queue.PriorityQueue'>
Event engine started. Thread ID: 102312
In 102312, Task 4th_5sec_event sleep 5 start @ 2024-03-16T15:58:45.399626
In 102312, Task 4th_5sec_event sleep 5 end @ 2024-03-16T15:58:50.402557
In 102312, Task 3rd_5sec_event sleep 5 start @ 2024-03-16T15:58:50.402557
In 102312, Task 3rd_5sec_event sleep 5 end @ 2024-03-16T15:58:55.413506
In 102312, Task 1st_1sec_event sleep 1 start @ 2024-03-16T15:58:55.413506
In 102312, Task 1st_1sec_event sleep 1 end @ 2024-03-16T15:58:56.418677
In 102312, Task 2nd_1sec_event sleep 1 start @ 2024-03-16T15:58:56.418677
In 102312, Task 2nd_1sec_event sleep 1 end @ 2024-03-16T15:58:57.424852
In 102312, Task 5th_5sec_event sleep 3 start @ 2024-03-16T15:58:58.403006
In 102312, Task 5th_5sec_event sleep 3 end @ 2024-03-16T15:59:01.406562
main thread: 117652, exit @ 2024-03-16T15:59:01.406562
```


## 参考资料
> - [官方文档](https://docs.python.org/3/library/queue.html#module-queue)

