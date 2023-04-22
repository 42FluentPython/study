Parallelism is a special case of concurrency

- threading, multiprocessing, and asyncio

Youtube, DropBox, Instagram, Reddit, … Python with concurrency!

# What’s New in This Chapter

- first illustration of Python’s three approaches to concurrency: threads, processes, and native coroutines.
- concurrent.futures, asyncio

This book is saying. This chapter is really important!!!

# The Big Picture

There are many factors that concurrent programming hard

function basic logic is not available in concurrency

a thread or a process is not cheap

message queue

worker.

messages and queues

coroutine is cheap to start

# A Bit of Jargon

- Concurrency
- Parallelism
- Execution unit, 실행단위
- Process, 동작중인 프로그램
- Thread, 프로세스 내의 실행 흐름
- Coroutine, 아직도 느낌이 안옴
- Queue, 데이터구조
- Lock: 잠궈. mutex, 상호배제
- Contention: 싸우자

## Processes, Threads, and Python’s Infamous GIL

1. Each instance of the Python interpreter is a process. `multiprocessing`, `concurrent.futures`
2. The Python interpreter uses a single thread to run the user’s program and the memory garbage collector. `threading`, `concurrent.futures`
3. GIL, Global Interpreter Lock. Only one thread can execute Python code at any time.
4. 5ms by default, `sys.getswitchinterval()`, `sys.setswitchinterval(s)`
5. built-in function or an extension written in C … can release GIL
6. disk I/O, network I/O, time.sleep(), NumPy/SciPy, lib, bz2 modules release the GIL.
7. buffer protocol?!, byte array, array.array, Numpy arrays
8. The effect of the GIL on network programming with Python threads is relatively small
9. Contention over the GIL slows down compute-intensive Python threads.
10. To run CPU-intensive Python code on multiple cores, you must use multiple Python processes.

### threading

threading = can be an appropriate model

Due to the GIL, only one thread can execute Python code at once

multiprocessing, concurrent.fu tures.ProcessPoolExecutor

# A Concurrent Hello World

concurrent “Hello World” = simplest program to show how Python can “walk and chew gum.”

multiprocessing, threading, asyncio

threading is look familiar

## Spinner with Threads

처리중입니다~~를 표시하는 스레드를 따로 분리.

itertools.cycle()

threading.Thread

threading.Event

## Spinner with Processes

depends on the operating system scheduler

multiprocessing.Process

multiprocessing.Event

multiprocessing.synchronize

how to communicate between processes that are isolated by the operating system

multiprocessing.shared_memeory

semaphore

## Spinner with Coroutines

application-level event loop.

queue of pending coroutines

one thread

async, await, asyncio

asyncio.run(coro())

asyncio.create_task(coro())

await coro()

```python
import asyncio
import itertools

async def spin(msg: str) -> None:
    for char in itertools.cycle(r"\|/-"):
        status = f"\r{char} {msg}"
        print(status, flush=True, end="")
        try:
            await asyncio.sleep(0.1)
        except asyncio.CancelledError:
            break
    blanks = " " * len(status)
    print(f"\r{blanks}\r", end="")

async def slow() -> int:
    await asyncio.sleep(3)
    return 42

def main() -> None:
    result = asyncio.run(supervisor())
    print(f"Answer: {result}")

async def supervisor() -> int:
    spinner = asyncio.create_task(spin("thinking!"))
    print(f"spinner object: {spinner}")
    result = await slow()
    spinner.cancel()
    return result

if __name__ == "__main__":
    main()
```

### Greenlet and gevent

- greenlet
- gevent
- wsgi
- nonblocking
- socket
- gunicorn

## Supervisors Side-by-Side

important contents.

threads versus coroutines

# The Real Impact of the GIL

a well-designed network library will release the GIL while waiting for the network

## Quick Quiz

1. Answer for multiprocessing → Ok
2. Answer for threading → Ok
3. Answer for asyncio → freeze

### Power Napping with sleep(0)

# A Homegrown Process Pool

소수체크 프로그램

## Process-Based Solution

멀티프로세스가 4배 이상 빠름

## Understanding the Elapsed Times

컴퓨터의 코어와 하이퍼스레딩

## Code for the Multicore Prime Checker

worker result

queue

코드 머리에 하나도 안들어옴

## Experimenting with More or Fewer Processes

6코어 이상 가면 똑같다. 왜냐면 필자 cpu가 6코어라.

## Thread-Based Nonsolution

context switching

side effect

swapping memory page

# Python in the Multicore World

2005년부터 CPU 클럭 향상은 멈추고 멀티코어의 시대가 열렸다.

Fortran, C가 Python보다 100배는 빠름.

GIL이 하는 역할이 큼

## System Administration

Python이 참 많이도 쓰여요.

concurrent.futures

## Data Science

## Server-Side Web/Mobile Development

IMPORTANT

## WSGI Application Servers

IMPORTANT

## Distributed Task Queues

# Chapter Summary

# Further Reading

## Concurrency with Threads and Processes

## The GIL

## Concurrency Beyond the Standard Library

## Concurrency and Scalability Beyond Python
