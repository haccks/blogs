---
title: How to use QThread correctly?
description: "Whether one should subclass QThread or use worker object and move it to the thread depends on the use case"
date: 2023-05-18 14:00:00 +0530
categories: [python]
tags: [python3, PySide6, Qt, QThread, PyQt6]     # TAG names should always be lowercase
render_with_liquid: false
---

## Prerequisite:
+ [Basic python programming](https://docs.python.org/3/tutorial/index.html)
+ [Basic understanding of threads](https://docs.python.org/3/library/threading.html)
+ Familiarity with [Qt for Python](https://doc.qt.io/qtforpython-6/) (PySide6 or PyQt6)
+ [Signals and Slots](https://doc.qt.io/qtforpython-6/tutorials/basictutorial/signals_and_slots.html)

## Intro

When it comes to use `QThread`, internet is divided into two groups:   

1. Subclass `QThread` and reimplement `run()`
2. Use *worker objects* by moving them to the thread using `QObject().moveToThread()`

There is actually a long debate on why not to subclass `QThread` in an old blog [You're doing it wrong](https://www.qt.io/blog/2010/06/17/youre-doing-it-wrong) posted by **Bradley T. Hughes** (one of the core developer of Qt). The blog suggests using the 2<sup>nd</sup> approach.  
After almost 3 years one of his colleague  **Olivier Goffart** posted a counter blog [You were not doing so wrong](https://woboq.com/blog/qthread-you-were-not-doing-so-wrong.html). This caused lots of confusion among programmers deciding what approach to use when using `QThread`.

As of the newest release of Qt (Qt6 at the time of writing this blog), documentation gives example of both approach. I dig a bit deeper and decided to write this blog and explain how really `QThread` works and when to use either approach with examples. I will use [`PySide6`](https://doc.qt.io/qtforpython-6/) for this blog and all of its examples (with minor changes in imports examples will work with PyQt6).  

To use `QThread` correctly one has to understand the basics of this object. Let's start with event loop...

## What is an event Loop?

An event loop is a loop that listen for any events like: user input, network traffic, sensors, timers etc. and dispatches these events to the target objects. In Qt, event loop starts when object's `exec()` method is called. `QApplication.exec()` starts the main event loop and wait until `exit()` is called. 

>*"It is necessary to call this function to start event handling. The main event loop receives events from the window system and dispatches these to the application widgets."* -- [`exec()` doc](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QEventLoop.html#PySide6.QtCore.PySide6.QtCore.QEventLoop.exec)
{: .prompt-info}    

A Thread can also have its own event loop.  

> An event loop in a thread makes it possible for the thread to use certain non-GUI Qt classes that require the presence of an event loop (such as `QTimer`, `QTcpSocket`, and `QProcess`). It also makes it possible to connect signals from any threads to slots of a specific thread.
{: .prompt-info}

## What is a [thread affinity](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#thread-affinity)?

A `QObject` instance is said to have a *thread affinity*, or that it *lives* in a certain thread. That means, when a `QObject` receives a *queued signal* or a *posted event*, the slot or event handler will run in the thread that the object lives in.

As documentation says:  
>*"By default, a `QObject` lives in the thread in which it is created. An object's thread affinity can be queried using [`thread()`](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#PySide6.QtCore.PySide6.QtCore.QObject.thread) and changed using [`moveToThread()`](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#PySide6.QtCore.PySide6.QtCore.QObject.moveToThread)."*    

When a `QObject` is moved to another thread, all its children will be automatically moved too.

>`moveToThread()` will fail if the `QObject` has a parent.
{: .prompt-info}

<!-- >Note that `QObject` class is [*reentrant*](https://doc.qt.io/qt-6/threads-qobject.html#qobject-reentrancy) unlike `QWidget` and all it's subclasses and therefore it can be used from multiple threads simultaneously. `QWidget` not being reentrant is why it, and it's subclasses, can only be used in main thread.
{: .prompt-info}  -->

<!-- >Once the thread affinity of a `QObject` is changed by `moveToThread()` do not touch this object in the calling thread. 
{: .prompt-warning} -->

## What is a QThread and How it works?

If an expensive/blocking operation is performed in main thread then it causes UI to freeze. To avoid this problem this task need to move to a separate thread. Qt has `QThread` class to perform long non-GUI tasks in a separate thread.  

>Never update GUI through a secondary thread. It should be done through main thread only.
{: .prompt-warning}

Quoting from the [`QThread` doc](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QThread.html#detailed-description):  
> *"A `QThread` object manages one thread of control within the program. `QThreads` begin executing in `run()`. By default, `run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread."*

Let's breakdown the above paragraph first,
1. *A `QThread` object manages one thread of control*: Yes, a **`QThread` is not actually a thread but a thread manager (manages one actual system thread)**.   

2. *`QThread`s begin executing in `run()`*: When `QThread.start()` is called then it eventually calls `run()` method of `QThread` and then this `run()` method starts the actual thread.

3. *By default, `run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread*: This event loop is optional. If re-implementation of `run()` doesn't call `exec()` then there will no event loop inside the thread.

Let's create a simple toy app to demonstrate downloading a large file. This will have a simple progressbar and a push button. When user press the button download will start and at the same time progress bar will be updated showing the progress of the downloading file.

### 1. Subclass `QThread` (Without event loop)  

> Note that when you subclass `QThread`, only code inside `run()` will execute in a separate thread. 
{: .prompt-info}   

```py
import sys
import time
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, \
    QPushButton, QVBoxLayout, QProgressBar
from PySide6.QtCore import QThread, Signal, Slot


class WorkerThread(QThread):
    progress = Signal(int)

    def __init__(self, n):
        super().__init__()
        self.n = n

    def do_work(self):
        for i in range(1, self.n+1):
            self.progress.emit(i)
            time.sleep(1)
            
    def run(self):
        """Override run method
        """
        self.do_work()


class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.setGeometry(100, 100, 300, 50)
        self.setWindowTitle('QThread Demo')

        # setup widget
        self.widget = QWidget()
        layout_v = QVBoxLayout()
        self.widget.setLayout(layout_v)
        self.setCentralWidget(self.widget)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)

        self.download_btn = QPushButton(text='Download')
        self.download_btn.clicked.connect(self.download)

        layout_v.addWidget(self.progress_bar)
        layout_v.addWidget(self.download_btn)

        self.show()

    @Slot()
    def download(self):
        num = 5
        self.download_btn.setEnabled(False)
        self.progress_bar.reset()
        self.progress_bar.setMaximum(num)

        self.worker_thread = WorkerThread(num)
        self.worker_thread.progress.connect(self.progress_bar.setValue)
        self.worker_thread.finished.connect(self.finished)
        self.worker_thread.finished.connect(self.worker_thread.deleteLater)
        self.worker_thread.start()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MainWindow()
    app.exec()

```
{: file="demo-subclass.py" }

This will have output below: 

Todo: Add gif
![qthread-1](/assets/img/media/qthread-1.png){: width="400" height="50}

Let's walkthrough the above code. We defined `WorkerThread` class, a subclass of `QThread`

```py
class WorkerThread(QThread):
    progress = Signal(int)

    def __init__(self, n):
        super().__init__()
        self.n = n

    def do_work(self):
        for i in range(1, self.n+1):
            self.progress.emit(i)
            time.sleep(1)
            
    def run(self):
        """Override run method
        Note that we are not calling `exec()` here so there will not be any local event loop to this thread!
        """
        self.do_work()
```
{: .nolineno }

We reimplemented `QThread().run()` method which calls `do_work` method. `do_work` inside a loop is emitting a custom signal called `progress` and then sleeps for 1 second to imitate a long running job.

The `download_btn` in `MainWindow` class is connected to a slot `download()`.

```py
    @Slot()
    def download(self):
        num = 5
        self.download_btn.setEnabled(False)
        self.progress_bar.reset()
        self.progress_bar.setMaximum(num)

        self.worker_thread = WorkerThread(num)
        self.worker_thread.progress.connect(self.progress_bar.setValue)
        self.worker_thread.finished.connect(self.finished)
        self.worker_thread.finished.connect(self.worker_thread.deleteLater)
        self.worker_thread.start()
```
{: .nolineno }

An instance of `WorkerThread` is instantiated in this slot and it's custom signal `progress` and builtin signal `finished` is connected to the required slots. Finally, a call to `worker_thread.start()` will eventually call `run()` and run a separate thread to complete `do_work()`. There will not be any event loop running in this thread.  

When you subclass `QThread` then keep in mind:  

+ `QThread` instance (`worker_thread`) lives in the old thread (main thread) that instantiated it, not in the new thread that calls `run()`
+ `run()` executes the new thread, therefore only code inside `run()` will execute in the new thread. So you need to override this method  
+ The thread will exit after the `run()` function has returned 
+ There will not be any event loop running in the thread unless you call `exec()` inside `run()`

### 2. Worker-Object approach (with event loop)  
We will define a worker class, a subclass of `QObject`. This class will have a slot `do_work()` and will do the expensive task.  

>*"The code inside the Worker's slot would then execute in a separate thread."*
{: .prompt-info}

We can use worker-object approach to build the same app.  

```py
import sys
import time
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget,
                               QPushButton, QVBoxLayout, QProgressBar)
from PySide6.QtCore import QThread, QObject, Signal, Slot


class Worker(QObject):
    progress = Signal(int)
    finished = Signal()

    def __init__(self, n):
        super().__init__()
        self.n = n

    @Slot()
    def do_work(self):
        for i in range(1, self.n+1):
            time.sleep(1)
            self.progress.emit(i)

        self.finished.emit()


class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.setGeometry(100, 100, 300, 50)
        self.setWindowTitle('QThread Demo')

        # setup widget
        self.widget = QWidget()
        layout_v = QVBoxLayout()
        self.widget.setLayout(layout_v)
        self.setCentralWidget(self.widget)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)

        self.btn_start = QPushButton('Download', clicked=self.download)

        layout_v.addWidget(self.progress_bar)
        layout_v.addWidget(self.btn_start)

        self.show()

    def download(self):
        num = 5
        self.btn_start.setEnabled(False)
        self.progress_bar.reset()
        self.progress_bar.setMaximum(num)

        self.worker = Worker(num)
        self.worker_thread = QThread()
        self.worker.moveToThread(self.worker_thread)

        self.worker.progress.connect(self.progress_bar.setValue)
        self.worker.finished.connect(self.finished)
        self.worker_thread.started.connect(self.worker.do_work)
        self.worker_thread.finished.connect(self.worker.deleteLater)
        self.worker_thread.finished.connect(self.worker_thread.deleteLater)

        # Won't work!
        # Because thread hosts an event loop, and it never
        # quits (so no finished signal emitted by the thread
        # object) until terminated manually by the worker object.
        # self.worker_thread.finished.connect(self.finished)

        # start the thread
        self.worker_thread.start()

    def finished(self):
        self.btn_start.setEnabled(True)
        self.worker_thread.quit()
        self.worker_thread.wait()


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MainWindow()
    app.exec()

```
{: file="demo-worker-object.py" }  

While using *worker-object* approach you need to take care of these things:  

+ Make sure the lifetime of `worker` object is as same as the `worker_thread` or beyond. Making `worker` an attribute of `MainWindow` class is intentional. If you do `worker = Worker(num)` then lifetime of this object will be done before executing `do_work` in the new thread 

+ Make sure [`worker` must not have any parents](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QObject.html#PySide6.QtCore.PySide6.QtCore.QObject.moveToThread) else it can't be moved to the new thread (the one started by `run()`)

+ `worker_thread` lives in main thread, i.e. it has thread affinity of main thread (the one it has been instantiated)  

<!-- + Notice that `moveToThread()` is called before connecting to any signals. Always move `worker` to the `worker-thread` before connecting signals -->

+ Though `worker` is instantiated in main thread, after call to `moveToThread()`, `worker` object will live in the new thread (i.e. all the signals/events for `worker` object will be handled in the new thread)
<!-- (more details [here](https://doc.qt.io/qt-6/threads-qobject.html#per-thread-event-loop))  -->

+ A custom signal `worker.finished()` is used to call the `finish()` slot and not the builtin `worker_thread.finished()`, why?  Because thread hosts an event loop, and it never quits (so no finished signal emitted by the thread object) until terminated manually by the `worker` object.  

+ Slots of `worker` instance is invoked in the new thread, but if this is called as normal member function then it will be invoked in the main thread.

### 3. Modifying examples to abort the download: Todo 

So far so good. Let's add one more functionality to our small app, a cancel button. This will give users an option to cancel the download in between.   

Let's add a `Cancel` button to the app. Update `__init__()` method in `MainWindow` class (in both examples) 

```py
    #...

    self.cancel_btn = QPushButton(text='Cancel')
    # self.cancel_btn.clicked.connect(self.cancel)

    self.container_frame = QFrame()
    layout_h = QHBoxLayout()
    self.container_frame.setLayout(layout_h)

    layout_h.addWidget(self.download_btn)
    layout_h.addWidget(self.cancel_btn)

    layout_v.addWidget(self.progress_bar)
    layout_v.addWidget(self.container_frame)  # Add container_frame to layout_v

    # ...
   
```
{: .nolineno }

Make the following changes in `WorkerThread` class of `demo-subclass.py` {: .filepath}
```py
def WorkerThread(QThread)
    progress = Signal(int)

    def __init__(self, n):
        super().__init__()
        self.n = n
        self.is_cancel = False

    def stop_download(self):
        self.is_cancel = True

    def do_work(self):
        for i in range(1, self.n + 1):
            if self.is_cancel:
                break
            self.progress.emit(i)
            time.sleep(1)
   
    # ...

class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        # ...

    @Slot()
    def download(self):
        self.cancel_btn.setEnabled(True)
        # ...

        self.cancel_btn.clicked.connect(self.worker.stop_download)
        self.worker_thread.start()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)
        self.cancel_btn.setEnabled(False)
        self.progress_bar.reset()

```
{: .nolineno }
{: file="demo-subclass-cancel.py" }
<sub>[Code on github]()</sub>

Todo: Add a gif

<!-- >*"It is generally unsafe to provide slots in your `QThread` subclass, unless you protect the member variables with a mutex."* -- [Qt Doc](https://doc.qt.io/qt-6/threads-qobject.html#accessing-qobject-subclasses-from-other-threads)
{: .prompt-warning} -->  

As we already know "only code inside `run()` will execute in a separate thread". While download is in progress and managed by worker thread, when we press `Cancel` button then the method `stop_download()` will execute in the main thread (`worker_thread` lives in main thread and hence the method`stop_download()`). Both of these jobs will be done in parallel.   

Note that we are setting `worker_thread.is_cancel` from main thread while reading its value in worker thread, is this a problem? Not really. In this case `is_cancel` is a boolean and is thread safe as it does not involve any *non-atomic operations* and is modified by main thread only. (More info [here](https://stackoverflow.com/a/77173551/2455888))

 But when in doubt, you can use mutex. If you wish, add `self.mutex = QMutex()` line to the `__init__()` method of `WorkerThread` and update the `stop_download()` slot as below

```py
def stop_download(self):
    self.mutex.lock()
    self.is_cancel = True
    self.mutex.unlock()
```
{: .nolineno}

 Let's modify `demo-worker-object.py` {: .filepath } file as follows

 ```py
 class Worker(QObject):
    progress = Signal(int)
    finished = Signal()

    def __init__(self, n):
        super().__init__()
        self.n = n
        self.is_cancel = False

    @Slot()
    def stop_download(self):
        print('Stop requested!')
        self.is_cancel = True

    @Slot()
    def do_work(self):
        for i in range(1, self.n + 1):
            if self.is_cancel:
                break
            self.progress.emit(i)
            time.sleep(1)

        self.finished.emit()


class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        # ...

    @Slot()
    def download(self):
        self.cancel_btn.setEnabled(True)
        # ...

        self.cancel_btn.clicked.connect(self.worker.stop_download)
        self.worker_thread.start()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)
        self.cancel_btn.setEnabled(False)
        self.worker_thread.quit()
        self.worker_thread.wait()
        self.progress_bar.reset()
 
 ```
{: .nolineno }
{: file="demo-subclass-cancel.py" }  

Todo: Add a gif  

As you can see, clicking `Cancel` is not doing what we expected it to do! *But, why?* 

Unlike subclass approach where only code in `run()` method is executed in a separate thread, worker-object approach has different rule: 

>*"Code inside the worker's slot will be executed in a separate thread."*  

Slot `stop_download()` will execute in the worker thread. Since `do_work()` is a blocking task, till this job finishes, the local event loop will wait to get back the control and meanwhile all the incoming signals will be queued in the event-queue of the worker thread. The slot `stop_download()` is invoked only after control returns to the event loop of the thread `worker_thread` is managing.


To fix this problem somehow we need to invoke `stop_download()` from main thread. We will add a new signal `cancelled` in `MainWindow` class and a new slot `cancel()`. This signal will be emitted in `cancel()` slot.

```py
class Worker(QObject):
    progress = Signal(int)
    finished = Signal()

    def __init__(self, n):
        super().__init__()
        self.n = n
        self.is_cancel = False

    @Slot()
    def stop_download(self):
        self.is_cancel = True

    @Slot()
    def do_work(self):
        for i in range(1, self.n + 1):
            if self.is_cancel:
                break
            self.progress.emit(i)
            time.sleep(1)

        self.finished.emit()


class MainWindow(QMainWindow):
    cancelled = Signal()
    def __init__(self, *args, **kwargs):
        # ...

    @Slot()
    def download(self):
        self.cancel_btn.setEnabled(True)

        # ...

        self.cancel_btn.clicked.connect(self.cancel)
        self.worker_thread.start()

    @Slot()
    def cancel(self):
         self.worker.stop_download()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)
        self.cancel_btn.setEnabled(False)
        self.worker_thread.quit()
        self.worker_thread.wait()
        self.progress_bar.reset()

```
{: .nolineno }
{: file="demo-worker-object-cancel.py" }

<sub>[Code on github]()</sub>

Todo: Add a gif

Note that `cancel_btn.clicked` signal is not connected to `worker.stop_download()` slot rather it is connected to `cancel()` slot of `MainWindow`. Since `cancel()` is invoked in main thread and hence it will invoke `worker.stop_download()` as a normal method in main thread instead of worker thread.

## Conclusion

For this particular app both approach has been used. But as we can see, for this app

1. No event loop is needed
2. No signals/slots need to be handled inside the secondary thread (we are emitting signals though)

In such case where you want to perform some expensive operation in another thread, where the thread does not need to receive any signals or events, subcalss `QThread`.

When to use worker-object approach?   
1.*When you need an event loop in `QThread`*. Certain non GUI classes (such as `QTimer`, `QTcpSocket`, and `QProcess`) requires the presence of event loop. If you are using instances of these classes in your thread then you will have to use worker-object approach.  

2. *If you have to handle signals/slots in the secondary thread*.