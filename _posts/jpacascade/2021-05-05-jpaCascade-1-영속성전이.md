# JPA CASCADE

Description: 영속 전이
상위 태그: Language
수정일: 2021년 5월 31일 오전 4:25
하위 태그: Java

## 영속성 전이: CASCADE

---

영속성 전이(cascade)란 쉽게 말해 부모 엔티티가 영속화될때, 자식 엔티티도 같이 영속화되고 부모 엔티티가 삭제 될때, 자식 엔티티도 삭제되는 등 부모의 영속성 상태가 전이되는 것을 이야기합니다.

### 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장.

![_2021-05-21__1.47.51.png](/images/jpacascade/_2021-05-21__1.47.51.png)

### 저장

```java
@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)
```

![_2021-05-21__1.48.12.png](/images/jpacascade/_2021-05-21__1.48.12.png)

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐

## CASCADE의 종류

---

- **CascadeType.ALL**
모든 Cascade 적용
- **CascadeType.PERSIST**
엔티티를 영속화 할 때이 필드에 보유 된 엔티티도 유지합니다. EntityManager가 flush 중에 새로운 엔티티를 참조하는 필드를 찾고이 필드가 CascadeType.PERSIST를 사용하지 않으면 오류이므로이 Cascade 규칙의 자유로운 적용을 제안합니다.
- **CascadeType.REMOVE**
엔티티를 삭제할 때, 이 필드에 보유 된 엔티티도 삭제하십시오.
- **CascadeType.MERGE**
엔티티 상태를 병합 할 때, 이 필드에 보유 된 엔티티도 병합하십시오.
- **CascadeType.REFRESH**
엔티티를 새로 고칠 때, 이 필드에 보유 된 엔티티도 새로 고칩니다.
- **CascadeType.DETACH**
부모 엔티티가 detach()를 수행하게 되면, 연관된 엔티티도 detach() 상태가 되어 변경사항이 반영되지 않는다.

주로 사용하는 옵션은 ALL, PERSIST, REMOVE 를 주로 사용합니다.

### 예제)

예제를 위한 엔티티

Parent.class

```java
package hellojpa;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent" ,cascade = CascadeType.PERSIST)
    private List<Child> childList = new ArrayList<>();
}
```

Child.class

```java
package hellojpa;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}
```

### 영속성 전이 ( PERSIST )

```java
package hellojpa;

import java.util.ArrayList;
import java.util.function.Consumer;
import org.hibernate.Hibernate;
import sun.jvm.hotspot.debugger.win32.coff.TestDebugInfo;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {

            Parent parent = new Parent();
            parent.setName("상길");

            Child child1 = new Child();
            child1.setName("자식상길");
            child1.setParent(parent);

            List<Child> childList = new ArrayList<>();
            childList.add(child1);

            parent.setChildList(childList);

            em.persist(parent);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            e.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();

    }

}
```

**결과 )**

![_2021-05-31__3.35.54.png](/images/jpacascade/_2021-05-31__3.35.54.png)

부모의 엔티티만 persist 했지만 부모가 갖고있는 자식 엔티티까지 persist가 전이되는걸 확인할 수 있습니다.

### 영속성 전이 ( REMOVE )

```java
package hellojpa;

import java.util.ArrayList;
import java.util.function.Consumer;
import org.hibernate.Hibernate;
import sun.jvm.hotspot.debugger.win32.coff.TestDebugInfo;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
						Parent parent = em.find(Parent.class, 1L);
            em.remove(parent);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            e.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();

    }

}
```

**결과 )**

![_2021-05-31__3.42.10.png](/images/jpacascade/_2021-05-31__3.42.10.png)

부모의 엔티티만 remove 했지만 부모와 연관된 자식 엔티티까지 함께 삭제되는걸 볼 수 있습니다.



[PostIT]

- references:
  - [https://coding-start.tistory.com/159](https://coding-start.tistory.com/159)
  - [https://postitforhooney.tistory.com/entry/JavaJPAHibernate-CascadeType란-그리고-종류](https://postitforhooney.tistory.com/entry/JavaJPAHibernate-CascadeType%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%A2%85%EB%A5%98)
- Written by: 박상길 (fkdl3919@gmail.com)
- reporting date: 2021-05-16