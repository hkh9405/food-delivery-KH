![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 배달의민족 서비스 by.황경환

# 서비스 시나리오

기능적 요구사항
- 고객이 메뉴를 선택하여 주문한다.
- 고객이 선택한 메뉴에 대해 결제한다.
- 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다.
- 상점주는 주문을 수락하거나 거절할 수 있다.
- 상점주가 주문을 수락하고 요리 완료 예정시간을 설정하면 가장 가까운 라이더를 조회하여 완료시간에 배달을 시작한다. (수정)
- 고객은 상점주가 수락하지 않은 주문에 대해 주문취소할 수 있다. (수정)
- 라이더가 배달을 수락하면 해당 라이더에게 배달이 배정되며 배달 정보를 볼 수 있다. (수정)
- 라이더가 해당요리를 pick한 후, 앱을 통해 통보한다.
- 고객이 주문상태를 중간중간 조회한다.
- 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다.
- 라이더가 배송이 완료되면 배송완료 버튼을 탭하여 모든 거래가 완료되며 고객과 상점주에게 배달완료상태가 보이게 된다. (수정)

비기능적 요구사항
1. 트랜잭션
    1. 결제가 되지 않은 주문건은 아예 거래가 성립되지 않아야 한다  Sync 호출 
1. 장애격리
    1. 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다  Circuit breaker, fallback
1. 성능
    1. 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다  CQRS
    1. 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다  Event driven


# 이벤트 스토밍
![image](https://user-images.githubusercontent.com/80162639/207079349-783440b2-31aa-408a-abf4-f14581307462.png)


# 요구사항 검증
![image](https://user-images.githubusercontent.com/80162639/207083558-d235d24b-6441-4ca2-aad8-701b5aa79fa4.png)

1. 고객이 메뉴를 선택하여 주문한다.(완료)
2. 고객이 선택한 메뉴에 대해 결제한다. (완료)
3. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다. (완료)

![image](https://user-images.githubusercontent.com/80162639/207084913-c9d6583c-1b98-432f-9b06-0e786ed3bb1b.png)

4. 상점주는 주문을 수락하거나 거절할 수 있다.
5. 상점주가 주문을 수락하고 요리 완료 예정시간을 설정하면 가장 가까운 라이더를 조회하여 완료시간에 배달을 시작한다. (수정)
6. 고객은 상점주가 수락하지 않은 주문에 대해 주문취소할 수 있다. (수정)

![image](https://user-images.githubusercontent.com/80162639/207087930-10f62b55-83c7-4006-85ee-875074c697b5.png)

7. 라이더가 배달을 수락하면 해당 라이더에게 배달이 배정되며 배달 정보를 볼 수 있다. (수정)
8. 라이더가 해당요리를 pick한 후, 앱을 통해 통보한다.
9. 고객이 주문상태를 중간중간 조회한다.
10. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다.
11. 라이더가 배송이 완료되면 배송완료 버튼을 탭하여 모든 거래가 완료되며 고객과 상점주에게 배달완료상태가 보이게 된다. (수정)


# 체크포인트
## 1. Saga (Pub / Sub)
*음식 주문처리
order에서 신규 주문을 발생시키면 pay를 거쳐 orderManage까지 연동되어 데이터가 흐르는 것을 확인할 수 있었다.
```
gitpod /workspace/food-delivery (main) $ http :8081/orders orderId="a123" custId="hkh9405" custAddr="불정로 10" custTel="010-1234-5678" orderInfo="김치찌개:1,제육볶음:2" totPrice=18000
HTTP/1.1 201 
Connection: keep-alive
Content-Type: application/json
Date: Mon, 12 Dec 2022 08:20:31 GMT
Keep-Alive: timeout=60
Location: http://localhost:8081/orders/1
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_links": {
        "order": {
            "href": "http://localhost:8081/orders/1"
        },
        "self": {
            "href": "http://localhost:8081/orders/1"
        }
    },
    "orderId": "a123",
    "custId": "hkh9405",
    "custAdd"r: "불정로 10",
    "custTel": "010-1234-5678",
    "orderInfo": "김치찌개:1,제육볶음:2",
    "totPrice": 18000,
    "status": null
}

gitpod /workspace/food-delivery (main) $ http :8082/pays
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Mon, 12 Dec 2022 08:22:24 GMT
Keep-Alive: timeout=60
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_embedded": {
        "pays": [
            {
                "_links": {
                    "pay": {
                        "href": "http://localhost:8082/pays/1"
                    },
                    "self": {
                        "href": "http://localhost:8082/pays/1"
                    }
                },
                "orderId": "a123",
                "custId": "hkh9405",
                "custAdd"r: "불정로 10",
                "custTel": "010-1234-5678",
                "orderInfo": "김치찌개:1,제육볶음:2",
                "totPrice": 18000,
                "status": null
            }
        ]
    },
    "_links": {
        "profile": {
            "href": "http://localhost:8082/profile/pays"
        },
        "self": {
            "href": "http://localhost:8082/pays"
        }
    },
    "page": {
        "number": 0,
        "size": 20,
        "totalElements": 1,
        "totalPages": 1
    }
}

gitpod /workspace/mall (main) $ http :8083/orderManages
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Mon, 12 Dec 2022 08:25:16 GMT
Keep-Alive: timeout=60
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_embedded": {
        "orderManages": [
            {
                "_links": {
                    "orderManage": {
                        "href": "http://localhost:8083/orderManages/1"
                    },
                    "self": {
                        "href": "http://localhost:8083/orderManages/1"
                    }
                },
                "orderId": "a123",
                "custId": "hkh9405",
                "custAdd"r: "불정로 10",
                "custTel": "010-1234-5678",
                "orderInfo": "김치찌개:1,제육볶음:2",
                "totPrice": 18000,
                "status": "orderCmplt"
            }
        ]
    },
    "_links": {
        "profile": {
            "href": "http://localhost:8083/profile/orderManages"
        },
        "self": {
            "href": "http://localhost:8083/orderManages"
        }
    },
    "page": {
        "number": 0,
        "size": 20,
        "totalElements": 1,
        "totalPages": 1
    }
}
```

## 2. CQRS
이벤트가 발생 시 고객이 볼 수 있는 OrderInfo, 점주가 볼 수 있는 OrderDtl, 라이더가 볼 수 있는 DeliveryInfo가 변경될 수 있게 하였다.

*OrderInfo

![image](https://user-images.githubusercontent.com/80162639/207140523-290c1096-8097-4d0e-accb-c4e5f23f8258.png)

![image](https://user-images.githubusercontent.com/80162639/207141059-6844e5dc-9f21-4243-b0bb-9b56977126c3.png)

create는 결재 완료시 상태가 "oderCmplt"로 변경

![image](https://user-images.githubusercontent.com/80162639/207141169-76cbb880-c635-496c-8779-463374880284.png)

![image](https://user-images.githubusercontent.com/80162639/207141271-a29b8fef-aa64-430b-911b-d8b269f2d83a.png)

위에서 부터 각각 주문승인, 결재취소, 주문취소, 배달시작, 배달완료 등의 이벤트가 발생시 상태값이 변경될 수 있도록 세팅



*OrderDtl

![image](https://user-images.githubusercontent.com/80162639/207144432-30b79845-ff0b-49b5-bd73-3441f0707a1b.png)

![image](https://user-images.githubusercontent.com/80162639/207144639-879c7c04-03ca-4b29-8d3b-e39a763b17ea.png)

결재 완료 시 점주에게 주문내용이 보일 수 있도록 함

![image](https://user-images.githubusercontent.com/80162639/207144694-70defef9-7bf1-4b29-a1d3-5dc71ef1a317.png)

배달시작과 배달완료 시 상태값이 변경될 수 있도록 세팅



## 3.Compensation / Correlation

Order 클래스에 PreRemove 어노테이션을 사용한 메소드를 구현

```
@PreRemove
public void onPreRemove(){
    if (!status.equals("orderCmplt")) {
        throw new RuntimeException("주문 승인이후에는 주문취소가 불가능합니다.");
    }

    // 주문취소 이벤트 pub
    OrderCanceled orderCanceled = new OrderCanceled(this);
    orderCanceled.publishAfterCommit();
}
```

PolicyHandler에서 오더취소를 통해 취소가능

```
@StreamListener(value=KafkaProcessor.INPUT, condition="headers['type']=='OrderCxl'")
    public void wheneverOrderCxl_OrderChg(@Payload OrderCxl orderCxl){

        OrderCxl event = orderCxl;
        System.out.println("\n\n##### listener OrderChg : " + orderCxl + "\n\n");

        // Sample Logic //
        Order.orderChg(event);

    }
```

## 4. Request / Response

Order 서비스에서 주문 완료시 Pay 서비스로 동기호출을 하게 되는데 이 때 양쪽의 서버가 모두 열려있지 않으면 에러가 발생한다. 이는 두 서버가 모두 처리가 되어야 하기 떄문에 발생

```
gitpod /workspace/food-delivery (main) $ http POST localhost:8081/orders orderId="a123" custId="hkh9405" custAddr="불정로 10" custTel="010-1234-5678" orderInfo="김치찌개:1,제육볶음:2" totPrice=18000
HTTP/1.1 500 
Connection: close
Content-Type: application/json
Date: Mon, 12 Dec 2022 20:41:21 GMT
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "error": "Internal Server Error",
    "message": "",
    "path": "/orders",
    "status": 500,
    "timestamp": "2022-12-12T20:41:21.235+00:00"
}
```

## 5. Circuit Breaker

Circuit Breaker 설정

```
feign:
  hystrix:
    enabled: true
    
hystrix:
  command:
    default:
      execution.isolation.thread.timeoutInMilliseconds: 1000
```

결과

```
gitpod /workspace/food-delivery (main) $ siege -c2 -t10S  -v --content-type "application/json" 'http://localhost:8081/orders POST {"orderId":"1234"}'
** SIEGE 4.0.4
** Preparing 2 concurrent users for battle.
The server is now under siege...
HTTP/1.1 500     0.44 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.44 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.05 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.06 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.06 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.05 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.02 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.05 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.05 secs:     261 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 500     0.01 secs:     261 bytes ==> POST http://localhost:8081/orders
```

## 6. Gateway / Ingress

```
spring:
  profiles: default
  cloud:
    gateway:
      routes:
        - id: order
          uri: http://localhost:8081
          predicates:
            - Path=/menus/**, /orderInfos/**, /orders/**,
        - id: pay
          uri: http://localhost:8082
          predicates:
            - Path=/pays/**, 
        - id: shop
          uri: http://localhost:8083
          predicates:
            - Path=/orderDtls/**, /orderManages/**,
        - id: delivery
          uri: http://localhost:8084
          predicates:
            - Path=/deliveryInfos/**, /deliveries/**, /cmpltTrts/**,
        - id: custHelp
          uri: http://localhost:8085
          predicates:
            - Path=/**, 
        - id: frontend
          uri: http://localhost:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true
```
