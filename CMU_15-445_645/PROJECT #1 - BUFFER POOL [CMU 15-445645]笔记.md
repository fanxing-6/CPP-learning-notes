# PROJECT #1 - BUFFER POOL 15-445/645笔记

![image-20220628215352423](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206282153843.png) 

   因为在主存中储存所有块是不可能的，我们需要管理主存中用于存储块的可用空间的分配 。**缓冲区**是主存中用于存储磁盘数据块拷贝的那部分。

**缓冲区管理器**

​	当数据库系统中的程序需要磁盘上的块时，它向缓冲区管理器发出请求（即调用)。如果这个块已经在缓冲区中，则缓冲区管理器将这个块在主存中的地址传给请求者。如果这个块不在缓冲区中，则缓冲区管理器首先在缓冲区中为这个块分配空间，如果需要，可能会把一些其他块移出主存，为这个新块腾出空间。被移出的块仅当它在最近一次写回磁盘后被修改过才被写回磁盘。然后，缓冲区管理器把被请求的块从磁盘读入缓冲区，并将这个块在主存中的地址传给请求者。缓冲区管理器的内部动作对发出磁盘块请求的程序是透明的。

​	如果熟悉操作系统的概念，你会发现缓冲区管理器看起来和大多数操作系统中的虚拟存储管理器没有什么不同。它们的一点区别是数据库的规模会比机器的硬件地址空间要大，因此存储器地址不足以对所有磁盘块进行寻址。此外，为了更好地为数据库系统服务，缓冲区管理器必须使用比典型的虚拟存储管理方案更加复杂的技术。



## TASK #1 - LRU REPLACEMENT POLICY   缓冲区替换策略

### 实现**LRUReplacer**

​		**LRUReplacer**的最大frame数与缓冲池的大小相同,因为它包含`BufferPoolManager`所有帧的占位，但并不是所有的帧（的占位）都在**LRUReplacer**，**LRUReplacer**初始化时是空的，只有新的`unpin`的frame会在**LRUReplacer**

- `Victim(frame_id_t*): `使用`LRU`方案(最近最少使用方案), 及最近最少访问的块被写回磁盘,并从缓冲区移走.  将其`frame`存储在参数中(出参), 如果`Replacer`为空, 返回`false`

- `Pin(frame_id_t): `将`frame`固定到`BufferPoolManager`中后, 调用此方法, 他应该从`LRUReplacer`移除相应`frame`

  **如果调用Pin就说明正在读取,所以不能被移除, 所以**

  ```cpp
  lru_cache.erase(lru_map[frame_id]);
  lru_map.erase(frame_id);
  ```

- `Unpin(frame_id_t)` : 当某一`frame`的`pin_count`变成 0 时, 调用 `Unpin` , 这个函数应该向`LRUReplacer`添加`frame`

  **当进程完成数据读取时, 将执行`Unpin`操作, 允许改块移除, 所以需要添加到 `LRUReplacer` , 多个进程能从缓冲区的一个块中读取数据, 要求每个进程在访问数据前执行`pin`操作,直到完成时执行`unpin`操作, 所有进程执行`Unpin`操作之后, 这个`frame`才能被移除, 所以需要使用`pin_count`进行计数, 保证实现上述操作**

- `Size()`: 返回`LRUReplacer`的`size`

**由于必须确保线程安全**, 所以使用锁, 因为`unpin`调用`size`, 按照我之前的想法, 不确定`Size()`是否会在其他位置调用, 所以这两个函数都要上锁, 如果使用`std::mutex`会导致死锁, 所以使用了递归锁`  std::recursive_mutex`, 解决这个问题 .

`std::recursive_mutex`: **允许同一线程对某互斥量的同一实例多次加锁** .有时在某些操作过程中，公有函数需要调用另一公有函数。在这种情况下，后者将同样试图锁住互斥，如果采用`std::mutex`便会导致未定义行为。用递归互斥代替普通互斥解决。



*lru_replacer.h*

```cpp

#pragma once

#include <list>
#include <mutex>  // NOLINT
#include <unordered_map>
#include <vector>

#include "buffer/replacer.h"
#include "common/config.h"

namespace bustub {

/**
 * LRUReplacer implements the Least Recently Used replacement policy.
 */
class LRUReplacer : public Replacer {
 public:
  /**
   * Create a new LRUReplacer.
   * @param num_pages the maximum number of pages the LRUReplacer will be required to store
   */
  explicit LRUReplacer(size_t num_pages);

  /**
   * Destroys the LRUReplacer.
   */
  ~LRUReplacer() override;

  bool Victim(frame_id_t *frame_id) override;

  void Pin(frame_id_t frame_id) override;

  void Unpin(frame_id_t frame_id) override;
  size_t Size() override;

 private:

  std::recursive_mutex lru_lock; // 递归锁
  std::list<frame_id_t> lru_cache;
  std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> lru_map;
  size_t capacity;
};

}  // namespace bustub

```

