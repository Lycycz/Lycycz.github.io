---
layout: post
title:  "SDL_OpenAudioDevice多线程创建出相同AudioDeviceId!"
date:   2021-03-23 20:45:25 +0800
tags: [jekyll]
categories: jekyll
---
## 调用函数
{% highlight c++ %}
SDL_AudioDeviceID SDL_OpenAudioDevice(const char*          device,
                                      int                  iscapture,
                                      const SDL_AudioSpec* desired,
                                      SDL_AudioSpec*       obtained,
                                      int                  allowed_changes)
{% endhighlight %}
返回值`SDL_AudioDeviceID`,这个id不代表具体硬件设备，应该叫Session,即一个会话。

网上的帖子详细解析: 
[源码解析][SDL_OpenAudioDevice]

## 多线程出现的问题

开启两个线程，同时调用函数，会导致返回会话id相同的情况或创建失败的情况。若对象封装后，在创建过程中出现相同id，在一个对象释放时close会导致崩溃的情况。 


### 复现问题代码如下

{% highlight c++ %}
std::thread t1([](){
  int device_id = SDL_AudioDeviceID(NULL, 0, &wanted_spec, NULL, 0);
});
std::thread t2([](){
  int device_id = SDL_AudioDeviceID(NULL, 0, &wanted_spec, NULL, 0);
});
t1.join();
t2.join();
{% endhighlight %}

两个线程同时开始，一般创建失败

{% highlight c++ %}
std::thread t1([](){
  int device_id = SDL_AudioDeviceID(NULL, 0, &wanted_spec, NULL, 0);
});
std::thread t2([](){
  Sleep(100);
  int device_id = SDL_AudioDeviceID(NULL, 0, &wanted_spec, NULL, 0);
});
t1.join();
t2.join();
{% endhighlight %}

两个线程间隔300ms以下，一般创建出相同id。 
间隔500ms以上，创建出两个id。

### 暂时的解决方法

暂时的解决办法是，错开创建时间。

[SDL_OpenAudioDevice]: http://www.libsdl.cn/bbs/forum.php?mod=viewthread&tid=90]
