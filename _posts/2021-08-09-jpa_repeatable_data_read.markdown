---
layout: post
title: JPA Session level repeatable read
description: JPA
date:   2021-08-09 23:11:00 +0530
categories: code
tags: [ Spring, JPA ]
author: easywaldo
comments: true
---

### 세션수준에서의 JPA 반복읽기모드
> JPA 가 DB로 부터 데이터를 읽어오는데에는 우선순위가 있다.

우선순위는 아래와 같다
- JPA 영속화 모델로 부터 데이터 조회 (조회된 데이터 없으면 다음 단계)
- JPA 2차 캐시에서 데이터 조회 (조회된 데이터 없으면 다음 단계)
- DB로 부터 조회

위와 같이 데이터를 JPA 에서는 조회를 하여 관리할 수 있다.


위의 우선순위를 확인 하기 위한 테스트 코드는 아래와 같다.

```java
@RestController
public class SampleController {
    private final MemberService memberService;
    (....)

    @ApiOperation(value = "회원 테스트 조회", notes = "")
    @GetMapping(value ="repeatableTest")
    public void repeatableReadTest() {
        Member member = this.memberService.findMember();
    }

    @ApiOperation(value = "unCommittedTest Test", notes = "")
    @GetMapping(value ="unCommittedTest")
    public void unCommittedTest() throws InterruptedException {
        this.memberService.findMemberV2();
    }

    @ApiOperation(value = "unCommittedTest Test", notes = "")
    @GetMapping(value ="unCommittedReadTest")
    public void unCommittedReadTest() {
        this.memberService.findMemberV3();
    }

    @Service
    public class DocumentQueryGenerator {
        (....)
 
    @Transactional(transactionManager = "easyTransactionManagerFactory", readOnly = false, isolation = Isolation.READ_UNCOMMITTED)
        public Member findMember() {
            Member member1 = this.memberRepository.findById(1).get();
            Member member2 = this.memberRepository.findByUserId("easywaldo").get();
            Member member3 = this.queryGenerator.findMemberTest();
            Member member4 = this.memberRepository.findById(1).get();
            Member member5 = this.memberRepository.jpqlFindUserId("easywaldo").get();
            Member member6 = this.queryGenerator.findMemberTest();
            Member member7 = this.queryGenerator.findMemberTestV2();
            Member member8 = this.queryGenerator.findMemberTestV2();
            Member member9 = this.queryGenerator.findMemberQuery();
            Member member10 = this.queryGenerator.findMemberQuery();
            return member3;
        }

        @Transactional(transactionManager = "easyTransactionManagerFactory", readOnly = false, isolation = Isolation.READ_UNCOMMITTED)
        public Member findMemberV2() throws InterruptedException {
            Member member1 = this.memberRepository.findById(1).get();
            member1.update("수정된 이름");
            entityManager.persist(member1);
            entityManager.setFlushMode(FlushModeType.COMMIT);
            entityManager.flush();
            System.out.println(String.format("findMemberV2 memberName: %s", member1.getMemberName()));
            findMemberV4();
            Thread.sleep(20000);
            return member1;
        }

        @Transactional(transactionManager = "easyTransactionManagerFactory", readOnly = false, isolation = Isolation.READ_COMMITTED)
        public Member findMemberV3() {
            Member member1 = this.memberRepository.findById(1).get();
            Member member2 = this.queryGenerator.findMemberTest();
            Member member3 = this.entityManager.find(Member.class, 1);
            System.out.println(String.format("findMemberV3 memberName: %s", member1.getMemberName()));
            System.out.println(String.format("findMemberV3 memberName: %s", member2.getMemberName()));
            System.out.println(String.format("findMemberV3 memberName: %s", member3.getMemberName()));
            return member1;
        }

        public Member findMemberV4() {
            Member member1 = this.memberRepository.findById(1).get();
            System.out.println(String.format("findMemberV4 memberName: %s", member1.getMemberName()));
            return member1;
        }

        public Member findMemberTest() {
            Member result = (Member)this.entityManager.find(Member.class, 1);
            return result;
        }

        public Member findMemberTestV2() {
            Member result = (Member)this.entityManager.createQuery(
                "select m from Member m where m.userId = 'easywaldo'").getSingleResult();
            return result;
        }

        public Member findMemberQuery() {
            var result = queryFactory.from(QMember.member)
                .where(QMember.member.memberSeq.eq(1))
                .select(Projections.fields(Member.class,
                    QMember.member.memberSeq,
                    QMember.member.memberName,
                    QMember.member.userId,
                    QMember.member.userPwd
                    ))
                .setLockMode(LockModeType.WRITE).fetchOne();

            result.update(String.format("%s-modifiedName", result.getMemberName()));

            return result;
        }


    @Getter
    @Entity
    @Table(name = "member")
    @NoArgsConstructor
    public class Member {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "member_seq")
        private Integer memberSeq;

        @Column(name = "member_name")
        private String memberName;

        @Column(name = "user_id")
        private String userId;

        @Column(name = "user_pwd")
        private String userPwd;


    //JpaRepository 상속 인터페이스
    public interface MemberRepository extends JpaRepository<Member, Integer> {
        Optional<Member> findByUserId(String userId);
        @Query(value = "select m from Member m where m.userId = ?1")
        Optional<Member> jpqlFindUserId(String userId);
    }

```