*lru_replacer.cpp*

```cpp


#include "buffer/lru_replacer.h"

namespace bustub {

LRUReplacer::LRUReplacer(size_t num_pages):capacity(num_pages) {};

LRUReplacer::~LRUReplacer() = default;

bool LRUReplacer::Victim(frame_id_t *frame_id) {
  std::lock_guard latch(lru_lock);

  if (lru_map.empty()) return false;

  frame_id_t frame = lru_cache.back(); // 把药牺牲的页返回, 并且从cache删除
  lru_cache.pop_back();
  lru_map.erase(frame);
  *frame_id = frame;

  return true;
}

void LRUReplacer::Pin(frame_id_t frame_id) {
  std::lock_guard latch(lru_lock);

  if (lru_map.count(frame_id) != 0) {
    lru_cache.erase(lru_map[frame_id]); // 从cache中删除, 这样就不会被删掉了, 相当于钉住
    lru_map.erase(frame_id);
  }
}

void LRUReplacer::Unpin(frame_id_t frame_id) { // 添加到cache中, frame可以被删除了

    std::lock_guard latch(lru_lock);
    if (lru_map.count(frame_id) != 0) return;


  while (this->Size() >= capacity) { // 正常来说不会出现这种情况, 写完后参考其他人的,发现很多人都这么写,就加上了...

    frame_id_t del_frame = lru_cache.front();
    lru_cache.pop_front();
    lru_map.erase(del_frame);
  }

  lru_cache.push_front(frame_id);
  lru_map[frame_id] = lru_cache.begin();
}

size_t LRUReplacer::Size() {
  std::lock_guard latch(lru_lock);
  return lru_cache.size();
}


}  // namespace bustub

```

![image-20220628214118254](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206282141356.png)

## TASK #2 - BUFFER POOL MANAGER INSTANCE 缓冲池管理器实例

​		接下来，中实现缓冲池管理器（`BufferPoolManagerInstance`）。`BufferPoolManagerInstance` 负责从 `DiskManager` 获取数据库页并将其存储在内存中。`BufferPoolManagerInstance`显式指示将脏页写入磁盘时，或者当它需要逐出页以便为新页腾出空间时，将脏页写出到磁盘。

​		系统中的所有内存中页都由 `Page` 对象表示。`BufferPoolManagerInstance` 不需要了解这些页面的内容。但是，作为系统开发人员，您必须了解 `Page` 对象只是缓冲池中内存的容器，因此并不特定于唯一页。也就是说，每个 `Page` 对象都包含一个内存块，`DiskManager` 将使用该内存块作为位置来复制它从磁盘读取**的物理页**的内容。`BufferPoolManagerInstance` 将重用同一 `Page` 对象，以便在数据来回移动到磁盘时存储数据。这意味着在系统的整个生命周期中，同一 `Page` 对象可能包含不同的物理页面。`Page`对象的标识器（`page_id`）跟踪它包含的物理页面;如果 `Page` 对象不包含物理页，则必须将其`page_id`设置为 `INVALID_PAGE_ID`.

每个 `Page` 对象还维护一个计数器，用于表示已“固定”该页面的线程数。不允许`BufferPoolManagerInstance`释放已固定`Page`。每个 `Page` 对象还会跟踪它是否脏。您的工作是记录页面在取消固定之前是否被修改过。`BufferPoolManagerInstance` 必须将脏 `Page` 的内容写回磁盘，然后才能重用该对象。

`BufferPoolManagerInstance` 实现将使用您在此分配的前面步骤中创建的 `LRUReplacer` 类。它将使用 `LRUReplacer` 来跟踪何时访问 `Page` 对象，以便它可以决定在必须释放帧以腾出空间以从磁盘复制新的物理页面时要y移除哪个对象。

------

**首先, 先说明几个概念 : **

- `page`:  数据库需要发文件分成很多小部分,所以文件是由一堆`page`构成的, `page` 也就是读取数据的最小单元, 本实验中每个`page`的大小为`4k`

