hibernate的hql  表一定要起别名

@Transient
忽略属性
报错 Can't create table 'mydb.#sql-1817b_79'
```java

```


```java
public List<Exam> getChosenExams(String studentId) {
		int sid=Integer.parseInt(studentId);
		String hql="SELECT e FROM SelectC sel,Exam e,Course c WHERE sel.cid = e.course.id AND c.id=sel.cid AND sel.sid =:sid";
		sessionFactory=config.buildSessionFactory();
		session=sessionFactory.openSession();
		List<Exam> exams=(List<Exam>)session.createQuery(hql).setParameter("sid",sid).getResultList();
		session.close();
		sessionFactory.close();
		return exams;
	}

	@Override
	public List<Exam> getTakenExams(String studentId) {
		int sid=Integer.parseInt(studentId);
		String hql="SELECT e FROM Exam e,Course c,Score s WHERE s.exam.id=e.id AND c.id=e.course.id AND s.student.id=:sid";
		sessionFactory=config.buildSessionFactory();
		session=sessionFactory.openSession();
		List<Exam> exams=(List<Exam>)session.createQuery(hql).setParameter("sid",sid).getResultList();
		session.close();
		sessionFactory.close();
		return exams;
	}
```
