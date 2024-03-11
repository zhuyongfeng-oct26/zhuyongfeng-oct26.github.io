---
layout: post
title: 常见的服务限流实现方式
categories: project-design
description:
keywords: 项目设计
---

# 常见的服务限流实现方式

![](/images/posts/project_design/01.png)

# 什么是服务限流

当谈及服务限流时，我们可以将其比喻为一种策略，类似于道路上的交通管制。就像城市需要控制车流量以保持道路流畅一样，微服务在面临大量请求时也需要进行限流，以保护系统免受过载的风险。

**服务限流，是指通过控制请求的速率或次数来达到保护服务的目的。**

# 常用限流算法

常见的4种限流方式，以及它们的实现与使用。

## 计数器法

### 定义

计数器算法是限流算法里最简单也是最容易实现的一种算法。在单位时间内累加请求次数，当请求次数达到设定的阈值时，触发限流策略，当进入下一个单位时间时访问次数清零。

例如限制每10s内限制最多接收50个请求：

![](/images/posts/project_design/02.png)

**临界问题**：当在8-10秒和10-12秒内分别并发50，虽然没有超过阈值，但如果算8-12秒，则并发数达到了100，已经超过了原先定义的10秒内不超过50的并发量。

![](/images/posts/project_design/03.png)

### 实现

```php
class CounterLimiter {
    private $maxRequests;
    private $timeWindow;
    private $requestCounter;
    private $lastResetTime;

    public function __construct($maxRequests, $timeWindow) {
        $this->maxRequests = $maxRequests;
        $this->timeWindow = $timeWindow;
        $this->requestCounter = 0;
        $this->lastResetTime = time();
    }

    public function allowRequest() {
        $currentTime = time();
        $elapsedTime = $currentTime - $this->lastResetTime;

        // 检查是否需要重置计数器
        if ($elapsedTime > $this->timeWindow) {
            $this->requestCounter = 0;
            $this->lastResetTime = $currentTime;
        }

        // 检查是否超过允许的最大请求数
        if ($this->requestCounter < $this->maxRequests) {
            $this->requestCounter++;
            return true;
        } else {
            return false;
        }
    }
}

// 示例用法
$maxRequests = 5;
$timeWindow = 60; // 60秒内最多处理5个请求
$limiter = new CounterLimiter($maxRequests, $timeWindow);

// 处理请求前进行限流检查
if ($limiter->allowRequest()) {
    // 允许处理请求的逻辑
    echo "Request allowed.\n";
} else {
    // 请求限流的逻辑
    echo "Request denied. Too many requests within the time window.\n";
}
```

## 滑动窗口算法

使用计数器算法来限制请求时，有时候可能会遇到一个问题，即在计数器重置的瞬间，如果有大量请求同时到达，可能会造成瞬时的高并发，进而影响系统的稳定性。

### 定义

为了避免计数器中的临界问题，让限制更加平滑，将固定窗口中分割出多个小时间窗口，分别在每个小的时间窗口中记录请求次数，然后根据时间将窗口往前滑动并删除过期的小时间窗口。

![](/images/posts/project_design/04.png)

可以发现，计数器算法也是滑动窗口的一种。它没有对时间窗口做进一步地划分，所以只有1格。当滑动窗口的格子划分的越多，那么滑动窗口的滚动就越平滑，限流的统计就会越精确。

### 实现

```php
class SlidingWindowLimiter {
    private $windowSize;        // 滑动窗口大小（秒）
    private $windowInterval;    // 窗格的时间间隔（秒）
    private $maxRequests;       // 窗格内允许的最大请求数
    private $window;            // 滑动窗口数组，每个元素代表一个窗格
    private $currentIndex;      // 当前窗格的索引

    public function __construct($windowSize, $windowInterval, $maxRequests) {
        $this->windowSize = $windowSize;
        $this->windowInterval = $windowInterval;
        $this->maxRequests = $maxRequests;
        $this->initializeWindow();
    }

    public function allowRequest() {
        $this->slideWindowIfNeeded();

        // 检查请求是否超过限制
        if ($this->window[$this->currentIndex] < $this->maxRequests) {
            $this->window[$this->currentIndex]++;
            return true;
        } else {
            return false;
        }
    }

    private function initializeWindow() {
        $numWindows = $this->windowSize / $this->windowInterval;
        $this->window = array_fill(0, $numWindows, 0);
        $this->currentIndex = 0;
    }

    private function slideWindowIfNeeded() {
        $currentTime = time();
        $elapsedTime = $currentTime % $this->windowSize;

        // 计算当前窗格的索引
        $currentIndex = (int)($elapsedTime / $this->windowInterval);

        // 如果当前窗格与上一次记录的不同，表示需要滑动窗口
        if ($currentIndex !== $this->currentIndex) {
            $this->currentIndex = $currentIndex;

            // 重置新的窗格
            $this->window[$this->currentIndex] = 0;
        }
    }
}

// 示例用法
$windowSize = 60;      // 滑动窗口总大小为60秒
$windowInterval = 10;  // 每个窗格的时间间隔为10秒
$maxRequests = 5;      // 在每个窗格内最多处理5个请求
$limiter = new SlidingWindowLimiter($windowSize, $windowInterval, $maxRequests);

// 处理请求前进行限流检查
if ($limiter->allowRequest()) {
    // 允许处理请求的逻辑
    echo "Request allowed.\n";
} else {
    // 请求限流的逻辑
    echo "Request denied. Too many requests within the time window.\n";
}
```

