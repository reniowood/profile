---
layout: post
title: "[Akka 코딩공작소] Chatper 3. 액터를 사용한 테스트 주도 개발"
categories: Akka
---

## 액터 테스트하기

액터를 테스트하기 위해 ScalaTest를 사용해 테스트를 작성한다.

Actor를 테스트하기 더 어려운 이유
* 단언문(assert)를 넣을 위치를 정하기 어렵다.
* 여러 스레드에서 병렬로 작동하는 시스템을 블록하지 않으며 테스트하기 까다롭다.
* 액터가 내부 상태를 감추고 있어 상태 확인이 어렵다.
* 통합 테스트를 할 때 여러 액터가 주고받는 메시지를 모두 확인해야 한다.

아카는 액터 테스트를 훨씬 쉽게 만들어주는 akka-testkit 모듈을 제공한다. 해당 모듈을 이용해 다음과 같은 테스트를 수행할 수 있다.
* 단일 스레드 단위 테스트
    * TestActorRef를 사용해 액터 인스턴스에 직접 접근할 수 있다.
* 다중 스레드 단위 테스트
    * TestKit과 TestProbe 클래스를 사용해
        * 액터에게 응답을 받거나
        * 메시지를 점검하거나
        * 메시지 도착 시간을 제한하는 등을 해볼 수 있다.
* 다중 JVM 테스트

TestKit을 사용하는 다중 스레드 스타일이 프로덕션에서 실행할 코드의 실제 환경과 가장 가까울 것이다.

## 단방향 메시지

액터를 세 가지 기준으로 나누어 테스트 할 수 있다.

* 액터의 동작을 밖에서 직접 관찰할 수 없는 경우
* 액터가 받은 메시지를 처리한 다음 다른 액터에게 메시지를 보내는 경우
* 액터가 메시지를 받으면 일반 객체와 정해진 방식으로 상호 작용하는 경우 

## 액터의 동작을 밖에서 직접 관찰할 수 없는 경우

단일 스레드 테스트

* TestActorRef를 이용해 테스트하려는 액터를 생성한다.

```scala
object TestActor {
    case class Message(data: String)
    case class GetState(receiver: ActorRef)
}

class TestActor extends Actor {
    import TestActor._

    val internalState = Vector[String]() // 외부에서 확인 불가

    def receive = {
        case Message(data) => internalState = internalState :+ data
        case GetState(receiver) => receiver ! internalState
    }

    def state = internalState
}
```

* underlyingActor를 이용해 TestActorRef로 생성한 액터에 접근 가능하다.

```scala
class ActorTest01 extends TestKit(ActorSystem("testSystem"))
    with WordSpecLike
    with MustMatchers
    with StopSystemAfterAll {

    "단일 스레드 환경에서 메시지를 받으면 내부 상태를 변경한다" in {
        import TestActor._

        val testActor = TestActorRef[TestActor]

        testActor ! Message("메시지 전송")
        testActor.underlyingActor.state must (contains("메시지 전송"))
    }
}
```

다중 스레드 테스트

* TestKit이 제공하는 ActorSystem을 사용해 테스트하려는 액터를 생성한다.
    * 액터 타입을 Props의 타입 인자로 지정해 만든 객체를 사용한다.
* 상태 변화를 알아내기 위해 직접 접근하지 않고 TestKit의 testActor를 사용한다.
    * 같은 actor를 매번 같은 thread가 처리한다는 보장이 없으므로 내부 변수를 들여다볼 수 없다.
    * testActor로 메시지를 전달하게 하면 내부 상태를 알아낼 수 있다.

```scala
class ActorTest02 extends TestKit(ActorSystem("testSystem"))
    with WordSpecLike
    with MustMatchers
    with StopSystemAfterAll {

    "다중 스레드 환경에서 메시지를 받으면 내부 상태를 변경한다" in {
        import TestActor._

        val actor = system.actorOf(Props[TestActor], "testActor")
        actor ! Message("첫번째 메시지 전송")
        actor ! Message("두번째 메시지 전송")
        actor ! GetState(testActor) // TestKit의 testActor가 메시지를 받게 한다.
        expectMsg(Vector("첫번째 메시지 전송", "두번째 메시지 전송"))
    }
}
```

### 액터가 받은 메시지를 처리한 다음 다른 액터에게 메시지를 보내는 경우

테스트 작성

```scala
"메시지를 받아 전송하는 액터" must {
    "메시지를 받아 처리해서 다른 액터에게 메시지를 보낸다" in {
        import SortingActor._

        val props = SortingActor.props(testActor)
        val sortingActor = system.actorOf(props, "sortingActor")

        // 정렬되지 않은 리스트를 전송한다.
        val unsortedList = (0 until 10000).map { _ =>
            Random.nextInt(100000)
        }.toVector
        sortingActor ! Sort(unsortedList)

        // expectMsgPF 함수를 사용해 정확하게 매치할 수 없는 결과를 확인한다.
        expectMsgPF() {
            case Sorted(sortedList) =>
                sortedList.size must be(size)
                unsortedList.sorted must be (sortedList)
        }
    }
}
```

