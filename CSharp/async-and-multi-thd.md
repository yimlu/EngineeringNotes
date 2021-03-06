## 异步与多线程 ##

异步与多线程这个问题，应该在不同的层面来看。
从计算机科学的角度，异步和多线程没有什么关系，异步同步是时序的问题，多线程是操作系统的概念。

异步有一些是用多线程来实现，一个概念DMA(直接内存访问)，有些硬件可以直接进行I/O,I/O结束后才通知CPU,因此有些I/O是可以进行异步操作的，无需CPU时间。暂时认为需要CPU计算参与的一些异步是用线程来实现的。

从.NET开发时的角度来看，异步是在调用某个方法后，不等待该方法返回即继续执行，那么那个方法就是异步方法，从暂时的实验来看，异步方法在后台用了thread pool,即也是用了多线程来实现（实验环境为Ubuntu + Mono）。

Winform的开发中，需要用到Invoke，BeginInvoke这样的异步调用。

Winform的窗体是由主线程创建的，而且对于控件的UI更新只能在主线程中执行。如果在其他线程中想要更新UI,必须用Message.这里涉及到了windows编程中的消息机制以及两个Api, SendMessage, PostMessage.

SendMessage方法会等待消息被处理后再返回，PostMessage则相反，发送消息后就立即返回了。

Winform中，只允许创建该控件的线程对该控件进行更新，原因是这样：控件是在主线程中创建的，消息泵也是在主线程中运行（有一点例外，Modal窗体有单独消息泵），主线程用SendMessage来更新控件，如果允许其他线程也进行SendMessage操作，那么会有不可预测的更新情况，最坏情况是有可能造成死锁（这里需要重新细读codeproject上的那篇文章）。因此，Winform中为了能够在非主线程中更新UI,提供了Invoke, BeginInvoke的方法，这些方法调用时使用的是PostMessage，那么非主线程只是发出了消息，最后还是主线程的消息泵去处理了这个消息，这样就不会有问题。

消息泵的处理顺序是先处理Send来的消息，然后再处理Post来的消息。