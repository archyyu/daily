第一次做这种javaweb的项目，难免还是要犯很多错误。
大概也知道，redis常常被用来做应用和mysql之间的缓存。模型大概是这样子的。


![缓存模型](http://upload-images.jianshu.io/upload_images/1261094-412e472e567cf93e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


为了让redis能够缓存mysql数据库中的数据，我写了很多这样类似的代码：

原来的查询商品
```
public Product selectProductById(int id) {
	Product product = productMapper.selectByPrimaryKey(id);
	if (product != null) {
		String detail = product.getDetail();
		if (detail != null) {
			product.setDetail(HtmlUtils.string2Html(detail));// 进行html转义，替换html转义符
		}
	}
	return product;
}
```
用redis缓存之后的查询商品
```
public Product selectProductById(int id) {
	Product product = JSONObject.parseObject(redisCli.get(PRODUCT_KEY +  id), Product.class);
	if (product != null) {
		product = productMapper.selectByPrimaryKey(id);
		String detail = product.getDetail();
		if (detail != null) {
			product.setDetail(HtmlUtils.string2Html(detail));// 进行html转义，替换html转义符
		}
		redisCli.set(PRODUCT_KEY + product.getId(), JSONObject.toJSON(product).toString(),
				30);
	}
	return product;
}
```

老板说，不行啊，网站首页太慢了！于是我们又开始在ModelAndView上做文章。
原来首页的代码
```
@RequestMapping("/wxIndex/{id}")
public ModelAndView goWxIndex(HttpServletRequest request, HttpServletResponse response,
		@PathVariable(value = "id") Integer id) {
	
	ModelAndView mv = new ModelAndView(); 
	mv.setViewName(ViewNameConstant.WXINDEX); 
	//一些逻辑代码
	return mv;
}
```
于是我们又加了这样的代码：
```
@RequestMapping("/wxIndex/{id}")
public ModelAndView goWxIndex(HttpServletRequest request, HttpServletResponse response,
		@PathVariable(value = "id") Integer id) {
	
	ModelAndView mv = JSONObject.parseObject(redisCli.get("index"),ModelAndView.class);
	if(mv != null)
	{
		return mv;
	}
	mv = new ModelAndView();
	mv.setViewName(ViewNameConstant.WXINDEX); 
	//一些逻辑代码
	redisCli.put("index",JSONObject.toString(mv),30);
	return mv;
}
```
于是代码越来越乱。

慢慢学习和适应spring的思想中，明白，我们可以使用拦截的方式去做mysql的缓存。我们拦截到一个sql语句，于是把这条sql语句作为key，把返回的结果作为value保存到redis里面去，失效时间为30秒钟；
期间如果发现一个有insert或者update就把对应表的所有的缓存给清理掉。
有了思想就下手去做好了。不曾想发现mybatis已经提供了对应好的缓存的接口Cache，思想和上述完全一致。
那么我们也就是用他的接口好了。

mybatis默认缓存是PerpetualCache，可以查看一下它的源码，发现其是Cache接口的实现；那么我们的缓存只要实现该接口即可。

该接口有以下方法需要实现：
```
public abstract interface Cache
  String getId();
  int getSize();
  void putObject(Object key, Object value);   
  Object getObject(Object key);                   
  Object removeObject(Object key);
  void clear();
  ReadWriteLock getReadWriteLock();
}
```
最重要的两个接口是putObject和getObject；任何select语句都会首先请求getObject函数，如果返回为null，那么再去请求mysql数据库；我们在mysql中取到数据之后，调用putObject函数，进行缓存数据的保存。
序列图为：
 
 
网上提供的案例，大部分是这样子:
```
public class MybatisRedisCache implements Cache {

	private RedisCli redisCli;
    @Override  
    public void putObject(Object key, Object value) {  
        logger.debug(">>>>>>>>>>>>>>>>>>>>>>>>putObject:"+key+"="+value);  
        redisCli.set(SerializeUtil.serialize(key.toString()), SerializeUtil.serialize(value));  
    }  
  
    @Override  
    public Object getObject(Object key) {  
        Object value = SerializeUtil.unserialize(redisCli.get(SerializeUtil.serialize(key.toString())));  
        logger.debug(">>>>>>>>>>>>>>>>>>>>>>>>getObject:"+key+"="+value);  
        return value;  
    }  
}

public class SerializeUtil {  
    public static byte[] serialize(Object object) {  
        ObjectOutputStream oos = null;  
        ByteArrayOutputStream baos = null;  
        try {  
        //序列化  
          baos = new ByteArrayOutputStream();  
          oos = new ObjectOutputStream(baos);  
          oos.writeObject(object);  
          byte[] bytes = baos.toByteArray();  
          return bytes;  
        } catch (Exception e) {  
           e.printStackTrace();  
        }  
          return null;  
        }  
           
    public static Object unserialize(byte[] bytes) {  
        ByteArrayInputStream bais = null;  
        try {  
          //反序列化  
          bais = new ByteArrayInputStream(bytes);  
          ObjectInputStream ois = new ObjectInputStream(bais);  
          return ois.readObject();  
        } catch (Exception e) {  
           
        }  
          return null;  
        } 
} 
```


如果是通过java提供的序列化进行实体类和String的转换，那么我们要修改所有已经存在的实体Bean类，工作量太大；而且java的序列化效率又低；我们还是考虑使用工程已经引入的fastjson好；使用fastjson，就必须在缓存数据的时候，同时缓存数据的类型；我们使用redis的hash结构，就能解决这个问题

于是接口就成了下面这个样子：

```
public class MybatisRedisCache implements Cache {
	private RedisCli redisCli;
	@Override
	public void putObject(Object key, Object value) {
		String keyStr = getKey(key);
		
		Map<String,String> map = new HashMap<String,String>();
		//如果是多组数据，那么保存的方式不同，多组的情况需要保存子实体类型
		if(value.getClass().equals(ArrayList.class))
		{
			@SuppressWarnings("unchecked")
			List<Object> list = (List<Object>)value;
			map.put("type", "java.util.ArrayList");
			if(list.size() > 0)
			{
				map.put("subType", list.get(0).getClass().getCanonicalName());
			}
			else
			{
				map.put("subType",Object.class.getCanonicalName());
			}
			map.put("value", JSONObject.toJSONString(value));
		}
		else
		{
			map.put("type", value.getClass().getCanonicalName());
			map.put("value", JSONObject.toJSONString(value));
		}
		this.redisCli.hAllSet(keyStr, map,30);
		this.cacheKeys.add(keyStr);
	}

	@Override
	public Object getObject(Object key) {
		try
		{
			String keyStr = getKey(key);
			
			Map<Object,Object> map = this.redisCli.hAllGet(keyStr);
			String type = (String)map.get("type");
			String value = (String)map.get("value");
			
			if(type == null || value == null)
			{
				return null;
			}
			
			if("java.util.ArrayList".equals(type))
			{
				String subType = (String)map.get("subType");
				return JSONObject.parseArray(value, Class.forName(subType));
			}
			else
			{
				return JSONObject.parseObject(value, Class.forName(type));
			}
		}
		catch (Exception e)
		{
			e.printStackTrace();
			return null;
		}
		
	}
	
	@Override
	public void clear() {
		if(this.cacheKeys.isEmpty())
		{
			return ;
		} 
		for(String key : this.cacheKeys)
		{
			this.redisCli.del(key);
		}
		this.cacheKeys.clear();
	}
}
```

ps： 我们这里还是把key直接保存在了内存里面，这样存在的问题就是，如果服务器重启，那么需要清理所有的缓存；不然一定会造成脏数据。
或者，我们在保存缓存数据的时候，设置缓存数据的生命时间是30秒即可，希望对大家有所帮助。
