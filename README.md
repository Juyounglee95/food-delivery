# 주제 - 도서관 시스템

도서관의 책 대여 및 예약, 관리 시스템입니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# 구현 Repository

총 5개
1. https://github.com/Juyounglee95/bookRental
2. https://github.com/Juyounglee95/gateway
3. https://github.com/Juyounglee95/bookManagement
4. https://github.com/Juyounglee95/gateway
5. https://github.com/Juyounglee95/view
# 서비스 시나리오

## 기능적 요구사항
1. 관리자는 도서를 등록한다.
2. 사용자는 도서를 예약한다.
3. 도서를 예약 시에는 포인트를 사용한다.
    3-1. 예약 취소 시에는 포인트가 반납된다.
4. 사용자는 도서를 반납한다.

## 비기능적 요구사항
1. 트랜잭션
    1. 결제가 되지 않은 경우 대여할 수 없다.  Sync 호출 
2. 장애격리
    1. 도서관리 기능이 수행되지 않더라도 대여/예약은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    2. 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 잠시후에 결제하도록 유도한다  Circuit breaker, fallback
3. 성능
    1. 사용자는 전체 도서 목록을 확인하여 전체 도서의 상태를 확인할 수 있어야한다. CQRS


# 체크포인트

- 분석 설계


  - 이벤트스토밍: 
    - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
      - 적어도 3개 이상 서비스 분리
    - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

  - 헥사고날 아키텍처
    - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
    
- 구현
  - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
    - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
    - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
    - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
  - Request-Response 방식의 서비스 중심 아키텍처 구현
    - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
    - 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
  - 이벤트 드리븐 아키텍처의 구현
    - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
    - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
    - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
    - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
    - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

  - 폴리글랏 플로그래밍
    - 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
    - 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
  - API 게이트웨이
    - API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
    - 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
- 운영
  - SLA 준수
    - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
    - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
    - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
    - 모니터링, 앨럿팅: 
  - 무정지 운영 CI/CD (10)
    - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명 
    - Contract Test :  자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?


# 분석/설계


* 이벤트스토밍 결과:  http://msaez.io/#/storming/nZJ2QhwVc4NlVJPbtTkZ8x9jclF2/every/a77281d704710b0c2e6a823b6e6d973a/-M5AV2z--su_i4BfQfeF