받은 메시지를 처리해 다른 액터에게 결과를 담은 메시지를 보내는 SortingActor를 구현한다.

```scala
object SortingActor {
    def props(receiver: ActorRef) = Props(new SortingActor(receiver))

    case class Sort(unsortedList: Vector[Int])
    case class Sorted(sortedList: Vector[Int])
}

class SortingActor(receiver: ActorRef) extends Actor {
    import SortingActor._
    def receive = {
        case Sort(unsortedList) => receiver ! Sorted(unsortedList.sorted)
    }
}
```

액터가 보낸 메시지의 수나 순서도 테스트할 수 있다.
* receiveWhile 함수를 사용하면 case 문이 매치되지 않을 때까지 받은 메시지를 수집한다.
* expectNoMsg 함수로 타임아웃 시간 이전에 아무 메시지도 도착하지 않았다는 것을 확인할 수 있다.

테스트 작성

```scala
"receiveWhile과 expectNoMsg를 이용해 짝수일 때만 메시지를 보내는지 확인한다" in {
    import FilteringActor._

    val props = FilteringActor.props(testActor)
    val filter = system.actorOf(props, "filter")

    filter ! Number(1)
    filter ! Number(2)
    filter ! Number(3)
    filter ! Number(4)
    filter ! Number(5)
    filter ! Number(6)
    val numbers = receiveWhile() {
        case Number(number) if number < 6 => number
    }
    numbers numst be(List(2, 4))
    expectMsg(Number(6))

    filter ! Number(1)
    expectNoMsg
    filter ! Number(2)
    expectMsg(Number(2))
    filter ! Number(3)
    expectNoMsg
    filter ! Number(4)
    expectMsg(Number(4))
}
```

액터 구현

```scala
object FilteringActor {
    def props(nextActor: ActorRef) = Props(new FilteringActor(nextActor))
    case class Number(number: Long)
}

class FilteringActor(nextActor: ActorRef) extends Actor {
    import FilteringActor._

    def receive = {
        case msg: Number =>
            if (msg.number % 2 == 0) {
                nextActor ! msg
            }
    }
}
```

둘 이상의 testActor가 필요한 경우에는 TestProbe 클래스를 사용한다.
* 클래스를 확장하지 않고 TestProbe()를 호출해서 새로운 TestProbe를 만든다.

### 액터가 메시지를 받으면 일반 객체와 정해진 방식으로 상호 작용하는 경우

액터가 ```akka.logger```를 이용해 로그를 남기는 지 테스트한다.

테스트 작성

```scala
object LoggingActorTest {
    val testSystem = {
        // ConfigFactory는 문자열을 파싱해서 설정 정보를 얻을 수 있다.
        val config = ConfigFactory.parseString(
            // TestEventListener는 로그에 기록되는 모든 이벤트를 처리할 수 있다.
            """
                akka.loggers = [akka.testkit.TestEventListener]
            """
        )
    }
}

import LoggingActorTest._

class LoggingActorTest extends TestKit(testSystem) with WordSpecLike with StopSystemAfterAll {
    "LoggingActor" must {
        "로그에 남길 메시지를 보내면 로그로 남긴다" in {
            // 로그 이벤트를 검사하기 위해 단일 스레드 환경에서 테스트를 실행한다.
            val dispatcherId = CallingThreadDispatcher.id
            val props = Props[Greeter].withDispatcher(dispatcherId)
            
            val loggingActor = system.actorOf(props)
            EventFilter.info(message = "[LOG] 로그 테스트", occurances = 1).intercept {
                loggingActor ! Log("로그 테스트")
            }
        }
    }
}
```

액터 구현

```scala
object LoggingActor {
    case class Log(message: String)
}

class LoggingActor extends Actor with ActorLogging {
    def receive = {
        case Log(message) => log.info(s"[LOG] $message")
    }
}
```

## 양방향 메시지

보낸 메시지에 대해 송신자에게 응답하는 액터가 있다면, 이를 테스트하기 위해 ```ImplicitSender``` 트레이트를 사용하면 굳이 송신자를 테스트 액터로 바꾸어줄 필요가 없다.

## 요약

* 액터는 행동을 내장한다. 테스트는 근본적으로 행동을 검사하는 수단이다.
* 메시지 기반 테스트는 깔끔하다. 변경 불가능한 상태만 오가기 때문에 테스트가 테스트 대상의 상태를 더럽힐 가능성을 차단할 수 있다.
* 일단 핵심 테스트 액터를 이해하고 나면 모든 종류의 액터에 대한 단위 테스트를 작성할 수 있다.