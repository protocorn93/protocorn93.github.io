---
title: Design Pattern - Factory
date: 2019-04-03 20:49:22
tags: ["Design Pattern", "Factory Pattern", "Factory Method Pattern", "Abstract Factory Pattern"]
category: ["Design Pattern"]
---

## Factory Pattern

이번엔 팩토리 패턴에 대해 공부를 해보았고 이를 정리해보았다. 

이번 포스팅에서 나올 팩토리 패턴은 세 가지이다.



- 팩토리 패턴 - 사용자에게 생성 로직을 노출하지 않은채 객체를 생성한다.
- 팩토리 메소드 패턴 - 객체 생성을 위한 인터페이스를 정의하지만 이를 구현한 객체가 어떤 클래스를 생성할지 정한다. 인터페이스 구현 객체에게 객체의 생성을 위임한다. 
- 추상 팩토리 패턴 - 연관된 혹은 의존성이 있는 객체의 그룹을 구체적인 클래스를 지정하지 않고 생성하기 위해 인터페이스를 제공한다.



**팩토리 패턴과 팩토리 메소드 패턴**

사실 이 둘의 차이를 알지 못했다. 같은 디자인 패턴인 줄 알았지만 엄연히 서로 다른 패턴이었다. 

팩토리 패턴은 단순히 클래스 생성의 로직을 숨겼다는 것에 의미가 있다. 코드를 예제로 살펴보자. 

```swift
class Barrack { 
	func makeMarin() -> Marin { 
    	return Marin()
    }
    
    func makeZergling() -> Zergling { 
    	return Zergling()
    }
}

class Marin {  }

class Zergling {  }
```

사용자는 `Barrack`은 필요한 객체 생성의 메소드를 호출하여 원하는 객체를 얻을 수 있다. 단순히 `Marin`과 `Zergling` 생성 로직을 감춘 것뿐이다. 

그럼 이제 팩토리 메소드를 살펴보자. 이 둘의 차이를 이해하는데 스택오버플로우의 다음 문장이 많은 도움이 되었다.

***Client doesn't know what concrete classes it will be required to create at runtime, but just wants to get a class that will do the job.
사용자는 런타임에 정확히 어떤 구체적인 타입의 클래스가 생성될지 모르지만 어떤 객체던지 그 일을 하길 원한다.***

맞다! 구체적으로 어떤 타입의 객체가 생성되던지 상관없이 **그 일**만 하면 된다.

스타크래프트에서 어떤 종족이든 유닛 조합이 존재하는데 이 조합에는 여러 유닛들이 포함되어 있고 이 유닛들은 단순히 내가 컨트롤에만 움직이면 된다. 이를 코드로 표현해보자.

먼저 팩토리 패턴에서 예로 들었던 배럭을 팩토리 메소드 패턴으로 바꿔보자. 

```swift
enum UnitType { 
	case marin, zergling
}

protocol Unit { 
	func attack() 
    func move() 
    func patrol()
}

class Marin: Unit { 
	func attack() { 
    	print("마린이 공격한다.")
    }
    
    func move() { 
    	print("마린이 움직인다.")
    }
    
    func patrol() { 
    	print("마린이 정찰한다.")
    }
}

class Zergling: Unit { 
	func attack() { 
    	print("저글링이 공격한다.")
    }
    
    func move() { 
    	print("저글링이 움직인다.")
    }
    
    func patrol() { 
    	print("저글링이 정찰한다.")
    }
}

class Barrack { 
    func make(_ type: UnitType) -> Unit { 
    	switch type { 
        case .marin:
            return Marin()
        case .zergling:
			return Zergling()
        }
    }
}
```

우리는 배럭을 통해 원하는 유닛의 구체적인 클래스를 몰라도 생성할 수 있다. 그리고 그 유닛은 공격, 정찰, 이동이라는 기본적인 유닛의 일을 할 수 있다. 또한 최초 한 번의 `switch`문을 통해 원하는 유닛 객체를 생성하고 그 이후에는 더 이상 조건문을 호출할 필요 없이 유닛에게 명령을 내릴 수 있다.

`if-else`/`switch`문과 같은 조건문은 프로그램 내에서 불가피하게 존재할 수밖에 없다. 하지만 조건문이 사용되는 곳이 많아지면 조건이나 케이스가 추가될 때마다 해당 조건문을 모두 수정해야 하는 상황이 온다. 함수 내부에서 조건문이 존재한다면 하나의 조건 혹은 케이스만 추가해도 함수는 길어진다. 그리고 함수 내부에 조건문이 있다는 것은 하나의 함수가 여러 가지 일을 할 수 있다는 것을 의미한다. 이는 SRP(Single Responsibility Principle)를 위반한다. 또한 조건 혹은 케이스가 추가될 때마다 코드를 수정해야 하기 때문에 OCP(Open Closed Principle)도 위반하게 된다. 그렇기 때문에 같은 조건문이 반복된다면 이는 팩토리 메소드 패턴으로 조건문을 숨기고 조건문을 한 곳에만 위치시킬 수 있다.

그렇다면 추상 팩토리 패턴은 무엇일까?

배럭처럼 유닛을 생성하는 건물은 이름은 다르지만 종족마다 존재한다. 그럼 이 역시 추상화를 진행할 수 있다. 우리는 이렇게 배럭의 역할을 하는 건물만 교체하면, 즉 종족만 바꾸면 그 종족에 해당하는 유닛을 생성할 수 있다. 

이렇게 어떤 팩토리를 사용할 것인지 결정만 하면 그 하위 제품들은 그에 따라 변경되도록 하는 것이 추상 팩토리 패턴이다.

```swift
enum Race { 
	case terran, zerg, protos
}

protocol Barrack {
    func make(_ type: UnitType) -> Unit?
}

class ZergBarrack: Barrack {
    func make(_ type: UnitType) -> Unit? {
        switch type {
        case .zergling:
            return Zergling()
        case .ultra:
            return Ultra()
        default: return nil
        }
    }
}

class TerranBarrack: Barrack {
    func make(_ type: UnitType) -> Unit? {
        switch type {
        case .marin:
            return Marin()
        case .medic:
            return Medic()
        default: return nil
        }
    }
}

class ProtosBarrack: Barrack {
    func make(_ type: UnitType) -> Unit? {
        switch type {
        case .zilet:
            return Zilet()
        case .dragon:
            return Dragon()
        default: return nil
        }
    }
}

class User {
    var race: Race
    
    init(_ race: Race) { 
        self.race = race
    }
    
    func buildBarrack() -> Barrack { 
        switch race { 
        case .zerg:
            return ZergBarrack()
        case .terran:
            return TerranBarrack()
       	case .protos:
            return ProtosBarrack()
        }
    } 
    
    func make(_ unit: UnitType) -> Unit? { 
        return buildBarrack().make(unit)
    }
}
```

유닛을 생성하는 건물도 종족에 따라 다른 유닛을 생성해주도록 추상화를 진행해주었다. 이렇게 추상화 팩토리 패턴을 사용하면 연관된 여러 객체들을 동시에 교체해줄 때 편리하게 사용할 수 있다. 하지만 그에 속한 객체가 많아질수록 관리하기가 어렵다는 점도 존재한다.



**참고자료**

- [Design Patterns: Factory vs Factory method vs Abstract Factory](https://stackoverflow.com/questions/13029261/design-patterns-factory-vs-factory-method-vs-abstract-factory)

- [팩토리 메서드 패턴 in Swift](http://blog.naver.com/itperson/220885347418)