> 먼저 세션 수준에서의 반복읽기가 아닌 다른 세션간에 반복읽기가 되는지 여부를 확인하기 위해 findMemberV2 메서드안에 스레드대기를 20초 설정을 하였다.

> findMember() 메서드에서는JPA findById, EntityManager 의 find, QueryDSL 의 fetch 등의 각각의 메서드가 JPA 안에서 어떻게 관리가 되는 것인지 확인해 볼 수 있도록 각각의 메서드들을 순차적으로 실행을 하도록 하였다.


```
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as member_s1_2_0_, member0_.member_name as member_n2_2_0_, member0_.user_id as user_id3_2_0_, member0_.user_pwd as user_pwd4_2_0_ from member member0_ where member0_.member_seq=?
23:42:51 TRACE o.h.type.descriptor.sql.BasicBinder - binding parameter [1] as [INTEGER] - [1]
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as member_s1_2_, member0_.member_name as member_n2_2_, member0_.user_id as user_id3_2_, member0_.user_pwd as user_pwd4_2_ from member member0_ where member0_.user_id=?
23:42:51 TRACE o.h.type.descriptor.sql.BasicBinder - binding parameter [1] as [VARCHAR] - [easywaldo]
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as member_s1_2_, member0_.member_name as member_n2_2_, member0_.user_id as user_id3_2_, member0_.user_pwd as user_pwd4_2_ from member member0_ where member0_.user_id=?
23:42:51 TRACE o.h.type.descriptor.sql.BasicBinder - binding parameter [1] as [VARCHAR] - [easywaldo]
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as member_s1_2_, member0_.member_name as member_n2_2_, member0_.user_id as user_id3_2_, member0_.user_pwd as user_pwd4_2_ from member member0_ where member0_.user_id='easywaldo'
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as member_s1_2_, member0_.member_name as member_n2_2_, member0_.user_id as user_id3_2_, member0_.user_pwd as user_pwd4_2_ from member member0_ where member0_.user_id='easywaldo'
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as col_0_0_, member0_.member_name as col_1_0_, member0_.user_id as col_2_0_, member0_.user_pwd as col_3_0_ from member member0_ where member0_.member_seq=?
23:42:51 TRACE o.h.type.descriptor.sql.BasicBinder - binding parameter [1] as [INTEGER] - [1]
23:42:51 DEBUG org.hibernate.SQL - select member0_.member_seq as col_0_0_, member0_.member_name as col_1_0_, member0_.user_id as col_2_0_, member0_.user_pwd as col_3_0_ from member member0_ where member0_.member_seq=?
23:42:51 TRACE o.h.type.descriptor.sql.BasicBinder - binding parameter [1] as [INTEGER] - [1]
```


위의 로그를 확인해보면 총 7개의 쿼리가 Fetch 되었음을 확인할 수 있다.

이유는 이와 같다.
1. findById(1) 로 Member 에 대한 최초 조회 (영속성 관리 대상)
2. findByUserId 로 Query by method 로 조회 (영속성 관리 되지 
않음 >> 직접 조회)
3. EntityManager 클래스의 find 메서드로 데이터 조회 (이미 1에 대한 영속화 데이터가 존재하므로 Fetch 실행 안함)
4. findById(1) 로 데이터 조회 (영속화 존재하므로 Fetch 실행 안함)
5. Jpql QueryAnnotation 을 활용한 select m from Member m where m.userId = ?1" 쿼리를 Fetch 실행
6. EntityManager 클래스의 find 메서드로 데이터 조회 (이미 1에 대한 영속화 데이터가 존재하므로 Fetch 실행 안함)
7. EntityManager createQuery  메서드로 Fetch 실행
8. EntityManager createQuery  메서드로 Fetch 실행
9. QueryDSL fetch 실행
10. QueryDSL fetch 실행

이 중 영속화 대상에서 존재하여 repeatable 이 되는 과정은
3번, 4번, 6번이다.

따라서 위와 같이 7개의 쿼리가 콘솔에서 출력이 되는 것이다.


> JPQL 메서드는 영속화 컨텍스트를 거치지 않고 직접 쿼리 실행을 한다.


