---
title: hibernate持久化---new一个对象 从session.save到存入数据库发生了什么
date: 2017-03-07 00:02:37
<!-- updated: 2017-03-07  -->
tags:
    - j2ee
    - hibernate
---
DAO: session.save(bean);

 **SessionImpl:实现session**
```java
public Serializable save(Object obj) throws HibernateException {
        return this.save((String)null, obj);
    }
public Serializable save(String entityName, Object object) throws HibernateException {
    return this.fireSave(new SaveOrUpdateEvent(entityName, object, this));
}

private Serializable fireSave(SaveOrUpdateEvent event) {
// 省略检查session有没有关闭之类的异常情况
			 Iterator var2 = this.listeners(EventType.SAVE).iterator();
			//  返回了所有SAVE类型的事件的listener
			 while(var2.hasNext()) {
					 SaveOrUpdateEventListener listener = (SaveOrUpdateEventListener)var2.next();
					 listener.onSaveOrUpdate(event);
			 }

			 this.checkNoUnresolvedActionsAfterOperation();
			 return event.getResultId();
	 }


// Listener的onSaveOrUpdate方法

	 public void onSaveOrUpdate(SaveOrUpdateEvent event) {
			 EventSource source = event.getSession();//获取event所在的session
			 Object object = event.getObject();
			 Serializable requestedId = event.getRequestedId();
			 if(requestedId != null && object instanceof HibernateProxy) {
					 ((HibernateProxy)object).getHibernateLazyInitializer().setIdentifier(requestedId);
			 }

			 if(this.reassociateIfUninitializedProxy(object, source)) {
					 LOG.trace("Reassociated uninitialized proxy");
			 } else {
					 Object entity = source.getPersistenceContext().unproxyAndReassociate(object);
					 event.setEntity(entity);
					 event.setEntry(source.getPersistenceContext().getEntry(entity));
					 event.setResultId(this.performSaveOrUpdate(event));
			 }

	 }
```
## Transient Persistent Detached的关系
transient: never persistent, not associated with any Session
persistent: associated with a unique Session
detached: previously persistent, not associated with any Session

Transient instances may be made persistent by calling save(), persist() or saveOrUpdate(). Persistent instances may be made transient by calling delete(). Any instance returned by a get() or load() method is persistent. Detached instances may be made persistent by calling update(), saveOrUpdate(), lock() or replicate(). The state of a transient or detached instance may also be made persistent as a new persistent instance by calling merge().

save() and persist() result in an SQL INSERT, delete() in an SQL DELETE and update() or merge() in an SQL UPDATE. Changes to persistent instances are detected at flush time and also result in an SQL UPDATE. saveOrUpdate() and replicate() result in either an INSERT or an UPDATE.

## 3.1. Entity states

Like in Hibernate (comparable terms in parentheses), an entity instance is in one of the following states:

-	New (transient): an entity is new if it has just been instantiated using the new operator, and it is not associated with a persistence context. It has no persistent representation in the database and no identifier value has been assigned.
-	Managed (persistent): a managed entity instance is an instance with a persistent identity that is currently associated with a persistence context.
-	Detached: the entity instance is an instance with a persistent identity that is no longer associated with a persistence context, usually because the persistence context was closed or the instance was evicted from the context.
-	Removed: a removed entity instance is an instance with a persistent identity, associated with a persistence context, but scheduled for removal from the database.
The EntityManager API allows you to change the state of an entity, or in other words, to load and store objects. You will find persistence with JPA easier to understand if you think about object state management, not managing of SQL statements.




