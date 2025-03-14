---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: dashboard
title: Understand Eventlet
---

<!-- Include Prism.js CSS -->
<link href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.25.0/themes/prism.min.css" rel="stylesheet" />

<section>
    <h1 class="text-4xl font-bold">Understanding Eventlet</h1>
    <p class="mt-10 text-xl">Eventlet is a Python concurrent network programming library created nearly 18 years ago. Its intent is to propose asynchronous feature at destination of the Python ecosystem. Eventlet has been created at a time where the Python standard library was not designed to support async. Since then, a lot of water has flowed under the bridge, and AsyncIO emerged from async waves.</p>
</section>
<section>
    <h2 class="mt-10 text-3xl font-bold">The promises of Eventlet</h2>

    <h3 class="mt-10 text-2xl font-bold">Magic Asynchronicity</h3>
    <p class="mt-10 text-xl">Eventlet was created with <a href="https://eventlet.readthedocs.io/en/latest/history.html#history" class="text-cyan-400" target="_blank">good intentions</a>. The main promise of Eventlet was to allow people to transform existing synchronous code in an asynchronous code without even rewriting one line of that existing code. Making asynchronous implicit. The magic behind this behavior is due to <a href="https://en.wikipedia.org/wiki/Monkey_patch" class="text-cyan-400" target="_blank">monkey patching</a>.</p>
    <p class="mt-10 text-xl"> This trick, monkey patching, is used to transform all the blocking IO of the stack loaded at runtime into non blocking IO. This is done by Eventlet by modifying the internal of the standard library through specific monkey patches.</p>
    <p class="mt-10 text-xl">The problem is that it makes Eventlet sensitive to each new version of Python. In parallel of that, one version of Eventlet needs to be compatible with several versions of Python at the same time, between 4 or 5 versions. CPython is daily maintained by <a href="https://github.com/python/cpython/graphs/contributors" class="text-cyan-400" target="_blank">hundred of developers</a> where Eventlet is maintained by <a href="https://github.com/eventlet/eventlet/graphs/contributors" class="text-cyan-400" target="_blank">2 or 3 persons</a>. That's not the same scale. In other words it is almost impossible to keep Eventlet synced with CPython.</p>
    <p class="mt-10 text-xl">Python <a href="https://peps.python.org/pep-0020/" class="text-cyan-400" target="_blank">advocate for explicit</a>. Eventlet on the other hand, <a href="https://eventlet.readthedocs.io/en/latest/patching.html#monkeypatching-the-standard-library" class="text-cyan-400" target="_blank">by using monkey patching</a>, introduce lot of implicit behaviours. Unfortunatelly, with time, all these implicit tricks finish by lot of incidence on existing code, that should not have been impacted. <a href="https://eventlet.readthedocs.io/en/latest/patching.html" class="text-cyan-400" target="_blank">Implicit async</a> transformations became <a href="https://github.com/search?q=%22if+eventlet%22+language%3APython+&type=code" class="text-cyan-400" target="_blank">explicit bug fixes here and there</a>.</p>
    <p class="mt-10 text-xl">The promise is not kept.</p>

    <h3 class="mt-10 text-2xl font-bold">Cooperative Multitasking</h3>
    <p class="mt-10 text-xl">Eventlet is presented as a cooperative multitasking library, but in practice, its behavior is not entirely cooperative in all situations.</p>
    <p class="mt-10 text-xl">Eventlet is supposed to be cooperative, it utilizes <a href="https://eventlet.readthedocs.io/en/latest/modules/greenthread.html" class="text-cyan-400" target="_blank">green threads</a> (or <a href="https://greenlet.readthedocs.io/en/latest/" class="text-cyan-400" target="_blank">greenlets</a>), which are lightweight execution units managed in user space. These green threads explicitly yield control through monkey patching of standard libraries (socket, time, threading, etc.). It is based on an event loop similar to asyncio, but with an approach based on lightweight threads rather than coroutines.<br>In the example below, <code class="text-xl language-python">eventlet.sleep()</code> explicitly allows other tasks to run:</p>

    <pre class="line-numbers"><code class="language-python">import eventlet

def task1():
    print("Task 1 starts")
    eventlet.sleep(2)  # Yields control here
    print("Task 1 ends")

def task2():
    print("Task 2 starts")
    eventlet.sleep(1)  # Yields control here
    print("Task 2 ends")

eventlet.spawn(task1)
eventlet.spawn(task2)

eventlet.sleep(3)  # Waits for tasks to execute</code></pre>

    <p class="mt-10 text-xl">But in reality, Eventlet is not always truly cooperative. Indeed, unpatched blocking calls <a href="https://greenlet.readthedocs.io/en/latest/python_threads.html" class="text-cyan-400" target="_blank">can block the entire program</a>. If an external library uses network or I/O calls without being "monkey patched," these calls will block all green threads. Example of unpatched code that will block the entire program:</p>
    <pre class="line-numbers"><code class="language-python">import eventlet
import time

def blocking_task():
    print("Starting a blocking task")
    time.sleep(3)  # Uses standard time.sleep, blocks all of Eventlet
    print("Blocking task ends")

eventlet.spawn(blocking_task)
eventlet.sleep(1)  # This line does not allow other tasks to run</code></pre>
    <p class="mt-10 text-xl">Eventlet do not provide any automatic preemption. Unlike classic threads or asyncio coroutines, a green thread does not yield control unless it performs a cooperative operation (<code class="text-xl language-python">eventlet.sleep()</code>, <code class="text-xl language-python">socket.recv()</code>, etc.). If a task runs a long loop without ever yielding control, it blocks all others.</p>
    <p class="mt-10 text-xl">Conclusion, Eventlet is theoretically cooperative, but in practice, it heavily relies on the proper behavior of the code and libraries used. It is not truly cooperative in all circumstances: if a blocking call or an infinite loop without <code class="text-xl language-python">sleep()</code> occurs, the entire program can freeze.</p>
</section>
<div class="mt-10 flex justify-between">
    <a href="{{ site.baseurl }}{% link getting-started.md %}" class="inline-block bg-gradient-to-r from-yellow-400 to-yellow-600 text-gray-900 font-semibold py-3 px-8 rounded hover:scale-105 transition-transform">
        <i class="fas fa-arrow-left mr-2"></i>Getting Started
    </a>
    <a href="{{ site.baseurl }}{% link alternatives.md %}" class="inline-block bg-gradient-to-r from-cyan-400 to-blue-600 text-gray-900 font-semibold py-3 px-8 rounded hover:scale-105 transition-transform">
        Understanding the Alternatives<i class="fas fa-arrow-right ml-2"></i>
    </a>
</div>

<!-- Include Prism.js JavaScript -->