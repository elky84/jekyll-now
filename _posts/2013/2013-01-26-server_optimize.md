---
layout: post
title: 서버 최적화 핵심 요약
date: 2013-01-26 00:23:54
categories: 서버
comments: true
---
1. Through-put
    * 초당 소화 가능 이벤트.
        * 이는 DB나 연결된 기능들과의 교신/처리가 포함된 계측 이어야 한다.
    * 분산 가능한 특정 이벤트들은 지정된 서버와의 교신 (P2P스러운 접근)만으로 처리함으로써, 비용을 분산 시킬 수 있다.
    * 일반적으로 다음 두가지 측정이 이루어지면 된다.
        *  프레임워크 (네트웍 엔진이라 불리우는) through-put.
        *  로직 through-put.
1. Through Pipe-line
    * 파이프라인이란 이벤트 처리를 위해 거치게 되는 과정을 표현한 것을 말한다.
    * 구현 별로 다른 어떤 과정에서도 wait-free 방식으로 구현하는 것이 좋다.
    * 어쩔 수 없는 상황이라면 해당 작업 전용 스레드를 할당 받는 것이 좋고.
    * Through Pipe-line은 스레드 디자인과 큰 연관이 있는데, blocking 동작이 있다면 퓨쳐 패턴으로 구현해 요청을 보낸 후, 다시 파이프라인으로 재 입력 받는 구조로 처리하는 것이 좋다.
    * 핵심은 적절한 분산을 통해 병목 지점 제거, blocking 상황을 제거해 Through-put 향상을 노리는 것이다.
1. 순환 구조 체크
    * 잘못된 순환이 이루어지면 이벤트 폭발이 일어날 수 있다.
    * 상호 순환 구조가 안 생기도록 이벤트가 어떠한 순서로 파이프라인을 순회하는지 검사하고, 파이프라인을 오래 타거나 반복해서 타는 것이 불가능 하도록 제약을 걸 필요가 있다. (spin-count를 계산해 몇 번 이상이면 이벤트를 버리는 등의 처리가 필요하다)
1. base-line
    * base-line은 초당 로직이 몇회 동작할 것인지 (프레임), 그렇다면 한 프레임이 몇 ms까지 사용해도 되는지, 또 패킷은 몇초안에 수신된다는 것을 전제로 동작하는지 등이 명확히 규정지어져야 한다.
    * base-line을 넘어설 것이 의심되면 유저 수 제한, 특정 기능 제한 등을 통해 기존 접속한 유저/잘 동작하는 기능은 보호 받을 수 있도록 구현해야 한다.
1. run-time apply
    * run-time 접속제한, run-time 기능 제한, run-time 리소스 리로드 등이 그래서 필요하다.
1. 객체의 생성 소멸 관리
    * 생성 시점과 소멸 시점은 같게 관리하는 것이 좋다.
    * 만약 주체가 다른 tier나, 다른 프로세스 등이라 관리가 어렵다면, 이에 대한 보호 장치가 반드시 필요하다. 
    * 어쩔 수 없는 상황이 아니라면 shared_ptr도 그다지 좋은 해결책이 아니다.
    * 개체의 소유권은 가능한한 엄격히 관리되어야 한다.
1. 과한 보호 장치에 대한 억제
    * 예외 상황을 검증하고 보호하기 위한 코드도 검사/검증 대상이다.
    * 이런 코드가 문제를 일으킬 여지도 있으니 이러한 코드도 세워둔 전제를 위배하지 않는지 검사하고 검증하자.
1. 무거운 단일 이벤트 보다는 작은 이벤트로 처리하라.
    * 가급적 모든 유저를 대상으로 하는 이벤트는 자제하고, 유저가 발생시키는 이벤트 단위로 동작 시켜라. (웹 서버 구조)
    * 시간에 의존하는 이벤트 (heart-beat 체크, 아이템 유효 시간 체크, 날짜별 이벤트) 들도 비동기 처리 후, 적용 대상만 추려 작은 이벤트로 발생시켜라.
    * 가능한 한 적용 대상 자체가 최대한 작게끔 유지 시키는 것이 좋다.
    * 가능하다면 비동기 처리던 아니건 간에 목표 대상 자체를 줄이고, 기능이 정말 필요한 이벤트에 물려서 동작 시켜라. (코드 공유가 잘 되어 있다면, 클라이언트에 서버 연산 예상 결과를 적용 시키고, 서버는 해당 기능이 실제로 active 되는 시점에서 트리거를 동작시키는 방법도 있다.
    * 가급적 polling보다는 event-driven을 선택하라
1. 확장성 (scalability)
    * 결합도를 낮추면 자연스레 확장성을 높일 수 있다.
    * stateless를 의도하면서도 성능을 높일 수 있는지 검토하라.
    * 서비스를 목적에 따라 작게 쪼개 관리할 수 있다면, (서비스 종류에 따라 관리 비용이 증가하지 않게 된다면)  확장성을 특히 runtime scale-in, scale-out을 좀 더 용이하게 이뤄낼 수 있다. 