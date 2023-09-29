---
title: How to use QThread correctly (part 2)?
description: "Whether one should subclass QThread or use worker object and move it to the thread depends on the use case"
date: 2023-09-30 00:30:00 +0530
categories: [python]
tags: [python3, PySide6, Qt, QThread, PyQt6]     # TAG names should always be lowercase
render_with_liquid: false
---

<sub>This part is the continuation of the blog [How to use QThread correctly (part 1)?](https://www.haccks.com/posts/how-to-use-qthread-correctly-p1/). If you haven't read the first part then please read [part 1](https://www.haccks.com/posts/how-to-use-qthread-correctly-p1/) before reading this.</sub>

## What is a QThread and How it works?

### 3. Modify examples to abort the download

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

class WorkerThread(QThread)
    progress = Signal(int)

    def __init__(self, n):
        super().__init__()
        self.n = n
        self.is_cancel = False

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
<!-- <sub>[Code on github]()</sub> -->

Todo: Add a gif


While download is in progress and managed by worker thread, when we press `Cancel` button then it will emit it's `clicked()` signal in the main thread. And receiver of this signal is main thread itself, therefore the method `stop_download()` will execute in the main thread (`worker_thread` lives in the main thread and therefore main thread will handle received signal and execute `stop_download()` slot). Both of these jobs is done in parallel (not really!).   

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

Unlike subclass approach, slot `stop_download()` will execute in the worker thread because `worker` lives in secondary thread (after calling its `moveToThread()` method) and this thread will handle its slot. 

Since `do_work()` is a blocking task, till this job finishes, the local event loop will wait to get back the control and meanwhile all the incoming signals will be queued in the event-queue of the worker thread. The slot `stop_download()` is invoked only after control returns to the event loop of the thread `worker_thread` is managing.

We can fix this problem by directly invoking `stop_download()` from main thread. 

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

<!-- <sub>[Code on github]()</sub> -->

Todo: Add a gif

Note that `cancel_btn.clicked` signal is not connected to `worker.stop_download()` slot rather it is connected to `cancel()` slot of `MainWindow`. Since `cancel()` is invoked in main thread and hence it will invoke `worker.stop_download()` as a normal method in main thread instead of worker thread.  

### 4. Using a non-GUI Qt class in secondary thread 

As I already mentioned [`QTimer`](https://doc.qt.io/qtforpython-6/PySide6/QtCore/QTimer.html) is a non-GUI class and it requires an event loop. Suppose we need to do something periodically in a secondary thread. Let's try `QThread` subclass approach. 

```py
import sys
from PySide6.QtWidgets import QApplication
from PySide6.QtCore import QThread, qDebug, QTimer, Slot


class WorkerThread(QThread):
    @Slot()
    def on_timeout(self):
        qDebug(f"on_timeout() called from thread: {self.currentThread()}")

    def do_work(self):
        qDebug(f"Worker thread: {self.currentThread()}")
        timer = QTimer()
        timer.timeout.connect(self.on_timeout)
        timer.start(1000)  # Start the timer and emit timeout() signal every second.
        self.exec()        # Start the event loop. Without this QTimer will never emit timeout() signal

    def run(self):
        self.do_work()


if __name__ == '__main__':
    app = QApplication(sys.argv)
    worker_thread = WorkerThread()
    worker_thread.start()
    qDebug(f"Main thread: {worker_thread.currentThread()}")
    app.exec()

```
{: .file=qtimer-subclass.py}

This will produce the output like 

```
Main thread: <PySide6.QtCore.QThread(0x6000004cf8c0) at 0x10de67440>
Worker thread: <__main__.WorkerThread(0x6000006c5ea0) at 0x10de67340>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000004cf8c0) at 0x10de67680>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000004cf8c0) at 0x10de675c0>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000004cf8c0) at 0x10de67680>
...
```
Notice the id of main thread is `0x6000004cf8c0` and `on_timeout()` is running in this thread. But, we expected to run `on_timeout()` in secondary thread. Why this behavior?   
As mentioned earlier, `worker_thread` lives in main thread and queued signals are handled in receiver's thread. So, secondary thread emits `timer.timeout`signal and it is received in main thread. Therefore, Slot `worker_thread.on_timeout()` is handled/executed by the main thread signal handler, not by secondary thread.

Using worker-object approach we can execute the periodic task in secondary thread.  

```py
import sys
from PySide6.QtWidgets import QApplication
from PySide6.QtCore import QThread, qDebug, QTimer, QObject, Slot


class Worker(QObject):
    @Slot()
    def on_timeout(self):
        qDebug(f"on_timeout() called from thread: "
               f"{self.thread()}")

    @Slot()
    def do_work(self):
        qDebug(f"Worker thread: {self.thread()}")
        self.timer = QTimer()
        self.timer.timeout.connect(self.on_timeout)
        self.timer.start(1000)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    worker = Worker()
    worker_thread = QThread()
    worker.moveToThread(worker_thread)
    worker_thread.started.connect(worker.do_work)
    worker_thread.start()

    qDebug(f"Main thread: {worker_thread.thread()}")
    app.exec()

```
{: .file=qtimer-worker-object.py}
And the output  

```
Main thread: <PySide6.QtCore.QThread(0x600002bd0300) at 0x10de12940>
Worker thread: <PySide6.QtCore.QThread(0x6000029fb560) at 0x10de12740>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000029fb560) at 0x10de12740>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000029fb560) at 0x10de12740>
on_timeout() called from thread: <PySide6.QtCore.QThread(0x6000029fb560) at 0x10de12740>
...
```
As you can see `worker.on_timeout()` is running in secondary thread.


## Conclusion

For this particular app both approach has been used. But as we can see, for this app

1. No event loop is needed
2. No signals/slots need to be handled inside the secondary thread (we are emitting signals though)

In such case where you want to perform some expensive operation in another thread, where the thread does not need to receive any signals or events, subclass `QThread`.

**When to use worker-object approach?**   

1. *When you need an event loop in `QThread`*. Certain non-GUI classes (such as `QTimer`, `QTcpSocket`, and `QProcess`) requires the presence of event loop. If you are using instances of these classes in your thread then you will have to use worker-object approach.  

2. *If you have to handle signals/slots in the secondary thread*.