- `frame`: 之内存中的空间, 和`page`的大小一样, 用来放置从磁盘读取的` page`, 一个`page`可以放在任何一个空的`frame`中,

  当`BUFFER POOL MANAGER`需要`frame`存储新的`page`的时候, 首先会查询`free_list`(存储空闲的`frame`)是否有空闲的`frame`, 如果有,  就将`page`存入这个空闲的`frame` ,如果没有空闲的`frame` ,  就会从`Replacer` 中找到一个`unpin`的 `frame`, 如果是脏页, 写回磁盘 , 否则, 将新的`page` 读取到这个`frame`中, 如果`Replacer`也无法返回一个可被替换 ( 牺牲 ) 的`frame`, 直接返回错误

具体实现, 可以看注释和代码



*buffer_pool_manager_instance.cpp*

```cpp
#include "buffer/parallel_buffer_pool_manager.h"

namespace bustub {

ParallelBufferPoolManager::ParallelBufferPoolManager(size_t num_instances, size_t pool_size, DiskManager *disk_manager,
                                                     LogManager *log_manager)
    : num_instances_(num_instances), pool_size_(pool_size), start_index_(0) {
  // Allocate and create individual BufferPoolManagerInstances
  for (size_t i = 0; i < num_instances_; i++) {
    parallel_mannager_instances.emplace_back(
        new BufferPoolManagerInstance(pool_size_, num_instances_, i, disk_manager, log_manager));
  }
}

// Update constructor to destruct all BufferPoolManagerInstances and deallocate any associated memory
ParallelBufferPoolManager::~ParallelBufferPoolManager() {
  for (auto instance_pointer : parallel_mannager_instances) {
    delete instance_pointer;
  }
};

size_t ParallelBufferPoolManager::GetPoolSize() {
  // Get size of all BufferPoolManagerInstances
  size_t all_buffer_size{0};
  for (auto instance_pointer : parallel_mannager_instances) {
    all_buffer_size += instance_pointer->GetPoolSize();
  }
  return all_buffer_size;
}

BufferPoolManager *ParallelBufferPoolManager::GetBufferPoolManager(page_id_t page_id) {
  // Get BufferPoolManager responsible for handling given page id. You can use this method in your other methods.
  size_t index = page_id % num_instances_;
  return parallel_mannager_instances[index];
}

Page *ParallelBufferPoolManager::FetchPgImp(page_id_t page_id) {
  // Fetch page for page_id from responsible BufferPoolManagerInstance

  return GetBufferPoolManager(page_id)->FetchPage(page_id);
}

bool ParallelBufferPoolManager::UnpinPgImp(page_id_t page_id, bool is_dirty) {
  // Unpin page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->UnpinPage(page_id, is_dirty);
}

bool ParallelBufferPoolManager::FlushPgImp(page_id_t page_id) {
  // Flush page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->FlushPage(page_id);
}

Page *ParallelBufferPoolManager::NewPgImp(page_id_t *page_id) {
  // create new page. We will request page allocation in a round robin manner from the underlying
  // BufferPoolManagerInstances
  // 1.   From a starting index of the BPMIs, call NewPageImpl until either 1) success and return 2) looped around to
  // starting index and return nullptr
  Page *p{nullptr};
  size_t index{start_index_ % num_instances_};
  size_t start(index);
  start_index_++;

  while (true) {
    p = parallel_mannager_instances[index]->NewPage(page_id);
    if (p != nullptr) return p;
    index = (index + 1) % num_instances_;
    if (index == start) return nullptr;
  }

  // 2.   Bump the starting index (mod number of instances) to start search at a different BPMI each time this function
  // is called
  return nullptr;
}

bool ParallelBufferPoolManager::DeletePgImp(page_id_t page_id) {
  // Delete page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->DeletePage(page_id);
}

void ParallelBufferPoolManager::FlushAllPgsImp() {
  // flush all pages from all BufferPoolManagerInstances
  for (auto instance_pointer : parallel_mannager_instances) {
    instance_pointer->FlushAllPages();
  }
}

}  // namespace bustub

```

![image-20220628214938795](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206282149839.png)

## TASK #3 - PARALLEL BUFFER POOL MANAGER 并行任务缓冲器

正如您在上一个任务中可能注意到的那样，单个缓冲池管理器实例需要采用闩锁才能实现线程安全。这可能会导致很多争用，因为每个线程在与缓冲池交互时都会在单个闩锁上争用。一种潜在的解决方案是在系统中设置多个缓冲池，每个缓冲池都有自己的闩锁。

`ParallelBufferPoolManager` 是一个包含多个 `BufferPoolManagerInstance`s 的类。对于每个操作，`ParallelBufferPoolManager` 选择一个 `BufferPoolManagerInstance` 并委派到该实例。

