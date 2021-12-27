---
title: '이고브프레임워크를 인텔리제이에서'
date: 2021-05-05T00:19:00+09:00
categories:
    - EGOVEFRAMEWORK
share: false
---

# 페이지 목차

1. [이고브프레임워크를 인텔리제이에서]

## EgoveFramework를 인텔리제이에서

---

-   안녕하세요. 이번에 회사에서 국가사업 개발을 맞게되었습니다. egoveframework를 사용해야하는데, 저희는 intellij를 사용하고있어 intellij를 사용하여 egoveframework 개발을 시작하였습니다. 하여 해당 프로잭트 진행중 처음부터 진행시 에러사항에 대해 포스팅 예정입니다. 우선 처음 intellij에 egoveproject 환경을 구성하는 과정입니다.

#프로젝트 생성
우선 프로젝트를 생성합니다.
![1-1](/images/egove/2021-05-16-egove-egoveframework1.PNG)

다음으로 ProjectSettings - Project 에서 sdk 버전과 language level을 설정합니다.
![1-2](/images/egove/2021-05-16-egove-egoveframework2.PNG)
그다음 Modules 로 이동후 (+) 버튼을 눌러 new Module를 선택합니다. 그후 Next버튼을 선택하고 GroupId와 ArtifactId를 입력후 Finish합니다. 그렇게하면 Maven 프로젝트가 생성됩니다.
![1-3](/images/egove/2021-05-16-egove-egoveframework3.PNG)
![1-4](/images/egove/2021-05-16-egove-egoveframework4.PNG)
![1-5](/images/egove/2021-05-16-egove-egoveframework5.PNG)

그리고 egoveframework 공식홈페이지에 접속해 다운로드 -> 공통컴포넌트 -> 다운로드를 통해 예제 공통 컴포넌트를 다운로드합니다.

![1-6](/images/egove/2021-05-16-egove-egoveframework6.PNG)
![1-7](/images/egove/2021-05-16-egove-egoveframework7.PNG)

JPA를 활용하여 개발 중 업데이트 후 select 시 update 된 값이 반영되지 않는 문제가 발생하였습니다.

```java
	@Transactional(propagation = Propagation.REQUIRED)
	public void updateTubeState(List<Tube> tubes) {
		for(Tube tube:tubes) {
			if(tube.getTubeState().getCodeId().equals("STATE-1")) {
				tubeService.updateTubeStateFail(tube);
			}else {
				tubeService.updateTubeState(tube);
			}
			Tube param = tubeService.findById(tube.getId());
			makeTubeLog(param);
		}
	}
```

-   다음과 같은 이유가 뭘까를 찾기 위해 테스트코들을 작성하고 진행해보았습니다. 테스트를 위해 테스트 테이블 생성 후 다음과 같은 방법으로 진행해 보았습니다.

```java
    @Table(name = "TEST")
    public class Test  {

        public Test(Long id) {
            this.id = id;
        }

        @Id
        @Column(name = "ID", nullable = false)
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column(name = "TEST_TITLE", nullable = false, length = 200)
        private String testTitle;


    }
	@Transactional
	public void test(Test test) {
		testRepository.updateSubject(test,"change11");
		Optional<Test> result = testRepository.findById(test.getId());
		System.out.print(result.get().getTestTitle());
	}

```

-   위와 같이 실험을 하니, 정상적으로 select 된 코드가 출력되었습니다. 문제가 무엇일까?? 그럼 기존 코드랑 테스트 코드랑 다른 점은 무엇일까?? 바로 영속성이었습니다. 테스트 코드는 영속성이 아닌 비영속상태의 객체를 생성하여 테스트하였기에 문제없이 진행되었습니다. 그럼 개발 중 오류 상태와 동일하게 만들기 위해 영속성을 만든 후 테스트해 보았습니다.

```java
	@Transactional
	public void test(Test test) {
		test = testRepository.findById(test.getId()).get();
		testRepository.updateSubject(test,"change11");
		Optional<Test> result = testRepository.findById(test.getId());
		System.out.print(result.get().getTestTitle());
	}

```

