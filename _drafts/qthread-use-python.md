---
title: How to use QThread correctly?
description: "Whether one should subclass QThread or use worker object and move it to the thread depends on the use case"
date: 2023-05-18 14:00:00 +0530
categories: [python]
tags: [python3, PySide6, Qt, QThread]     # TAG names should always be lowercase
render_with_liquid: false
---

## Prerequisite:
+ Basic python programming
+ Basic understanding of threads
+ Qt Framework
+ PySide6 (or PyQt6)

## Intro

When it comes to use `QThread`, internet is divided into two groups:   

1. Subclass `QThread` and reimplement `run()`
2. Use *worker objects* by moving them to the thread using `QObject().moveToThread()`

There is actually a long debate on why not to subclass `QThread` in an old blog [You're doing it wrong](https://www.qt.io/blog/2010/06/17/youre-doing-it-wrong) posted by **Bradley T. Hughes** (one of the core developer of Qt). The blog suggests using the 2<sup>nd</sup> approach.  
After almost 3 years one of his colleague  **Olivier Goffart** posted a counter blog [You were not doing so wrong](https://woboq.com/blog/qthread-you-were-not-doing-so-wrong.html). This caused lots of confusion among programmers deciding what approach to use when using `QThread`.

As of the newest release of Qt (Qt6 at the time of writing this blog), documentation gives example of both approach. I dig into deeper and decided to write this blog and explain how really `QThread` works and when to use either approach with example. I will use [`PySide6`](https://doc.qt.io/qtforpython-6/) for this blog and all of its example.  

To use `QThread` correctly one has to understand the basics of this object. Let's start with event loop...

## What is an vent Loop?

An event loop is a loop that listen for any events like: user input, network traffic, sensors, timers etc. and dispatches these events to the target objects. In Qt, event loop starts when object's `exec()` method is called. [`QApplication().exec()`](https://doc.qt.io/qt-6/qeventloop.html#exec) starts the main event loop and wait until `exit()` is called. 

>*"It is necessary to call this function to start event handling. The main event loop receives events from the window system and dispatches these to the application widgets."* -- [`exec()` doc](https://doc.qt.io/qt-6/qeventloop.html#exec)
{: .prompt-info}


## What is a QThread and How it works?

Quoting from the documentation:  
> *"A `QThread` object manages one thread of control within the program. `QThreads` begin executing in `run()`. By default, `run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread."*

Let's breakdown the above paragraph first,
1. *A `QThread` object manages one thread of control*: Means if same thread is used to do two different long running tasks at the same time then they won't be done in parallel. One task has to wait for other to finish.  
2. *`QThreads` begin executing in `run()`*: When `QThread().start()` is called then it eventually calls `run()` method of `QThread` and then this `run()` method starts the actual thread. So, basically **`QThread` is not actually a thread but a wrapper around a thread**.  
3. *`run()` starts the event loop by calling `exec()` and runs a Qt event loop inside the thread*: A local event loop starts within the thread and wait until `exit()` is called (more on this later).


## Examples
Let's create a simple toy app to demonstrate downloading a large file. This will have a simple progressbar and a push button. When user press the button download will start and at the same time progress bar will be updated showing the progress of the downloading file. (Add cancel download button later)

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
        self.timer = QTimer()

    def do_work(self):
        for i in range(1, self.n+1):
            time.sleep(1)
            self.progress.emit(i)

    def run(self):
        self.do_work()

class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.setGeometry(100, 100, 300, 50)
        self.setWindowTitle('QThread subclass 1')

        # setup widget
        self.widget = QWidget()
        layout = QVBoxLayout()
        self.widget.setLayout(layout)
        self.setCentralWidget(self.widget)

        self.progress_bar = QProgressBar(self)
        self.progress_bar.setValue(0)

        self.download_btn = QPushButton('Download', clicked=self.download)

        layout.addWidget(self.progress_bar)
        layout.addWidget(self.download_btn)

        self.show()

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

    def finished(self):
        QMessageBox.information(self, "Download status", "Download Complete!")
        self.download_btn.setEnabled(True)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MainWindow()
    app.exec()

```
<!-- {: .nolineno } -->
{: file="example-1" }
