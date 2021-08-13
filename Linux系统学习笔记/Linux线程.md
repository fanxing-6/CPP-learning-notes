# Linux线程

## 互斥量

这是一个死锁情况的演示

主线程先获取互斥量`mutex_a`,然后休息五秒钟,在这段时间内线程获取互斥量`mutex_b`,然后主线程还要获取互斥量`mutex_b`但是在线程那里,线程等待互斥量`mutex_b`这样就发生了死锁

```cpp
#include <pthread.h>
#include <unistd.h>
#include <stdio.h>
#include <iostream>

using namespace std;
int a = 0;
int b =0;

pthread_mutex_t mutex_a;
pthread_mutex_t  mutex_b;
void* another(void* arg)
{
    pthread_mutex_lock(&mutex_b);
    cout<<"in child thread , got mutex b ,wainting for mutex a"<<endl;
    sleep(5);
    ++b;
    pthread_mutex_lock(&mutex_a);
    b += a++;
    pthread_mutex_unlock(&mutex_a);
    pthread_mutex_unlock(&mutex_b);
    pthread_exit(nullptr);




}
int main()
{
    pthread_t id;
    pthread_mutex_init(&mutex_a, nullptr);
    pthread_mutex_init(&mutex_b, nullptr);
    pthread_create(&id, nullptr,another, nullptr);

    pthread_mutex_lock(&mutex_a);
    printf("in parent thread , got mutex a, waiting for mutex\n");
    sleep(5);
    ++a;
    pthread_mutex_lock(&mutex_b);
    a += b++;
    pthread_mutex_unlock(&mutex_b);
    pthread_mutex_unlock(&mutex_a);
    
    pthread_join(id, nullptr);
    pthread_mutex_destroy(&mutex_a);
    pthread_mutex_destroy(&mutex_b);

}
```



## 线程同步机制包装

```cpp
//
// Created by root on 7/26/21.
//

#ifndef UNTITLED1_LOCKER_H
#define UNTITLED1_LOCKER_H

#include <exception>
#include <pthread.h>
#include <semaphore.h>

class sem 
{
private:
    sem_t m_sem;
public:
    sem()
    {
        if(sem_init(&m_sem,0,0) != 0)
        {
            throw  std::exception();
        }
        
    }
    ~sem()
    {
        sem_destroy(&m_sem);
    }
    bool wait()
    {
        return sem_wait(&m_sem) == 0;
    }
    bool post()
    {
        return sem_post(&m_sem) == 0;
    }
};

class locker
{
private:
    pthread_mutex_t  m_mutex;
public:
    locker()
    {
        if(pthread_mutex_init(&m_mutex, nullptr) != 0)
        {
            throw std::exception();
        }
    }
    ~locker()
    {
        pthread_mutex_destroy(&m_mutex);
    }
    bool lock()
    {
        return pthread_mutex_lock(&m_mutex) == 0;
    }
    bool unlock()
    {
        return pthread_mutex_unlock(&m_mutex) == 0;
    }
    
};

class cond
{
private:
    pthread_mutex_t m_mutex;
    pthread_cond_t m_cond;
public:
    cond()
    {
        if (pthread_mutex_init(&m_mutex, nullptr) != 0)
        {
            throw std::exception();
        }
        if(pthread_cond_init(&m_cond, nullptr) != 0)
        {
            pthread_mutex_destroy(&m_mutex);
            throw std::exception();
        }
        
    }
    ~cond()
    {
        pthread_mutex_destroy(&m_mutex);
        pthread_cond_destroy(&m_cond);
    }
    bool wait()
    {
        int ret = 0;
        pthread_mutex_lock((&m_mutex));
        ret = pthread_cond_wait(&m_cond,&m_mutex);
        pthread_mutex_unlock(&m_mutex);
        return ret == 0;
    }
    bool signal()
    {
        return pthread_cond_signal(&m_cond) == 0;
    }
    
};

#endif //UNTITLED1_LOCKER_H

```

