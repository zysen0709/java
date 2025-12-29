# 典型回答

滑动窗口限流是一种流量控制策略，用于控制在一定时间内允许执行的操作数量或请求频率。它的工作方式类似于一个滑动时间窗口，在窗口内允许的操作数量是固定的，窗口会随着时间的推移不断滑动。

[✅什么是滑动窗口限流？](https://www.yuque.com/hollis666/fo22bm/hvuigwzxls4qd3sy?view=doc_embed)

滑动窗口限流的主要**优点是可以在时间内平滑地控制流量**，而不是简单地设置固定的请求数或速率。这使得系统可以更**灵活地应对突发流量或峰值流量**，而不会因为固定速率的限制而浪费资源或降低系统性能。

利用Redis，我们就可以实现一个简单的滑动窗口限流的功能。因为滑动窗口和时间有关，所以很容易能想到要基于时间进行统计。

**那么我们只需要在每一次有请求进来的时候，记录下请求的时间戳和请求的数据，然后在统计窗口内请求的数量时，只需要统计窗口内的被记录的数据量有多少条就行了。**

在Redis中，我们可以基于ZSET来实现这个功能。假如我们限定login接口一分钟只能调用100次：

那么，我们就可以把login接口这个需要做限流的资源名作为key在redis中进行存储，然后value我们选择ZSET这种数据结构，把他的score设置为当前请求的时间戳，member的话建议用请求的详情的hash进行存储（或者UUID、MD5什么的），避免在并发时，时间戳一致出现scode和member一样导致被zadd幂等的问题。<br/>![image.png](https://cdn.nlark.com/yuque/0/2023/png/5378072/1697870905641-4dc17d40-7078-4c52-8e35-809d363812d4.png#averageHue=%23fefaf8&clientId=u51215e88-2426-4&from=paste&height=288&id=u0c8d88db&originHeight=288&originWidth=772&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25682&status=done&style=none&taskId=u126b54d8-0865-44a1-bc8c-85fd564e7f0&title=&width=772)


所以，我们实现滑动窗口限流的主要思想是：**只保留在特定时间窗口内的请求记录，而丢弃窗口之外的记录。**

主要步骤如下：

1. 定义滑动窗口的时间范围，例如，窗口大小为60秒。
2. 每次收到一个请求时，我们就定义出一个zset然后存储到redis中。
3. 然后再通过ZREMRANGEBYSCORE命令来删除分值小于窗口起始时间戳（当前时间戳-60s）的数据。
4. 最后，再使用ZCARD命令来获取有序集合中的成员数量，即在窗口内的请求量。

```java
import redis.clients.jedis.Jedis;

public class SlidingWindowRateLimiter {
    private Jedis jedis;
    private String key;
    private int limit;

    public boolean allowRequest(String key) {
        //当前时间戳
        long currentTime = System.currentTimeMillis();
        //窗口开始时间是当前时间减60s
        long windowStart = currentTime - 60 * 1000;
        //删除窗口开始时间之前的所有数据
        jedis.zremrangeByScore(key, "-inf", String.valueOf(windowStart));
        //计算总请求数
        long currentRequests = jedis.zcard(key);
    	//窗口足够则把当前请求加入
        if (currentRequests < limit) {
            jedis.zadd(key, currentTime, String.valueOf(currentTime));
            return true;
        }

        return false;
    }
}

```

以上代码在高并发情况下，可能会存在原子性的问题，需要考虑加事务或者lua脚本：

```java
import redis.clients.jedis.Jedis;

public class SlidingWindowRateLimiter {
    private Jedis jedis;
    private String key;
    private int limit;

    public SlidingWindowRateLimiter(Jedis jedis, String key, int limit) {
        this.jedis = jedis;
        this.key = key;
        this.limit = limit;
    }

    public boolean allowRequest(String key) {
        // 当前时间戳
        long currentTime = System.currentTimeMillis();

        // 使用Lua脚本来确保原子性操作
        String luaScript = "local window_start = ARGV[1] - 60000\n" +
                           "redis.call('ZREMRANGEBYSCORE', KEYS[1], '-inf', window_start)\n" +
                           "local current_requests = redis.call('ZCARD', KEYS[1])\n" +
                           "if current_requests < tonumber(ARGV[2]) then\n" +
                           "    redis.call('ZADD', KEYS[1], ARGV[1], ARGV[1])\n" +
                           "    return 1\n" +
                           "else\n" +
                           "    return 0\n" +
                           "end";

        Object result = jedis.eval(luaScript, 1, key, String.valueOf(currentTime), String.valueOf(limit));
        
        return (Long) result == 1;
    }
}

```
# 扩展知识

# ZREMRANGEBYSCORE 

 Redis Zremrangebyscore 命令用于移除有序集中，指定分数（score）区间内的所有成员。

redis Zremrangebyscore 命令基本语法如下：

```java
 ZREMRANGEBYSCORE key min max
```

而我们代码中使用的'-inf'在redis中表示负无穷。-inf代表负无穷，+inf代表正无穷



在提供的Java代码中，使用 `jedis.zadd` 方法是为了将当前的请求时间戳添加到Redis的有序集合中。这个方法是Jedis库提供的，用于操作Redis中的有序集合数据类型。下面是对这个方法的详细解释：



### `jedis.zadd` 方法

`zadd` 是Redis命令，用于向有序集合中添加一个或多个成员，或者更新已存在成员的分数。在有序集合中，每个成员都是唯一的，但分数（score）可以重复。成员在集合中按分数进行排序。

#### 方法签名

在Jedis库中，`zadd` 方法的常见签名是：

```java
public Long zadd(String key, double score, String member);
```

- **key**: 有序集合的键名。
- **score**: 成员的分数，用于排序。在这个场景中，分数是请求的时间戳。
- **member**: 要添加到集合中的成员。在这个场景中，成员也是请求的时间戳，转换为字符串格式。

#### 使用场景

在滑动窗口速率限制器中，`zadd` 被用来记录一个事件或请求发生的时间。通过将时间戳作为分数和成员值添加到Redis的有序集合中，可以轻松地根据时间范围（使用分数）查询、计数或删除旧的请求记录。

#### 示例代码中的使用

```java
jedis.zadd(key, currentTime, String.valueOf(currentTime));
```

这行代码执行以下操作：

1. **key**: 是有序集合的键名，用于标识特定的速率限制器实例。
2. **currentTime**: 作为分数，用于排序和后续的时间范围查询。
3. **String.valueOf(currentTime)**: 将当前时间戳转换为字符串，作为成员值存储。这样做可以确保每个请求都有唯一的标识，即使它们在同一毫秒内发生。

### 结果

使用 `zadd` 方法后，如果成员是新添加的，它会返回1，表示成功添加了新成员。如果成员已经存在并且分数被更新，它会返回0。这个返回值通常用于判断操作是否成功，但在上述代码中没有使用这个返回值。

通过这种方式，`jedis.zadd` 在滑动窗口速率限制器中起到了关键作用，使得可以有效地跟踪和限制在给定时间窗口内的请求频率。





要实现对一个 `login` 接口的速率限制，你可以使用多种技术和工具。这里，我将提供一个基于Redis的滑动窗口速率限制器的实现流程，这种方法可以有效地限制短时间内对`login`接口的请求频率，以防止滥用和DoS攻击。

### 步骤 1: 设定环境

首先，确保你的系统中安装了Redis，并且有一个可用的Redis服务。同时，确保你的应用程序能够使用Jedis或其他Redis客户端库与Redis进行通信。

### 步骤 2: 实现速率限制器

你可以使用上文提供的 `SlidingWindowRateLimiter` 类作为速率限制器的实现。这个类使用Redis的有序集合来跟踪特定时间窗口内的请求。

```java
import redis.clients.jedis.Jedis;

public class SlidingWindowRateLimiter {
    private Jedis jedis;
    private String key;
    private int limit;

    public SlidingWindowRateLimiter(Jedis jedis, String key, int limit) {
        this.jedis = jedis;
        this.key = key;
        this.limit = limit;
    }

    public boolean allowRequest() {
        long currentTime = System.currentTimeMillis();
        long windowStart = currentTime - 60 * 1000; // 60 seconds window
        jedis.zremrangeByScore(key, "-inf", String.valueOf(windowStart));
        long currentRequests = jedis.zcard(key);
        if (currentRequests < limit) {
            jedis.zadd(key, currentTime, String.valueOf(currentTime));
            return true;
        }
        return false;
    }
}
```

### 步骤 3: 集成到登录接口

在你的应用程序中，找到处理登录请求的部分。在处理登录逻辑之前，使用`SlidingWindowRateLimiter`来检查是否允许当前请求。

```java
public class LoginController {
    private SlidingWindowRateLimiter rateLimiter;

    public LoginController() {
        this.rateLimiter = new SlidingWindowRateLimiter(new Jedis("localhost", 6379), "loginRateLimiter", 100); // 100 requests per minute
    }

    public void handleLoginRequest(String username, String password) {
        if (!rateLimiter.allowRequest()) {
            System.out.println("Rate limit exceeded. Try again later.");
            return;
        }

        // Proceed with login logic
        System.out.println("Login successful for user: " + username);
    }
}
```

### 步骤 4: 测试

在部署这个逻辑到生产环境之前，进行彻底的测试以确保速率限制器按预期工作。测试应该包括正常的登录尝试和超过速率限制的情况。

### 步骤 5: 监控和调整

一旦部署，持续监控应用程序的行为，确保没有误拦截合法请求，同时速率限制器能够有效防止滥用。根据实际情况调整时间窗口大小和请求限制。

通过这套流程，你可以有效地为`login`接口实现速率限制，提高应用的安全性和稳定性。
