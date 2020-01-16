---
title: "缓存队列"
date: 2020-01-15T10:38:08+08:00
draft: false
description: "缓存队列"
tags: 
 - C++
 - 缓存队列
categories: 
 - C/C++
---









使用缓存队列接收消息，避免高并发时消息漏接丢失的情况，提供一种消息缓冲的机制，还可以将消息进行统一分发处理，方便管理。

 <!--more-->

缓存队列是用来存放消息的数据结构，使用队列可以保证消息按接收先后顺序进行处理，对消息并发做个缓冲。在大型系统中，经常会用到消息队列来缓解系统压力或进行系统解耦，这里考虑到只是一个小型服务采用了一个简单的缓存队列来解决一下可能的发生并发问题，和业务分发管理。

C++实现的缓存队列类模板，包含：

* 消息存放的数据结构

* 线程循环读取消息处理

* 接收消息的函数

* 互斥锁



传入消息类型Message（如下为包含一个int类型的消息类别，一个string类型的消息数据），保存在一个set集合里面。

```c++
#include <set>
#include <stdint.h>
#include <string>
#include <pthread.h>

template <class __R>
class MessageQueue {
public:
	struct Message;
	 //函数指针，用于消息处理函数，调用时实现
	typedef __R(*Handler)(Message& msg,void* opaque); 
	//构造函数，设置好回调的消息处理函数
	MessageQueue(Handler handler, void* opaque) : _handler(handler), _opaque(opaque) 
	{
		//初始化一个互斥锁
		MESSAGE_MUTEX = new pthread_mutex_t();
		pthread_mutex_init(MESSAGE_MUTEX, NULL);
	}
	// 析构函数
	~MessageQueue()
	{
		//释放锁
		pthread_mutex_destroy(MESSAGE_MUTEX);
		delete MESSAGE_MUTEX;
	}
	// 传入消息到队列中，加锁防止冲突
	void AddTask(int type, string &msg)
	{
		pthread_mutex_lock(MESSAGE_MUTEX);
		_messages.insert(Message(type,msg));
		pthread_mutex_unlock(MESSAGE_MUTEX);
	}
	// 消息处理函数，调用handler进行消费，调用模板的地方，启动一个线程专门来处理消息
	void ProcessMessages()
    {
		pthread_mutex_lock(MESSAGE_MUTEX);
		for (set<Message>::iterator msg=_messages.begin(); msg != _messages.end();) {
			// 遍历处理任务
			int result = _handler((Message&)*msg, _opaque);
			// 处理完删除任务
			_messages.erase(msg);
		}
		pthread_mutex_unlock(MESSAGE_MUTEX);
    }
private:
	Handler           _handler;
	void*             _opaque;
	pthread_mutex_t*  _msg_mutex;
	set<Message>      _messages;
	
public:
	// 定义消息结构体
	struct Message{
		int type;
		string data;
        Message(int type, string data){
        	this->type = type;
        	this->data = data;
        }
        Message():Message(0){}
    private:
    	friend class MessageQueue<__R>;
	}；
};
```

使用类的声明示例：

```c++
class MessageHandler
{
public:
    // 构造函数使用handler，实例化消息缓存队列_queue，启动线程循环处理消息任务
    MessageHandler();   
    ~MessageHandler();
    // 线程处理函数，调用模板中的处理函数
    static void *handle_thread(void *);
    // 根据类别实现消息处理
    int HandlerMessage(int type, string &data);
    // 调用HandlerMessage，传入队列模板的函数实现
    inline static int handler_msg(MessageQueue<int>::Message& msg,void* opaque);
    // 添加任务到队列
    void add_task(int type, string &data);
private:
    MessageQueue<int>  _queue;
}；
```

函数定义如下：

```c++
MessageHandler::MessageHandler() :_queue(handler,this)
{
    _thread = new pthread_t();
    _thread_attr = new pthread_attr_t();

    pthread_attr_init(_thread_attr);
    pthread_create(_thread, _thread_attr, handle_thread, (void *)this);
}
MessageHandler::~MessageHandler()
{
    void *pret=NULL;
    pthread_join(*_thread, &pret);
    pthread_attr_destroy(_thread_attr);
    
    delete _thread;
    delete _thread_attr;
}
int MessageHandler::HandlerMessage(int type, string &data)
{
	switch (type)
    {
        case xx:
            break;
        //...
    }
    return 0;
}
inline int MessageHandler::handler_msg(MessageQueue<int>::Message& msg,void* opaque)
{
    //实际为this指针
    MessageHandler* handler = (MessageHandler*)opaque;
    return handler->HandlerMessage(msg.type, msg.data);
}
void MessageHandler::Add_Task(int type, string &data)
{
    _queue.AddTask(type, data);
}
void *MessageHandler::handlethread(void *opaque)
{
    // 静态函数内，使用类对象的this指针访问类中的成员
    MessageHandler* handler =(MessageHandler*)opaque;
	while(true)
    {
        handler->_queue.ProcessMessages();
        usleep(16*1000);
    }
    return NULL;
}
```