我们使用给定的页面 ID 来确定要使用的特定 `BufferPoolManagerInstance`。如果我们有许多 `BufferPoolManagerInstance`s `num_instances`那么我们需要某种方法将给定的页面 ID 映射到 [0， num_instances） 范围内的数字。对于此项目，我们将使用模运算符，`page_id mod num_instances`将给定page_id映射到正确的范围。

当 `ParallelBufferPoolManager` 首次实例化时，它的起始索引应为 0。每次创建新页面时，您都将尝试每个 `BufferPoolManagerInstance`，从起始索引开始，直到一个成功。然后将起始索引增加 1。

请确保在创建单个 `BufferPoolManagerInstance`时，使用采用`uint32_t num_instances`和`uint32_t instance_index`的构造函数，以便正确创建页面 ID。

*paraller_buffer_pool_manager.h*

```
  std::vector<BufferPoolManagerInstance*> parallel_mannager_instances;
  size_t num_instances_;
  size_t pool_size_;
  size_t start_index_;
```

*paraller_buffer_pool_manager.cpp*

```cpp
#include "buffer/parallel_buffer_pool_manager.h"

namespace bustub {

ParallelBufferPoolManager::ParallelBufferPoolManager(size_t num_instances, size_t pool_size, DiskManager *disk_manager,
                                                     LogManager *log_manager)
    : num_instances_(num_instances), pool_size_(pool_size), start_index_(0) {
  // Allocate and create individual BufferPoolManagerInstances
  for (size_t i = 0; i < num_instances_; i++) {
    parallel_mannager_instances.emplace_back(
        new BufferPoolManagerInstance(pool_size_, num_instances_, i, disk_manager, log_manager));
  }
}

// Update constructor to destruct all BufferPoolManagerInstances and deallocate any associated memory
ParallelBufferPoolManager::~ParallelBufferPoolManager() {
  for (auto instance_pointer : parallel_mannager_instances) {
    delete instance_pointer;
  }
};

size_t ParallelBufferPoolManager::GetPoolSize() {
  // Get size of all BufferPoolManagerInstances
  size_t all_buffer_size{0};
  for (auto instance_pointer : parallel_mannager_instances) {
    all_buffer_size += instance_pointer->GetPoolSize();
  }
  return all_buffer_size;
}

BufferPoolManager *ParallelBufferPoolManager::GetBufferPoolManager(page_id_t page_id) {
  // Get BufferPoolManager responsible for handling given page id. You can use this method in your other methods.
  size_t index = page_id % num_instances_;
  return parallel_mannager_instances[index];
}

Page *ParallelBufferPoolManager::FetchPgImp(page_id_t page_id) {
  // Fetch page for page_id from responsible BufferPoolManagerInstance

  return GetBufferPoolManager(page_id)->FetchPage(page_id);
}

bool ParallelBufferPoolManager::UnpinPgImp(page_id_t page_id, bool is_dirty) {
  // Unpin page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->UnpinPage(page_id, is_dirty);
}

bool ParallelBufferPoolManager::FlushPgImp(page_id_t page_id) {
  // Flush page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->FlushPage(page_id);
}

Page *ParallelBufferPoolManager::NewPgImp(page_id_t *page_id) {
  // create new page. We will request page allocation in a round robin manner from the underlying
  // BufferPoolManagerInstances
  // 1.   From a starting index of the BPMIs, call NewPageImpl until either 1) success and return 2) looped around to
  // starting index and return nullptr
  Page *p{nullptr};
  size_t index{start_index_ % num_instances_};
  size_t start(index);
  start_index_++;

  while (true) {
    p = parallel_mannager_instances[index]->NewPage(page_id);
    if (p != nullptr) return p;
    index = (index + 1) % num_instances_;
    if (index == start) return nullptr;
  }

  // 2.   Bump the starting index (mod number of instances) to start search at a different BPMI each time this function
  // is called
  return nullptr;
}

bool ParallelBufferPoolManager::DeletePgImp(page_id_t page_id) {
  // Delete page_id from responsible BufferPoolManagerInstance
  return GetBufferPoolManager(page_id)->DeletePage(page_id);
}

void ParallelBufferPoolManager::FlushAllPgsImp() {
  // flush all pages from all BufferPoolManagerInstances
  for (auto instance_pointer : parallel_mannager_instances) {
    instance_pointer->FlushAllPages();
  }
}

}  // namespace bustub

```

![image-20220628215542762](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206282155810.png)

写的时候要仔细, 否则就要痛苦的`debug`......
