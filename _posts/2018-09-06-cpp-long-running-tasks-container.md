---
layout: post
title: C++ long running task stored in the task pool
categories: [code]
tags: [c++, async, timeout]
---





This is an example of code where we use std::async to create a task in a sub thread and store long running tasks in a container, when the timeout has reached.







```
#include <iostream>
#include <future>
#include <vector>

int main()
{
    auto start = std::chrono::steady_clock::now();
    {
    std::cout << "Hello, Wandbox!" << std::endl;
    
    std::vector<std::future<int>> m_tasks;
    
    for (int i = 0; i!=5; ++i)
    {
         std::future<int> future = std::async(std::launch::async, [](int number_in)
        //m_tasks.push_back( std::async(std::launch::async, [](int number_in)
        {
            std::cout << "Started task, number_in = " << number_in << '\n';
            if (number_in % 2 == 0)
            {
                std::this_thread::sleep_for(std::chrono::milliseconds(5000));
            }
            else
            {
                std::this_thread::sleep_for(std::chrono::milliseconds(1000));
            }
            
            return number_in;
        }, i);
        
        std::future_status status(std::future_status::deferred);
        bool has_timed_out(false);
        auto future_start = std::chrono::steady_clock::now();
        do
        {
            
            auto future_diff = std::chrono::steady_clock::now() - future_start;
            if (std::chrono::duration_cast<std::chrono::milliseconds>(future_diff).count() > 1000)
            {
                has_timed_out = true;
                break;
            }
            
            status = future.wait_for(std::chrono::milliseconds(100));

            auto elapsed_ticks = std::chrono::steady_clock::now() - start;
            int elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(elapsed_ticks).count();
            
            switch(status)
            {
                case std::future_status::deferred: std::cout << elapsed << " deferred...\n";    break;
                case std::future_status::timeout:  std::cout << elapsed << " timeout...\n";      break;
                case std::future_status::ready:    std::cout << elapsed << " ready...\n";        break;
            }
        }
        while (status != std::future_status::ready && !has_timed_out);
        
        
        if (has_timed_out)
        {
            m_tasks.push_back(std::move(future));
        }
        //std::cout << "Asking future.get() = " << future.get() << std::endl;
        
    }
    
    
    for (auto &f : m_tasks)
    {
        std::cout << "Asking f.get() = " << f.get() << std::endl;
    }
    }
    auto end = std::chrono::steady_clock::now();
    std::cout << "Time taken = " << std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count() << " ms \n";
}
```
#cpp #async #timeout
