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

As of the newest release of Qt (Qt6 at the time of writing this blog), documentation gives example of both approach. I dig a bit deeper and decided to write this blog and explain how really `QThread` works and when to use either approach with example. I will use [`PySide6`](https://doc.qt.io/qtforpython-6/) for this blog and all of its example (with minor changes in imports examples will work with PyQt6).  

To use `QThread` correctly one has to understand the basics of this object. Let's start with event loop...

## What is an vent Loop?

An event loop is a loop that listen for any events like: user input, network traffic, sensors, timers etc. and dispatches these events to the target objects. In Qt, event loop starts when object's `exec()` method is called. [`QApplication().exec()`](https://doc.qt.io/qt-6/qeventloop.html#exec) starts the main event loop and wait until `exit()` is called. 

>*"It is necessary to call this function to start event handling. The main event loop receives events from the window system and dispatches these to the application widgets."* -- [`exec()` doc](https://doc.qt.io/qt-6/qeventloop.html#exec)
{: .prompt-info}  

If a long running task is performed in main thread then it causes UI to freeze. To avoid this problem this task need to move to a new separate thread. Qt has `QThread` class to perform long no-GUI tasks in a separate thread.  

>Never update GUI through a secondary thread. It should be done through main thread only.
{: .prompt-warning}

## What is a [thread affinity](https://doc.qt.io/qt-6/qobject.html#thread-affinity)?

In Qt, if a `QObject` is owned by or lives in a thread then it is said this object has thread affinity. As documentation says:  
>*"By default, a `QObject` lives in the thread in which it is created. An object's thread affinity can be queried using thread() and changed using `moveToThread()`."*    

When a `QObject` is moved to another thread, all its children will be automatically moved too.

>`moveToThread()` will fail if the `QObject` has a parent.
{: .prompt-info}

>Note that `QObject` class is [*reentrant*](https://doc.qt.io/qt-6/threads-qobject.html#qobject-reentrancy) unlike `QWidget` and all it's subclasses and therefore it can be used from multiple threads simultaneously. `QWidget` not being reentrant is why it, and it's subclasses, can only be used in main thread.
{: .prompt-info} 

>Once the thread affinity of a `QObject` is changed by `moveToThread()` do not touch this object in the calling thread. 
{: .prompt-warning}

## What is a QThread and How it works?

Quoting from the documentation:  
> *"A `QThread` object manages one thread of control within the program. `QThreads` begin executing in `run()`. By default, `run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread."*

Let's breakdown the above paragraph first,
1. *A `QThread` object manages one thread of control*: Means if same thread is used to do two different long running tasks at the same time then they won't be done in parallel. One task has to wait for other to finish.

2. *`QThreads` begin executing in `run()`*: When `QThread().start()` is called then it eventually calls `run()` method of `QThread` and then this `run()` method starts the actual thread. So, basically **`QThread` is not actually a thread but a wrapper around a thread**.  

3. *`run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread*: A local event loop starts within the thread and wait until `exit()` is called. This event loop is optional (more on this later).


Let's create a simple toy app to demonstrate downloading a large file. This will have a simple progressbar and a push button. When user press the button download will start and at the same time progress bar will be updated showing the progress of the downloading file.

### 1. Subclass `QThread` (Without event loop)

```py
import sys
import time
from PySide6.QtWidgets import QApplication, QMainWindow, QWidget, \
    QPushButton, QVBoxLayout, QProgressBar, QMessageBox
from PySide6.QtCore import QThread, Signal, Slot, QTimer


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


class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.setGeometry(100, 100, 300, 50)
        self.setWindowTitle('QThread Demo')

        # setup widget
        self.widget = QWidget()
        layout = QVBoxLayout()
        self.widget.setLayout(layout)
        self.setCentralWidget(self.widget)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)

        self.download_btn = QPushButton(text='Download')
        self.download_btn.clicked.connect(self.download)

        layout.addWidget(self.progress_bar)
        layout.addWidget(self.download_btn)

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
        QMessageBox.information(self, "Download status", "Download Complete!")
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

Let's start with `WorkerThread`, a subclass of `QThread`. 

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

We reimplemented `QThread().run()` method which calls `do_work` method. `do_work` looping 5 times and emitting a custom signal called `progress` and then sleeps for 1 second to imitate a long running job.

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

In this slot an instance of `WorkerThread` is instantiated and it's custom signal `progress` and builtin signal `finished` is connected to the required slots. Finally, a call to `worker_thread.start()` will eventually call `run()` and run a separate thread to complete `do_work()`. There will not be any event loop running in this thread. This is a simple working example. 

> Note that when you subclass `QThread`, only code inside `run()` will execute in a separate thread. 
{: .prompt-info}   

### 2. Worker-Object approach (with event loop)

We can also use worker-object approach to build the same app.  

```py
import sys
import time
from PySide6.QtWidgets import (QApplication, QMainWindow, QWidget,
                               QPushButton, QVBoxLayout, QProgressBar,
                               QMessageBox)
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
        layout = QVBoxLayout()
        self.widget.setLayout(layout)
        self.setCentralWidget(self.widget)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)

        self.btn_start = QPushButton('Download', clicked=self.start)

        layout.addWidget(self.progress_bar)
        layout.addWidget(self.btn_start)

        self.show()

    def start(self):
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
        self.worker_thread.finished.connect(self.worker.deleteLater())

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

+ Make sure [`worker` must not have any parents](https://doc.qt.io/qt-6/qobject.html#moveToThread) else it can't be moved to the `worker_thread` 

+ `worker_thread` is owned by main thread, i.e. it has thread affinity of main thread (the one it has been instantiated)  

<!-- + Notice that `moveToThread()` is called before connecting to any signals. Always move `worker` to the `worker-thread` before connecting signals -->

+ Though `worker` is instantiated in main thread, after call to `moveToThread()`, `worker_thread` will own the `worker` object (more details [here](https://doc.qt.io/qt-6/threads-qobject.html#per-thread-event-loop))  

+ A custom signal `worker.finished()` is used to call the `finish()` slot and not the builtin `worker_thread.finished()`, why?  Because thread hosts an event loop, and it never quits (so no finished signal emitted by the thread object) until terminated manually by the `worker` object.  

### 3. Modifying examples to abort the download  

So far so good. Let's add one more functionality to our small app, a cancel button. This will give users an option to cancel the download in between. First let's modify the first example `demo-subclass.py`{: .filepath}. If possible, we will try to achieve the same for second example! 

Update `__init__()` method in `MainWindow` class  

```py
    #...

    self.cancel_btn = QPushButton(text='Cancel')
    self.cancel_btn.clicked.connect(self.cancel)

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

We added a `Cancel` button and connected it's `clicked` signal to `self.cancel()` slot. How do we implement this slot? Well, first we will add another custom signal to `MainWindow` and a new slot to `WorkerThread` to handle this signal. This signal will emit when user will press `Cancel`.   



```py
class WorkerThread(QThread):
    progress = Signal(int)

    def __init__(self, n):
        super().__init__()
        self.n = n
        self.is_cancel = False

    # This slot should not be here because we are not using mutex to protect  
    # is_cancel member
    @Slot()
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
    cancelled = Signal()

    def __init__(self, *args, **kwargs):
        # ...

    @Slot()
    def download(self):
        self.cancel_btn.setEnabled(True)

        # ...

        self.cancelled.connect(self.worker_thread.stop_download)
        self.worker_thread.start()

    @Slot()
    def cancel(self):
        self.cancelled.emit()
        self.progress_bar.reset()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)
        self.cancel_btn.setEnabled(False)

```
{: .nolineno }
{: file="demo-subclass-cancel.py" }


Let's see if this works:  

Todo: Add gif  

One important point to note here is the constructor `__init__()` in `WorkerThread` will execute in the main thread (more on this later). That means the object `is_cancel` will have thread affinity of main thread and therefore we are writing to object from main thread while reading it's value from secondary thread!  

>*"It is generally unsafe to provide slots in your `QThread` subclass, unless you protect the member variables with a mutex."* -- [Qt Doc](https://doc.qt.io/qt-6/threads-qobject.html#accessing-qobject-subclasses-from-other-threads)
{: .prompt-warning}

Since it is a bad practice to have a slot in a subclass of QThread, we should move `stop_download()` slot to main thread with these changes  

```py
@Slot()
def stop_download(self):
    self.worker_thread.is_cancel = True
```
{: .nolineno}

and in `MainWindow` replace
```py
self.cancelled.connect(self.worker_thread.stop_download) 
``` 
{: .nolineno}

with 

```py
self.cancelled.connect(self.stop_download)
```
{: .nolineno}

Now let's modify second example file and see if it works...  

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
        print('cancel signal emitted')
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

        self.cancelled.connect(self.worker_thread.stop_download)
        self.worker_thread.start()

    @Slot()
    def cancel(self):
        self.cancelled.emit()
        self.progress_bar.reset()

    @Slot()
    def finished(self):
        self.download_btn.setEnabled(True)
        self.cancel_btn.setEnabled(False)
        self.worker_thread.quit()
        self.worker_thread.wait()

```
{: .nolineno }
{: file="demo-worker-object-cancel.py" }

Todo: Add a gif

As you can see, clicking `Cancel` is not doing what we expected to do! But, why? 

Let's understand why it worked for subclass approach but not for worker-object. 
[Qt doc](https://doc.qt.io/qt-6/qthread.html#details) says for subclass approach  

>*"It is important to remember that a `QThread` instance lives in the old thread that instantiated it, not in the new thread that calls `run()`. This means that all of `QThread`'s queued slots and invoked methods will execute in the old thread.*  
>
>*Unlike queued slots or invoked methods, methods called directly on the `QThread` object will execute in the thread that calls the method. When subclassing `QThread`, keep in mind that the constructor executes in the old thread while `run()` executes in the new thread."*

While download is in progress and managed by another thread, when we press `Cancel` button then the slot `stop_download()` will execute in the main thread (will have main thread affinity). Both of these jobs will be done in parallel.   

Unlike subclass approach where only code in `run()` method is executed in a separate thread, worker-object approach has different rule: 
>*"Code inside the worker's slot will be executed in a separate thread."*  

As I already said *a `QThread` object manages one thread of control*, therefore, slot `stop_download()` will execute in the new thread (same thread as of the `do_work()`). Since `do_work()` is a long job, the local event loop will be busy till this job finishes and all the incoming signals will be queued and hence the execution of slots. In other words `do_work()` is a blocking call in this new thread.  

To make this work we need to use some [synchronization mechanism](https://doc.qt.io/qt-6/threads-synchronizing.html) to update `is_cancel` attribute (we will not discuss it in this blog).

## Conclusion

For this particular app we built we can say that  

1. No event loop is needed
2. No signals or slots are handled inside the secondary thread (we are emitting signals though)
3. 