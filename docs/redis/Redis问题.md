有一个业务场景，存储图片id和图片信息存储到数据库的id，使用 String、String 来存储会导致使用容量特别多，是因为SDS有许多的对象信息属性，以及 hash 所在的元属性，next、data、...

常规使用 String、String 存储对象占用大小 

```
127.0.0.1:6379> info memory
# Memory
used_memory:709528
used_memory_human:692.90K
used_memory_rss:672600
used_memory_rss_human:656.84K
used_memory_peak:231888872
used_memory_peak_human:221.15M
total_system_memory:0
total_system_memory_human:0B
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:0.95
mem_allocator:jemalloc-3.6.0
127.0.0.1:6379> info memory
# Memory
used_memory:88702336
used_memory_human:84.59M
used_memory_rss:88665408
used_memory_rss_human:84.56M
used_memory_peak:231888872
used_memory_peak_human:221.15M
total_system_memory:0
total_system_memory_human:0B
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.00
mem_allocator:jemalloc-3.6.0
```

87,992,808



使用 hash 占用内存大小 String(key 前5位)、hash(String(key 后3位)、String)

```
127.0.0.1:6379> info memory
# Memory
used_memory:710648
used_memory_human:693.99K
used_memory_rss:673720
used_memory_rss_human:657.93K
used_memory_peak:231888872
used_memory_peak_human:221.15M
total_system_memory:0
total_system_memory_human:0B
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:0.95
mem_allocator:jemalloc-3.6.0
127.0.0.1:6379> info memory
# Memory
used_memory:19956880
used_memory_human:19.03M
used_memory_rss:19919952
used_memory_rss_human:19.00M
used_memory_peak:231888872
used_memory_peak_human:221.15M
total_system_memory:0
total_system_memory_human:0B
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.00
mem_allocator:jemalloc-3.6.0
```

19,246,232



Java Demo

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
@EnableAutoConfiguration
public class RedisTests {

    @Autowired
    RedisTemplate<String, String> redisTemplate;

    @Test
    public void test() {
        int n = 1000000;
        for (int i = 0; i < n; i++) {
            String key = RandomUtil.randomNumbers(8);
            String val = RandomUtil.randomNumbers(8);
            redisTemplate.opsForValue().set(key, val);
        }
    }

    @Test
    public void test2() {
        int n = 1000000;
        for (int i = 0; i < n; i++) {
            String allKey = RandomUtil.randomNumbers(8);
            String val = RandomUtil.randomNumbers(8);
            String key = allKey.substring(0, 5);
            String hashKey = allKey.substring(5);
            redisTemplate.opsForHash().put(key, hashKey, val);
        }
    }
}
```

