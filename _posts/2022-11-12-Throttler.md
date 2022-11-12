---
layout:     post
title:      "Argo Workflow中的sync工具Throttler源码分析"
subtitle:   "Throttler"
date:       2022-11-12 11:00:00
author:     "Reverie"
catalog: false
header-style: text
tags: 
- golang
- cloud
---

# Throtter

> Throttler allows the controller to limit number of items it is processing in parallel.
Items are processed in priority order, and one processing starts, other items (including higher-priority items) will be kept pending until the processing is complete.
Implementations should be idempotent.

总的来说比较简单。
Throttler是一个接口，定义了以下四个方法
```golang
type Throttler interface {
	Init(i []interface{}) error
    // Add添加一个对象，以及它的优先级
	Add(key Key, priority int32, creationTime time.Time)
	// Admit返回bool值，代表这个对象是否该被处理
	Admit(key Key) bool
	// Remove 告知throttler不再需要处理这个对象
	Remove(key Key)
}
```

再来看看具体实现：
```golang
type Key = string
type QueueFunc func(Key)

type item struct {
	key          string
	creationTime time.Time
	priority     int32
	index        int
}

type priorityQueue struct {
	items     []*item
	itemByKey map[string]*item
}

type throttler struct {
	queue       QueueFunc
	bucketFunc  BucketFunc
	inProgress  map[BucketKey]map[Key]bool
	pending     map[BucketKey]*priorityQueue
	lock        *sync.Mutex
	parallelism int
}
```

## BucketFunc
Bucket Func主要有两个实现：SingleBucket返回空字符串，NamespaceBucket返回这个key的namespace
```golang
type BucketKey = string
type BucketFunc func(Key) BucketKey

var SingleBucket BucketFunc = func(key Key) BucketKey { return "" }
var NamespaceBucket BucketFunc = func(key Key) BucketKey {
	namespace, _, _ := cache.SplitMetaNamespaceKey(key)
	return namespace
}

type bucket map[Key]bool
type buckets map[BucketKey]bucket
```

##Init
```golang
func (t *throttler) Init(wfs []wfv1.Workflow) error {
    // 获取锁
	t.lock.Lock()
	defer t.lock.Unlock()
	if t.parallelism == 0 {
		return nil
	}

	for _, wf := range wfs {
        // 获取对象的key
		key, err := cache.MetaNamespaceKeyFunc(&wf)
		if err != nil {
			return err
		}
        // 如果对象是running状态，将此对象放入inProgress这个map里
		if wf.Status.Phase == wfv1.WorkflowRunning {
            // 获取bucket key
			bucketKey := t.bucketFunc(key)
			if _, ok := t.inProgress[bucketKey]; !ok {
				t.inProgress[bucketKey] = make(bucket)
			}
			t.inProgress[bucketKey][key] = true
		}
	}
	return nil
}
```

## Add
Add方法添加一个新的对象，
```golang
func (t *throttler) Add(key Key, priority int32, creationTime time.Time) {
    // 获取锁
	t.lock.Lock()
	defer t.lock.Unlock()
	if t.parallelism == 0 {
		return
	}
    // 获取bucket key
	bucketKey := t.bucketFunc(key)
    // 将bucket key 在pending map里初始化
	if _, ok := t.pending[bucketKey]; !ok {
		t.pending[bucketKey] = &priorityQueue{itemByKey: make(map[string]*item)}
	}
    // 将key放入pending map里的优先队列
	t.pending[bucketKey].add(key, priority, creationTime)
    // pending -> process
	t.queueThrottled(bucketKey)
}

func (t *throttler) queueThrottled(bucketKey BucketKey) {
	if _, ok := t.inProgress[bucketKey]; !ok {
		t.inProgress[bucketKey] = make(bucket)
	}
	inProgress := t.inProgress[bucketKey]
	pending, ok := t.pending[bucketKey]
    // 如果并行数大于当前process的数量，说明可以从pending里拿出一对象来，放入process
	for ok && pending.Len() > 0 && t.parallelism > len(inProgress) {
        // 根据优先级来出队
		key := pending.pop().key
		inProgress[key] = true
		t.queue(key)
	}
}
```

## Admit
Admit返回bool值，代表这个对象是否可以被处理
```golang
func (t *throttler) Admit(key Key) bool {
    // 获取锁
	t.lock.Lock()
	defer t.lock.Unlock()
	if t.parallelism == 0 {
		return true
	}
	bucketKey := t.bucketFunc(key)
    // 如果当前key是在process map里，说明可以运行
	if x, ok := t.inProgress[bucketKey]; ok && x[key] {
		return true
	}
	t.queueThrottled(bucketKey)
	return false
}
```

## Remove
Remove 从throttler移除掉这个对象
```golang
func (t *throttler) Remove(key Key) {
	t.lock.Lock()
	defer t.lock.Unlock()
	bucketKey := t.bucketFunc(key)
	if x, ok := t.inProgress[bucketKey]; ok {
		delete(x, key)
	}
	if x, ok := t.pending[bucketKey]; ok {
		x.remove(key)
	}
	t.queueThrottled(bucketKey)
}
```