DefaultSaveOrUpdateEventListener的对transient事件(包括transient和deleted)的处理方法
```java
protected Serializable entityIsTransient(SaveOrUpdateEvent event) {
		LOG.trace("Saving transient instance");
		EventSource source = event.getSession();
		EntityEntry entityEntry = event.getEntry();
		if(entityEntry != null) {//讲道理应该是deleted了，那么status应该是deleted
				if(entityEntry.getStatus() != Status.DELETED) {//竟然不是deleted!抛异常
						throw new AssertionFailure("entity was persistent");
				}
				// 是deleted的实体，于是强行flush
				source.forceFlush(entityEntry);
		}
		// 如果是transient的对象 会直接跳到这里。
		Serializable id = this.saveWithGeneratedOrRequestedId(event);
		source.getPersistenceContext().reassociateProxy(event.getObject(), id);
		return id;
}

protected Serializable saveWithGeneratedOrRequestedId(SaveOrUpdateEvent event){
    return this.saveWithGeneratedId(event.getEntity(), event.getEntityName(), (Object)null, event.getSession(), true);
}

protected Serializable saveWithGeneratedId(Object entity, String entityName, Object anything, EventSource source, boolean requiresImmediateIdAccess) {
			 if(entity instanceof SelfDirtinessTracker) {
					 ((SelfDirtinessTracker)entity).$$_hibernate_clearDirtyAttributes();
			 }

			 EntityPersister persister = source.getEntityPersister(entityName, entity);
			 Serializable generatedId = persister.getIdentifierGenerator().generate(source, entity);
			 if(generatedId == null) {
					 throw new IdentifierGenerationException("null id generated for:" + entity.getClass());
			 } else if(generatedId == IdentifierGeneratorHelper.SHORT_CIRCUIT_INDICATOR) {
					 return source.getIdentifier(entity);
			 } else if(generatedId == IdentifierGeneratorHelper.POST_INSERT_INDICATOR) {
					 return this.performSave(entity, (Serializable)null, persister, true, anything, source, requiresImmediateIdAccess);
			 } else {
					 if(LOG.isDebugEnabled()) {
							 LOG.debugf("Generated identifier: %s, using strategy: %s", persister.getIdentifierType().toLoggableString(generatedId, source.getFactory()), persister.getIdentifierGenerator().getClass().getName());
					 }

					 return this.performSave(entity, generatedId, persister, false, anything, source, true);
			 }
	 }


	 public EntityPersister getEntityPersister(String entityName, Object object) {
        this.errorIfClosed();
        if(entityName == null) {
            return this.factory.getEntityPersister(this.guessEntityName(object));
        } else {
            try {
                return this.factory.getEntityPersister(entityName).getSubclassEntityPersister(object, this.getFactory());
            } catch (HibernateException var6) {
                try {
                    return this.getEntityPersister((String)null, object);
                } catch (HibernateException var5) {
                    throw var6;
                }
            }
        }
    }
		// 类：DynamicMapEntityTuplizer
		public static String extractEmbeddedEntityName(Map entity) {
			return (String)entity.get("$type$");
	}
```

知识准备 4 EntityIdentityInsertAction:对应由数据库生成关键字值id的insert操作。有可能立即执行，不需要等到flush或commit时。EntityInsertAction：对应由程序生成关键字的Insert操作，没机会立即执行。需等到flush或commit时，才执行。EntityUpdateAction:对应update操作。EntityDeleteAction:对应delete操作。 知识准备 5/** * Represents the status of an entity with respect to this session. These statuses are for internal * book -keeping only and are not intended to represent any notion that is visible to the \_application_. /public enum Status {
MANAGED,
READ_ONLY,
DELETED,
GONE,
LOADING,
SAVING} MANAGED和READ_ONLY：update时，将游离态对象转换为持久态时，需要将该Entity创建一个EntityEntry对象，并将其放入StatefulPersistenceContext中。
这时，将EntityEntry的Status标识为MANAGED和READ_ONLY。如果hbm中class的属性mutable设置true(默认为true，可变），则为
MANAGED，否则为READ_ONLY。DELETED：delete时，首先把EntityDeleteAction放入ActionQueue队列，但未执行，并将该Entity对应的EntityEntry的Status标识为DELETED。
表示已标示为删除，但未真正执行操作。GONE:当flush和commit时，EntityDeleteAction真正得到执行，当执行完之后，将EntityEntry的Status标识为GONE。SAVING：执行保存操作时，需要为Entity在StatefulPersistenceContext中创建一个EntityEntry，并将其Status标识为SAVING。LOADING:数据Load时的标识。 知识准备 6瞬时态、游离态、持久态、删除态的判断逻辑：1）如果Entity在StatefulPersistenceContext中存在相应的EntityEntry对象，且EntityEntry的Status为DELETED,那么判断为删除态（DELETED）；2）如果Entity在StatefulPersistenceContext中存在相应的EntityEntry对象，且EntityEntry的Status不为DELETED,那么判断为持久态（PERSISTENT）；3）如果Entity在StatefulPersistenceContext中不存在相应的EntityEntry对象，如果Entity关键字值为null或与hbm文件中id元素配置的unsaved-value
相等，那么判断为瞬时态（TRANSIANT）；4）如果Entity在StatefulPersistenceContext中不存在相应的EntityEntry对象，除3）外，否则判断为游离态（DETACHED）。

------------
参考


[hibernate文档](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/Session.html)
[stackoverflow关于persistence context的解释](http://stackoverflow.com/questions/19930152/what-is-persistence-context)
[Coin163
Hibernate 专题研究系列（一） save/update/saveOrUpdate等方法学习](http://amp.coin163.com/it/5624610946476426335)
