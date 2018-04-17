---
layout: post
title:  "redis 一致性hash"
date:   2018-04-17
categories: redis
---

#Redis 一致性hash

##1. 一致性hash的相关理论知识.

	* [The link](http://www.cnblogs.com/haippy/archive/2011/12/10/2282943.html)
	* [The link](http://blog.csdn.net/cywosp/article/details/23397179/)
##2.  还是写一下对一致性hash算法好坏的四个定义
	* 平衡性(Balance).平衡性是指哈希的结果能尽可能的分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。
	* 单调性(Monotonicity).单调性是指如果已经有一些内容通过哈希分派到相应的缓冲中，又有新的缓冲加入到系统中，哈希的结果应能够保证原有已分配的内容可以被映射到原有的或者新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。
	* 分散性(Spread).在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲中时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的算法应该尽量避免不一致的情况发生，也就是尽量降低分散性。
	* 分散性(Load).负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。
	* 平滑性(Smoothness).平滑性是指缓存服务器的数目平滑改变和缓存对象的平滑改变是一致的。
##3. 一致性hash的java实现
	`
	public class ConsistentHash {
	    private SortedMap<Long,String> ketamaNodes=new TreeMap<Long,String>();
	    private int numberOfReplicas=1024;
	    private HashFunction hashFunction= Hashing.md5(); //guava
	    private List<String> nodes;
	    private volatile boolean init=false; //标志是否初始化完成
	
	    public ConsistentHash(int numberOfReplicas,List<String> nodes){
	        this.numberOfReplicas=numberOfReplicas;
	        this.nodes=nodes;
	
	        init();
	    }
	
	    public String getNodeByKey(String key){
	        if(!init)throw new RuntimeException("init uncomplete...");
	
	        byte[] digest=hashFunction.hashString(key, Charset.forName("UTF-8")).asBytes();
	        long hash=hash(digest,0);
	        //如果找到这个节点，直接取节点，返回
	        if(!ketamaNodes.containsKey(hash)){
	            //得到大于当前key的那个子Map，然后从中取出第一个key，就是大于且离它最近的那个key
	            SortedMap<Long,String> tailMap=ketamaNodes.tailMap(hash);
	            if(tailMap.isEmpty()){
	                hash=ketamaNodes.firstKey();
	            }else{
	                hash=tailMap.firstKey();
	            }
	
	        }
	        return ketamaNodes.get(hash);
	    }
	
	    public synchronized void addNode(String node){
	        init=false;
	        nodes.add(node);
	        init();
	    }
	
	    private void init(){
	        //对所有节点，生成numberOfReplicas个虚拟节点
	        for(String node:nodes){
	            //每四个虚拟节点为1组
	            for(int i=0;i<numberOfReplicas/4;i++){
	                //为这组虚拟结点得到惟一名称
	                byte[] digest=hashFunction.hashString(node+i, Charset.forName("UTF-8")).asBytes();
	                //Md5是一个16字节长度的数组，将16字节的数组每四个字节一组，分别对应一个虚拟结点，这就是为什么上面把虚拟结点四个划分一组的原因
	                for(int h=0;h<4;h++){
	                    Long k = hash(digest,h);
	                    ketamaNodes.put(k,node);
	                }
	            }
	        }
	        init=true;
	    }
	
	    public void printNodes(){
	        for(Long key:ketamaNodes.keySet()){
	            System.out.println(ketamaNodes.get(key));
	        }
	    }
	
	    public static long hash(byte[] digest, int nTime)
	    {
	        long rv = ((long)(digest[3 + nTime * 4] & 0xFF) << 24)
	                | ((long)(digest[2 + nTime * 4] & 0xFF) << 16)
	                | ((long)(digest[1 + nTime * 4] & 0xFF) << 8)
	                | ((long)digest[0 + nTime * 4] & 0xFF);
	        return rv;
	    }
	}
	`
##4. 下面看一下jedis中的一致性hash实现。
###1.首先看一个实例,有三个redis，使用ShardedJedisPool连接redis集群。
	`
	public class ShardedRedis {
	    public static void main(String[] args){
	        GenericObjectPoolConfig config=new GenericObjectPoolConfig();
	        config.setMaxTotal(1000);
	        config.setMaxIdle(500);
	
	        List<JedisShardInfo> jedisShardInfoList=new ArrayList<JedisShardInfo>();
	        JedisShardInfo shardInfo1=new JedisShardInfo("192.168.217.157",6380);
	        JedisShardInfo shardInfo2=new JedisShardInfo("192.168.217.157",6381);
	        JedisShardInfo shardInfo3=new JedisShardInfo("192.168.217.157",6382);
	        jedisShardInfoList.add(shardInfo1);
	        jedisShardInfoList.add(shardInfo2);
	        jedisShardInfoList.add(shardInfo3);
	
	        ShardedJedisPool pool=new ShardedJedisPool(config,jedisShardInfoList);
	
	        set("user1","a",pool);
	        set("user12","a",pool);
	        set("user13","a",pool);
	        set("usera","a",pool);
	        set("userb","a",pool);
	    }
	
	    public static void set(String key,String value,ShardedJedisPool pool){
	        ShardedJedis shardedJedis=pool.getResource();
	        shardedJedis.set(key,value);
	        pool.returnResource(shardedJedis);
	    }
	}

	`
###2.jedis实现
	* Jedis是通过ShardedJedis向redis集群写入的数据,ShardedJedis中的关键方法：<br/>
	`
	public Sharded(List<S> shards, Hashing algo, Pattern tagPattern) {
		this.algo = algo;
		this.tagPattern = tagPattern;
		initialize(shards);
    }

	//初始化哈希环
    private void initialize(List<S> shards) {
		nodes = new TreeMap<Long, S>();
	
		for (int i = 0; i != shards.size(); ++i) {
		    final S shardInfo = shards.get(i);
		    if (shardInfo.getName() == null)
			for (int n = 0; n < 160 * shardInfo.getWeight(); n++) {
			    nodes.put(this.algo.hash("SHARD-" + i + "-NODE-" + n),
				    shardInfo);
			}
		    else
			for (int n = 0; n < 160 * shardInfo.getWeight(); n++) {
			    nodes.put(
				    this.algo.hash(shardInfo.getName() + "*"
					    + shardInfo.getWeight() + n), shardInfo);
			}
		    resources.put(shardInfo, shardInfo.createResource());
		}
    }

	//将key，value存储到相应的shard
	 public String set(String key, String value) {
		Jedis j = getShard(key);
		return j.set(key, value);
	 }

	public R getShard(String key) {
		return resources.get(getShardInfo(key));
    }

	//根据key获取shard
    public S getShardInfo(byte[] key) {
		SortedMap<Long, S> tail = nodes.tailMap(algo.hash(key));
		if (tail.isEmpty()) {
		    return nodes.get(nodes.firstKey());
		}
		return tail.get(tail.firstKey());
    }
	`
	* 以上就是jedis中一致性hash的主要代码，当redis单机存储无法满足要求时，可以考虑使用一致性hash构建redis集群。