## 漏桶算法

漏桶算法是一种简单而有效的流量控制算法，用于平滑流量的突发性波动。这个算法的核心思想是，将系统对请求的处理速度限制为固定的速率，不论请求的到达速度有多快，都以相同的速率进行处理。

### 定义

举例来说，如果一个系统的处理能力为每秒处理10个请求，而漏桶的容量为20个请求，无论用户以多快的速度发送请求，系统都会以每秒10个的速度进行处理。如果用户发送了30个请求，系统会在3秒内以每秒10个的速度处理这些请求，丢弃多余的请求。

![](/images/posts/project_design/05.png)

### 实现

```php
class LeakyBucket {
    private $capacity;       // 桶的容量
    private $rate;           // 漏水速率（每秒处理请求数）
    private $waterLevel;     // 当前桶中的水量
    private $lastLeakTime;   // 上一次漏水的时间戳

    public function __construct($capacity, $rate) {
        $this->capacity = $capacity;
        $this->rate = $rate;
        $this->waterLevel = 0;
        $this->lastLeakTime = time();
    }

    public function allowRequest() {
        $this->leak();  // 先漏水

        // 检查是否有足够的容量处理请求
        if ($this->waterLevel < $this->capacity) {
            $this->waterLevel++;
            return true;
        } else {
            return false;
        }
    }

    private function leak() {
        $currentTime = time();
        $timeElapsed = $currentTime - $this->lastLeakTime;

        // 计算漏水量
        $leakAmount = $timeElapsed * $this->rate;

        // 更新桶中的水量
        $this->waterLevel = max(0, $this->waterLevel - $leakAmount);

        // 更新上一次漏水的时间戳
        $this->lastLeakTime = $currentTime;
    }
}

// 示例用法
$bucketCapacity = 10;    // 桶的容量
$leakRate = 2;          // 漏水速率，每秒处理2个请求
$leakyBucket = new LeakyBucket($bucketCapacity, $leakRate);

// 处理请求前进行漏桶算法检查
for ($i = 1; $i <= 15; $i++) {
    if ($leakyBucket->allowRequest()) {
        echo "Request {$i}: Allowed\n";
    } else {
        echo "Request {$i}: Denied (Bucket Full)\n";
    }

    // 模拟请求间隔
    sleep(1);
}
```

## 令牌桶算法

### 定义

令牌桶算法是一种用于流量控制和限流的经典算法，它通过维护一个令牌桶，以固定的速率向桶中放入令牌，每个请求需要获取一个令牌才能被处理。当令牌桶中没有足够的令牌时，请求将被限制或延迟处理。

![](/images/posts/project_design/06.png)

### 实现

```php
class TokenBucket {
    private $capacity;          // 令牌桶容量
    private $tokens;            // 当前令牌数量
    private $fillRate;          // 令牌生成速率（令牌/秒）
    private $lastRefillTime;    // 上一次令牌生成时间戳

    public function __construct($capacity, $fillRate) {
        $this->capacity = $capacity;
        $this->tokens = $capacity;
        $this->fillRate = $fillRate;
        $this->lastRefillTime = time();
    }

    public function allowRequest() {
        $this->refillTokens();  // 生成令牌

        // 检查是否有足够的令牌处理请求
        if ($this->tokens > 0) {
            $this->tokens--;
            return true;
        } else {
            return false;
        }
    }

    private function refillTokens() {
        $currentTime = time();
        $timeElapsed = max(0, $currentTime - $this->lastRefillTime);

        // 计算应该生成的令牌数量
        $newTokens = $timeElapsed * $this->fillRate;

        // 令牌桶不能超过最大容量
        $this->tokens = min($this->capacity, $this->tokens + $newTokens);

        // 更新上一次令牌生成时间戳
        $this->lastRefillTime = $currentTime;
    }
}

// 示例用法
$bucketCapacity = 10;    // 令牌桶容量
$fillRate = 2;           // 令牌生成速率，每秒生成2个令牌
$tokenBucket = new TokenBucket($bucketCapacity, $fillRate);

// 处理请求前进行令牌桶算法检查
for ($i = 1; $i <= 15; $i++) {
    if ($tokenBucket->allowRequest()) {
        echo "Request {$i}: Allowed\n";
    } else {
        echo "Request {$i}: Denied (Insufficient Tokens)\n";
    }

    // 模拟请求间隔
    sleep(1);
}
```