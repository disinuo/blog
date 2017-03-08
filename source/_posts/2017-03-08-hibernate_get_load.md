---
title: hibernate：Get vs Load -- 源码分析
date: 2017-03-07 23:35:37
updated: 2017-03-07
tags:
    - j2ee
    - hibernate
---
## 前言
- 本文贴的源码都是删减版。。。~一些异常的判断、处理，都被省略掉了
只贴了主要的部分~
- Hibernate版本  5.1.0
<!-- more -->

## 源码比较
### SessionImpl类
```java
// 848。被dao直接调用的方法
// ------load-------
public <T> T load(Class<T> entityClass, Serializable id) throws HibernateException {
  return this.byId(entityClass).getReference(id);
}
public Object load(String entityName, Serializable id) throws HibernateException {
  return this.byId(entityName).getReference(id);
}
// ------get-------
public <T> T get(Class<T> entityClass, Serializable id) throws HibernateException {
  return this.byId(entityClass).load(id);
}
public Object get(String entityName, Serializable id) throws HibernateException {
  return this.byId(entityName).load(id);
}
```
load调用了getReference；get调用了load【感觉这方法名有点迷。。。
然后这两个方法做一些判断后，分别调用了`doReference`和`doLoad`
```java
// 2398。load调用doGetReference
protected T doGetReference(Serializable id) {
  //实体id,实体名,LockOptions,session
  LoadEvent event=new LoadEvent(id, this.entityPersister.getEntityName(), false, SessionImpl.this);
  SessionImpl.this.fireLoad(event, LoadEventListener.LOAD);//触发事件的方式
  return event.getResult();
}
------------
// 2446。get调用doLoad
protected final T doLoad(Serializable id) {
   LoadEvent event = new LoadEvent(id, this.entityPersister.getEntityName(), false, SessionImpl.this);
   SessionImpl.this.fireLoad(event, LoadEventListener.GET);
   return event.getResult();
}
 ```
 调用同一个方法，只是LoadType不同。
 先看这个相同的方法
 ```java
// 986。
private void fireLoad(LoadEvent event, LoadType loadType) {
  // 注意这里是获得【事件】类型为LOAD的监听器。事件类型和加载类型是不一样的概念！
  // 本文解析的get()方法和load()方法的事件类型都是LOAD，但加载类型不同
  Iterator var3 = this.listeners(EventType.LOAD).iterator();
  while(var3.hasNext()) {
      LoadEventListener listener = (LoadEventListener)var3.next();
      listener.onLoad(event, loadType);
  }
}
```
比较LoadType的不同之处
```java
// Get 允许为null,不允许创建代理。
// LOAD 不允许为null,允许创建代理
LoadEventListener.LoadType GET = (new LoadEventListener.LoadType("GET")).
                                setAllowNulls(true).
                                setAllowProxyCreation(false).
                                setCheckDeleted(true).
                                setNakedEntityReturned(false);
LoadEventListener.LoadType LOAD = (new LoadEventListener.LoadType("LOAD")).
                                setAllowNulls(false).
                                setAllowProxyCreation(true).
                                setCheckDeleted(true).
                                setNakedEntityReturned(false);
```
### DefaultLoadEventListener类
get和load都调用这个onLoad，并且二者的LoadEvent一模一样
```java
// 63。
public void onLoad(LoadEvent event, LoadType loadType) throws HibernateException {
   EntityPersister persister = this.getPersister(event);
   this.doOnLoad(persister, event, loadType);
 }
// 86。两者的persister,event都相同。只有loadType不同
private void doOnLoad(EntityPersister persister, LoadEvent event, LoadType loadType) {
//  entityKey相同
   EntityKey e = event.getSession().generateEntityKey(event.getEntityId(), persister);
   if(loadType.isNakedEntityReturned()) {//-----情况1 两者都不是这种情况
       event.setResult(this.load(event, persister, e, loadType));
   } else if(event.getLockMode() == LockMode.NONE) {//-----情况2
       event.setResult(this.proxyOrLoad(event, persister, e, loadType));
   } else {//------情况3
       event.setResult(this.lockAndLoad(event, persister, e, loadType, event.getSession()));
   }
}
 ```
 因为LoadEvent的构造函数把LockMode设置成了`DEFAULT_LOCK_MODE`,
 而其类的内部又定义了 `DEFAULT_LOCK_MODE = LockMode.NONE;`
 所以get和load都属于情况2