-   테스트 결과 업데이트한 subject가 findbyid를 통해 select시 출력되지 않았습니다.
    로그 확인 결과 다음과 같은 결과가 나왔습니다.

```log
    Hibernate: UPDATE TEST SET TEST_TITLE =? WHERE ID= ?
```

-   select 쿼리를 execute 하지 않았습니다. 아 그러면 select 문을 execute만 해주면 되겠구나. 생각하여 findbyid가 아닌 직접 nativequery select 코드를 만들어 select 해주었습니다.

```java
	@Transactional
	public void test(Test test) {
		test = testRepository.findById(test.getId()).get();

		testRepository.updateSubject(test,"change11");
		Optional<Test> result = testRepository.select(test.getId());
		System.out.print(result.get().getTestTitle());
	}

```

```log
    Hibernate: UPDATE TEST SET TEST_TITLE =? WHERE ID= ?
    Hibernate: SELECT * FROM TEST WHERE ID= ?
    before
```

-   log 결과 select를 해주었지만, return 된 값은 업데이트되지 않은 값이었습니다.
    문제가 무엇일까 고민하고 search 결과 다음과 같은 결론에 도달하였습니다.

조회한 Entity에 이미 영속성이 존재한다면, query를 통해 조회한 결과를 버리고 영속성의 기존 결과를 반환한다.

-   이러한 문제를 해결하기 위한 방법으로는 2가지 방법을 적용해보았습니다. 1)영속중인 객체에 직접 값을 업데이트 해주는 방법

```java
	@Transactional
	public void test(Test test) {
		test = testRepository.findById(test.getId()).get();

		testRepository.updateSubject(test,"change11");
		test.setTestTitle("change11");
		//Optional<Test> result = testRepository.select(test.getId());
		System.out.print(result.get().getTestTitle());
	}

```

```log
    Hibernate: UPDATE TEST SET TEST_TITLE =? WHERE ID= ?
    change11
```

-   2. 영속성을 해제해 주는 방법
       @Modifying(clearAutomatically = true)
       UPDATE 쿼리에 @Modifying의 clearAutomatically를 true해주면 영속성을 clear 해줍니다.

```java
    @Transactional
	public void test(Test test) {
		test = testRepository.findById(test.getId()).get();

		testRepository.updateSubject(test,"change11");
		Optional<Test> result = testRepository.select(test.getId());
		System.out.print(result.get().getTestTitle());
	}

    @Modifying(clearAutomatically = true)
	@Query(value = "UPDATE TEST SET TEST_TITLE =:subject WHERE ID= :#{#test.id}", nativeQuery = true)
	void updateSubject(Test test, String subject);

```

```log
    Hibernate: UPDATE TEST SET TEST_TITLE =? WHERE ID= ?
    Hibernate: SELECT * FROM TEST WHERE ID= ?
    change11
```

-   @Modifying 어노테이션의 경우 영속성을 해제해줘, select 시 영속성이 없어 현재 DB값을 select 하여 받아옵니다.
    다만 영속성을 해제하는 문제는 JPA의 장점인 영속성을 clear 하는 것이기에 필요한 경우에만 사용하면 좋을 것 같습니다.
    두 가지 경우의 방법은 다르지만, 결과는 같습니다. 적절히 사용해주면 좋을 것 같습니다.

-   references: [https://gmlwjd9405.github.io/2019/08/06/persistence-context.html](https://gmlwjd9405.github.io/2019/08/06/persistence-context.html), [https://velog.io/@modsiw/JPAJava-Persistence-API%EC%9D%98-%EA%B0%9C%EB%85%90](https://velog.io/@modsiw/JPAJava-Persistence-API%EC%9D%98-%EA%B0%9C%EB%85%90)
-   Written by: 이재봉 (jblee6110@gmail.com)
-   reporting date: 2021-05-05