## 이벤트 도출
![image](https://user-images.githubusercontent.com/18453570/79930892-9c3cc800-8484-11ea-9076-39259368f131.png)


## 액터, 커맨드 부착하여 읽기 좋게
![image](https://user-images.githubusercontent.com/18453570/79931004-de660980-8484-11ea-9573-8cf3d8509e9e.png)

## 어그리게잇으로 묶기
![image](https://user-images.githubusercontent.com/18453570/79931210-6ea44e80-8485-11ea-959b-2f500a9a7c1d.png)


## 바운디드 컨텍스트로 묶기

![image](https://user-images.githubusercontent.com/18453570/79931545-32bdb900-8486-11ea-8518-558b5cf02d77.png)

    - 도메인 서열 분리 
        - Core Domain:  bookRental, bookManagement : 핵심 서비스
        - Supporting Domain:   marketing, customer : 경쟁력을 내기위한 서비스
        - General Domain:   point : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 (핑크색으로 이후 전환할 예정)

## 폴리시 부착

![image](https://user-images.githubusercontent.com/18453570/79933209-584cc180-848a-11ea-8289-c59468228c67.png)


## 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)

![image](https://user-images.githubusercontent.com/18453570/79933604-76ff8800-848b-11ea-8092-bd7510bf5d0b.png)

- View Model 추가

## 기능적/비기능적 요구사항을 커버하는지 검증

![image](https://user-images.githubusercontent.com/18453570/79933961-5f74cf00-848c-11ea-9870-cbd05b6348c5.png)

<기능적 요구사항 검증>

- 관리자는 도서를 등록한다.  ok
- 사용자는 도서를 예약한다.  ok
- 도서를 예약 시에는 포인트를 사용한다.  ok 
- 예약 취소 시에는 포인트가 반납된다.  ok
- 사용자는 도서를 반납한다.  ok
- 사용자는 예약을 취소할 수 있다 (ok)
- 예약이 취소되면 포인트가 반납되고, 도서의 상태가 예약 취소로 변경된다 (ok)
- 사용자는 도서상태를 중간중간 조회한다 (View-green sticker 의 추가로 ok)
- 도서가 등록/예약/예약취소/반납 시, 도서의 상태가 변경되어 전체 도서 리스트에 반영된다. 사용자와 관리자 모두 이를 확인할 수 있다. ok


## 비기능 요구사항에 대한 검증

- 마이크로 서비스를 넘나드는 시나리오에 대한 트랜잭션 처리
    - 도서 예약시 결제처리:  예약완료시 포인트 결제처리에 대해서는 Request-Response 방식 처리
    - 결제 완료시 도서 상태 변경:  Eventual Consistency 방식으로 트랜잭션 처리함.
    - 나머지 모든 inter-microservice 트랜잭션: 데이터 일관성의 시점이 크리티컬하지 않은 모든 경우가 대부분이라 판단, Eventual Consistency 를 기본으로 채택함.




## 헥사고날 아키텍처 다이어그램 도출  
    
![image](https://user-images.githubusercontent.com/18453570/80059618-5f95cd00-8567-11ea-9855-6fdc2e51bfd0.png)

- Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
- 호출관계에서 PubSub 과 Req/Resp 를 구분함
- 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현함. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)
bookManagement/  bookRental/  gateway/  point/  view/

```
cd bookManagement
mvn spring-boot:run

cd bookRental
mvn spring-boot:run 

cd gateway
mvn spring-boot:run  

cd point
mvn spring-boot:run

cd view
mvn spring-boot:run
```

## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언. 이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용함.

```
package library;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;

@Entity
@Table(name="PointSystem_table")
public class PointSystem {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private Long bookId;
    private Long pointQty =(long)100;

    @PostPersist
    public void onPostPersist(){
        PointUsed pointUsed = new PointUsed(this);
        BeanUtils.copyProperties(this, pointUsed);
        pointUsed.publish();


    }

    public Long getBookId() {
        return bookId;
    }

    public void setBookId(Long bookId) {
        this.bookId = bookId;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public Long getPointQty() {
        return pointQty;
    }

    public void setPointQty(Long pointQty) {
        this.pointQty = pointQty;
    }




}


```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package library;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface PointSystemRepository extends PagingAndSortingRepository<PointSystem, Long>{


}
```
- 적용 후 REST API 의 테스트
```
# bookManagement 서비스의 도서 등록처리
http POST http://52.231.116.117:8080/bookManageSystems bookName="JPA"

# bookRental 서비스의 예약처리
http POST http://52.231.116.117:8080/bookRentalSystems/returned/1

# bookRental 서비스의 반납처리
http POST http://52.231.116.117:8080/bookRentalSystems/reserve/1

# bookRental 서비스의 예약취소처리
http POST http://52.231.116.117:8080/bookRentalSystems/reserveCanceled/1

# 도서 상태 확인
http://52.231.116.117:8080/bookLists

```

## 동기식 호출 과 비동기식 

분석단계에서의 조건 중 하나로 예약(bookRental)->결제(point) 간의 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출하도록 한다. 

- 결제서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현 

```
# (app) pointSystemService.java

@FeignClient(name="point", url="http://52.231.116.117:8080")
public interface PointSystemService {

    @RequestMapping(method= RequestMethod.POST, path="/pointSystems", consumes = "application/json")
    public void usePoints(@RequestBody PointSystem pointSystem);

}

```

- 예약을 받은 직후(@PostPersist) 결제를 요청하도록 처리 -> BookRental의 생성은 BookManageSystem에서 도서를 등록한 직후 발생하기 때문에, Post요청으로 예약이 들어온 후 결제 요청하도록 처리함.

```
# BookRentalSystemController.java (Entity)

     @PostMapping("/bookRentalSystems/reserve/{id}")
     public void bookReserve(@PathVariable(value="id")Long id){
      PointSystem pointSystem = new PointSystem();
      pointSystem.setBookId(id);
      PointSystemService pointSystemService =  Application.applicationContext.
              getBean(library.external.PointSystemService.class);
      pointSystemService.usePoints(pointSystem);
  }
    }
```
결제가 이루어진 후에 도서대여시스템으로 이를 알려주는 행위는 동기식이 아니라 비 동기식으로 처리하여 도서대여시스템의 처리를 위하여 도서 상태 업데이트는 블로킹 되지 않도록 처리한다.
 
- 이를 위하여 결제이력에 기록을 남긴 후에 곧바로 결제승인이 되었다는 도메인 이벤트를 카프카로 송출한다(Publish)

```

#PointSystem.Java (Entity)
{
 @PostPersist
    public void onPostPersist(){
        PointUsed pointUsed = new PointUsed(this);
        BeanUtils.copyProperties(this, pointUsed);
        pointUsed.publish();


    }
}
```
결제 완료 이벤트를 도서대여시스템의 리스너가 받아, 도서의 상태를 예약완료로 변경한다.


```
(BookRentalSystem) PolicyHandler.JAVA
{
    @StreamListener(KafkaProcessor.INPUT) //포인트 결제 완료시
    public void wheneverPointUsed_ChangeStatus(@Payload PointUsed pointUsed){
        try {
            if (pointUsed.isMe()) {
                System.out.println("##### point use completed : " + pointUsed.toJson());
                BookRentalSystem bookRentalSystem = bookRentalSystemRepository.findById(pointUsed.getBookId()).get();
                bookRentalSystem.setBookStatus("Reserved Complete");
                bookRentalSystemRepository.save(bookRentalSystem);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```


도서상태가 변경되면, Reserved라는 이벤트를 발행한다.


```
BookRentalSystem.java (Entity)

{
   @PostUpdate
    public void bookStatusUpdate(){

        if(this.getBookStatus().equals("Returned")){
            Returned returned = new Returned(this);
            BeanUtils.copyProperties(this, returned);
            returned.publish();

        }else if(this.getBookStatus().equals("Canceled")){

            ReservationCanceled reservationCanceled = new ReservationCanceled(this);
            BeanUtils.copyProperties(this, reservationCanceled);
            reservationCanceled.publish();
        }else if(this.getBookStatus().equals("Reserved Complete")){
            Reserved reserved = new Reserved(this);
            BeanUtils.copyProperties(this, reserved);
            reserved.publish();
        }

    }
    
}
```


결과 : 포인트가 사용된 후에, 예약이 완료되는 것과 도서의 상태가 변경된 것을 BookListView확인 할 수 있다.

![image](https://user-images.githubusercontent.com/18453570/80061051-12b3f580-856b-11ea-989c-f4cf958613d5.png)




# 운영

## CI/CD 설정


각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 azure를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 azure-pipeline.yml 에 포함되었다.

## pipeline 동작 결과

아래 이미지는 azure의 pipeline에 각각의 서비스들을 올려, 코드가 업데이트 될때마다 자동으로 빌드/배포 하도록 하였다.

![image](https://user-images.githubusercontent.com/18453570/79945720-6b22be80-84a9-11ea-8465-132806bc0f97.png)

그 결과 kubernetes cluster에 아래와 같이 서비스가 올라가있는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/18453570/79971771-c2d42080-84cf-11ea-9385-0896baf668a4.png)

또한, 기능들도 정상적으로 작동함을 알 수 있다.

**<이벤트 날리기>**

![image](https://user-images.githubusercontent.com/18453570/80060143-cb2c6a00-8568-11ea-934a-111ccd8c21c9.png)
![image](https://user-images.githubusercontent.com/18453570/80060146-ce275a80-8568-11ea-993a-9f206ed4e7e8.png)
![image](https://user-images.githubusercontent.com/18453570/80060149-d089b480-8568-11ea-83ef-8a2496163806.png)
![image](https://user-images.githubusercontent.com/18453570/80060153-d2537800-8568-11ea-8c01-0a4740373c4a.png)
![image](https://user-images.githubusercontent.com/18453570/80060164-d5e6ff00-8568-11ea-8f75-b8e735ba7e18.png)

**<동작 결과>**

![image](https://user-images.githubusercontent.com/18453570/80060261-15ade680-8569-11ea-8256-d28b1e7f1e67.png)


### 오토스케일 아웃


- 포인트서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 15프로를 넘어서면 replica 를 10개까지 늘려준다:
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다:

![image](https://user-images.githubusercontent.com/18453570/80059958-51947c00-8568-11ea-9567-1b7d69c7381f.png)

- 워크로드를 2분 동안 걸어준 후 테스트 결과는 아래와 같다.

![image](https://user-images.githubusercontent.com/18453570/80060025-8274b100-8568-11ea-8f60-fa428c62168c.png)


## 무정지 재배포

Autoscaler설정과 Readiness 제거를 한뒤, 부하를 넣었다. 

이후 Readiness를 제거한 코드를 업데이트하여 새 버전으로 배포를 시작했다.

그 결과는 아래는 같다.

![image](https://user-images.githubusercontent.com/18453570/80060602-ec418a80-8569-11ea-87ea-34004c1ce5d3.png)
![image](https://user-images.githubusercontent.com/18453570/80060605-eea3e480-8569-11ea-9825-a375530f1953.png)


다시 Readiness 설정을 넣고 부하를 넣었다.

그리고 새버전으로 배포한 뒤 그 결과는 아래와 같다.


![image](https://user-images.githubusercontent.com/18453570/80060772-565a2f80-856a-11ea-9ee3-5d682099b899.png)
![image](https://user-images.githubusercontent.com/18453570/80060776-5823f300-856a-11ea-89a9-7c945ea05278.png)

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.