#### 分流get和load
proxyOrLoad方法
```java
//  152。
private Object proxyOrLoad(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options) {
  if(!persister.hasProxy()) {
    //   hasProxy() Determine whether this entity supports dynamic proxies.
    // 不支持代理的话，就老实的去调用load
      return this.load(event, persister, keyToLoad, options);//等下下面会提到这个方法
  } else {
    // 支持代理
      PersistenceContext persistenceContext = event.getSession().getPersistenceContext();
      Object proxy = persistenceContext.getProxy(keyToLoad);
      // 这句代码有点长。
      // 大概意思就是，通过上下文获得一个代理
      // 这个代理非空的话，则返回一个NarrowProxy
      // 这个代理为空的话-----如果允许创建代理--LOAD---就调用创建代理方法
      // ------------------不允许创建--GET---------再调用上面的load-->doLoad
      return proxy != null?
      this.returnNarrowedProxy(event, persister, keyToLoad, options, persistenceContext, proxy)
      :(
          options.isAllowProxyCreation()?
          this.createProxyIfNecessary(event, persister, keyToLoad, options, persistenceContext)
          :this.load(event, persister, keyToLoad, options)
      );
  }
}
// 187。
private Object createProxyIfNecessary(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options, PersistenceContext persistenceContext) {
    Object existing = persistenceContext.getEntity(keyToLoad);
    if(existing != null) {//上下文中有这个实体，则返回
        return existing;
    } else {//没有这个实体，则创建代理并返回。就是是假的实体！！！
        Object proxy = persister.createProxy(event.getEntityId(), event.getSession());
        persistenceContext.getBatchFetchQueue().addBatchLoadableEntityKey(keyToLoad);
        persistenceContext.addProxy(keyToLoad, proxy);
        return proxy;
    }
}
```
#### 实实在在的加载数据
```java
// 132。
private Object load(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options) {
   Object entity = this.doLoad(event, persister, keyToLoad, options);
   return entity;
}
// 239。session缓存-->二级缓存-->数据库
private Object doLoad(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options) {
   Object entity = this.loadFromSessionCache(event, keyToLoad, options);
   if(entity == REMOVED_ENTITY_MARKER) {
       LOG.debug("Load request found matching entity in context, but it is scheduled for removal; returning null");
       return null;
   } else if(entity == INCONSISTENT_RTN_CLASS_MARKER) {
       LOG.debug("Load request found matching entity in context, but the matched entity was of an inconsistent return type; returning null");
       return null;
   } else if(entity != null) {
       return entity;
   } else {
       entity = this.loadFromSecondLevelCache(event, persister, keyToLoad);
       if(entity != null) {
          //  省略。这里源代码在写日志~
       } else {
           entity = this.loadFromDatasource(event, persister);
       }
       return entity;
   }
}
```
另外，当访问load得到的对象的属性时，会访问数据库。--------所谓的延迟加载
## 使用比较
- load拿到的是假对象为什么还要用它？
举个例子~
就比如一个预订房间系统，用户每提交一个订单Order就应该包含一个房间Room
如果每次都像下面这样的话
```
Room room=session.get(Room.class,roomId);
order.setRoom(room);
```
  就每次都要访问数据库，开销太大。
  因为这里拿到room之后我们也不需要得到它的属性值，只是set给order，其实只需要一个id就够了
  所以可以用load来加载实体~如果订单数量大的话差距会很明显的！~
- 听起来好像不错那我们为什么还要用get？(下面假设实体支持代理)
因为load不管三七二十一都会返回一个对象。
由于返回对象的时候不访问数据库，所以load返回的对象不能保证真的在数据库里。一旦我们访问对象的属性---访问数据库的时候，发现这个对象并不存在，就会抛异常，后知后觉。。。
而get，当需要加载的对象不存在时，会返回null，我们可以立即知道这个对象是否真的存在
- 所以 当确定对象存在的时候再用load
## 要点总结
1. 绝大部分情况，get返回实体，load返回代理
2. 当session缓存中有代理时，get会返回代理
3. 当session缓存中没有代理、但有实体时，load返回实体
4. 实体不支持代理时，get和load相当于是一样的

------------
参考资料

[hibernate的get和load](https://www.mkyong.com/hibernate/different-between-session-get-and-session-load/)
[Hibernate Session get() vs load() difference with examples](http://www.journaldev.com/3472/hibernate-session-get-vs-load-difference-with-examples)
[Hibernate深入浅出（八）持久层操作——延迟加载（Lazy Loading）](http://1831651.blog.51cto.com/1821651/1225423)
[从源码上分析hibernate的load和get之间的区别](https://www.iflym.com/index.php/code/201112050001.html)
[Understanding Caching in Hibernate ](https://www.dynatrace.com/blog/understanding-caching-in-hibernate-part-one-the-session-cache/)
[Hibernate - Difference between session's get() and load()](http://gmarwaha.blogspot.co.at/2007/01/hibernate-difference-between-sessions.html)
[hibernate的get和load区别](http://www.cnblogs.com/binjoo/articles/1621254.html)
