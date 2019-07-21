---
layout: post
title:  "[번역] 레일즈의 Polymorphic Association 이해하기"
subtitle:   "What's the Deal with rails' Polymorphic Associations?"
categories: development
tags: ruby_or_rails
comments: true
---


> *본 글은 [What's the Deal with Rails' Polymorphic Associations?](https://thoughtbot.com/blog/whats-the-deal-with-rails-polymorphic-associations) - Jared Carroll 를 번역한 글입니다.*

> *이 글의 원문은 2006년 12월 27일에 처음 작성되었고, 2016년 6월 1일에 업데이트 되었습니다.*
> 
> *따라서 본문에서 저자가 표현하는 현재 시점 및 관련 내용이 실제와 다소 차이가 있을 수 있음을 알려드립니다.*

---

최근 ActiveRecord의 주요 기능 중에 Polymorphic Association (역: 다형성 관계) 라는 것이 추가되었습니다. 처음에는 깔끔하게 이해되지는 않았지만, 더 쉬운 이해를 돕는 몇 가지 예제를 찾아냈습니다.

이런 코드가 있다고 해보겠습니다:

```ruby
class Person < ActiveRecord::Base
  has_one :address, :as => :addressable
end

class Company < ActiveRecord::Base
  has_one :address, :as => :addressable
end

class Address < ActiveRecord::Base
  belongs_to :addressable, :polymorphic => true
end
```

이 것은 기본적으로 `Address` 클래스가 임의의 모델과 `belongs_to`관계를 맺는 것을 가능하게 합니다. 이를 아래 코드와 같다고 해볼까요:

```ruby
class Address < ActiveRecord::Base
  belongs_to :person
  belongs_to :company
end
```

그러면 이 `addresses` 테이블은 모든 왜래키들을 갖게 될 겁니다. 하지만 하나의 `address`는 하나의 `person` 또는 `company` 에 속하는 것이지 둘 모두에 속할 수는 없어야 하므로 이 왜래키들 중 오직 하나의 값을 취해야 합니다.

네 여기서 출발합니다.

현재 레일즈 공식 문서에서는 `has_one` 과 `has_many`에 대한 `as` 키워드 매개 변수를 참조하여 "다형 인터페이스"를 지정합니다. 이건 뭘까요?

"다형성 인터페이스"를 생각할 때, 저는 루비의 `<<` 메시지의 라인 처럼 생각합니다. 저는 이 메시지를 배열, 문자열, IO 스트림 등과 같은 여러 가지 다른 객체에게로 보낼 수 있습니다. 그 객체들은 모두 이 메세지를 수신 할 때 무엇을 해야하는지 이미 알고 있기 때문이죠.

저는 `<<` 를 모든 다른 클래스가 구현하고 있는 일종의 인터페이스라고 생각합니다. Java와 같이 정적으로 유형화 된 언어 관점에서 볼 때, 공통 인터페이스를 구현하는 클래스들의 작은 계층이 있는 것처럼 보입니다.

Java 가 오퍼레이터 오버로딩을 허용하고, `<<` 가 오퍼레이터 였다고 가정하면 이 공통 인터페이스는 아마도 이렇게 생겼을 것입니다.

```java
public interface Collection {
  Collection <<(Object anObject);
}
```

그리고 마찬가지로 클래스들은 이렇게 생겼겠죠.

```java
public class Array implements Collection {
  public Collection << (Object object) {
    // object 가 추가됩니다
  }
}

public class String implements Collection {
  public Collection << (Object object) {
    // object 가 추가됩니다
  }
}
```

방향을 살짝 돌려서, `Person` 과 `Company` 가 `Address` 를 상속하도록 만들어봅시다.



```ruby
class Address < ActiveRecord::Base
end

class Person < Address
end

class Company < Address
end
```

이제 `Person`이나 `Company`는 `Address`가 아니기 때문에 "polymorphic interface" 대신에 수퍼 클래스 이름을 사용할 수 있습니다.

```ruby
class Addressable < ActiveRecord::Base
end

class Person < Addressable
end

class Company < Addressable
end
```

좋네요. `Person` 과 `Company` 객체는 `addressable` 입니다. 하지만 `Address` 클래스에는 무슨 일이 벌어졌을까요.

```ruby
class Addressable < ActiveRecord::Base
  has_one :address
end
```

이제 `Person`과 `Company`는 `address`라는 `has_one`관계를 가졌습니다. 이들은 이것을 `Addressable`로부터 상속하고 있고, 다형성 관계는 없습니다.

하지만 잠시만요, 상속이라구요? 특히 레일즈의 세계에서는 "단일 테이블 상속"(STI)을 하는데요? 만일 `Person`과 `Company` 사이에 공통된 부분이 거의 없다고 가정해봅시다. "단일 테이블 상속"을 사용한다면 무슨 일이 벌어질까요? 어떤 행의 데이터가 `Person`을 나타낸다면 `Company`를 나타내는 수많은 속성 컬럼이 비어있을 것이고, 그 반대의 경우도 마찬가지가 될 것입니다.

이것은 그다지 좋지 않네요.

시스템에 `Addressable`의 종류가 더 많아질 수록 테이블은 점점 더 거대해질 겁니다. 제게 있어 `Addressable`을 사용하는 것은 합리적이긴 합니다. 상속은 상태가 아니라 행동에 관한 것이고 데이터베이스에 대해 생각하지 않고 모델링한다면 아마도 `Addressable` 클래스를 사용했을 것입니다.

결론적으로 STI("단일 테이블 상속")는 기본적으로 동작이 아닌 공통 상태가 많은 클래스의 상속 계층을 매핑하는 방법이라고 말할 수 있습니다.

여기 상속 계층 구조를 관계형 데이터베이스에 매핑하는 다른 방법이 있습니다. 사실, 일반적으로 세 가지가 사용됩니다만 레일스는 우리에게 가장 간단한 STI만을 제공합니다.

**단일 테이블 상속 (STI)** - 전체 계층 구조는 해당 행의 클래스 이름을 가지는 여분의 컬럼들(레일즈에서는 "type")이있는 단일 테이블로 매핑됩니다.  
**구체 클래스 당 하나의 테이블** - 각각의 구체 클래스는 그 자신에 속한 하나의 테이블을 가집니다. 그리고 수퍼 클래스는 추상 클래스이며, 자신의 테이블은 가지지 않습니다. 수퍼 클래스의 모든 공통 상태는 각 서브 클래스 테이블에 복제됩니다.  
**클래스 당 하나의 테이블** - 계층의 각 클래스는 자체 테이블을 가집니다. 서브 클래스 테이블에는 수퍼 클래스 테이블의 대응하는 행을 참조하기 위한 외래 키가 있습니다.  
위 예시에서 두 번째 경우는 작동하지 않았습니다. 그러나 마지막 방법인 **"클래스 당 한나의 테이블"**은 흥미롭네요.

우리의 Person, Company, and Addressable 예제로 돌아와 봅시다.

만약 우리가 이 클래스들을 각각 그 자신만의 테이블로 매핑한다면 무엇을 해야하고, 데이터베이스는 어떻게 생겨야 할까요?

```
addressables (id)
people (id, name, age, height, weight, addressable_id)
companies (id, size, established_date, addressable_id)
addresses (id, street, city, state, addressable_id)
```

흠. 우리는 상위 클래스로 Addressable을 사용하여 상속을 모델링하는 괜찮은 계층 구조를 얻었습니다. 그리고 이제 모든 하위 클래스를 자체 테이블에 집어 넣을 필요도 없습니다. 이렇게 우리는 그 모든 비어있는 컬럼을 가진 크고 못생긴 STI 테이블을 없앴습니다. 데이터베이스에서 Person을 읽어 올 때, 우리는 addressable_id 로 Person의 상위 클래스 테이블 addressables 를 조인할 것입니다. 그리고 Person의 address 를 읽으려면 addressable_id 로 addresses를 조인해야 합니다.

이렇게 우리는 다형성 연관성을 제거하고 상속으로 대체했습니다. 레일즈가 "클래스 당 하나의 테이블" 이라는 상속 매핑 스키마를 지원한다고 가정했으나 그렇게 하지 않았고, 계속 진행해 보겠습니다. 저는 우리의 계층 구조가 마음에 듭니다. 사람은 주소를 지정할 수 있습니다. 회사도 주소가 있습니다. 멋지고 논리적입니다. 여기서 잠깐, addressables 테이블은 어떻게 처리될까요? 이 테이블은 하나의 컬럼을 가집니다. 좀 이상하네요. 지금부터, 이 방법의 몇 가지 이상함과 불리함을 나열해보겠습니다.

먼저, 또다른 기능을 추가한다고 해봅시다. 사용자들은 `people`과 `company`들에 `tag`할 수 있기를 원합니다. 우리는 다형 관계를 사용하는 레일즈 플러그인인 acts_as_taggable을 사용할 수도 있습니다. 우리의 예시를 위해 그렇게 하지는 않겠습니다.

Taggable 클래스를 사용하면 좋을 것 같습니다. 하지만 잠시만요. 루비는 다중 상속을 허용하지 않으며 Java/C#의 인터페이스와 같은 것을 갖고 있지 않습니다. 대신에 루비는 모듈을 사용합니다. 네 우린 망했군요. Addressable을 클래스가 아닌 모듈로 모델링해야합니다. 사실 망쳤다고 보기는 어렵습니다. 현재 까지의 요구사항에 따라 모델링 한 것이었으니까요. 우리는 태깅 기능이 나타날 줄 몰랐을 뿐입니다. 이건 애자일한 과정에서 흔히 있는 이야기죠.

어쨌거나, 모듈을 사용해봅시다.

```ruby
module Addressable
end

module Taggable
end

class Person < ActiveRecord::Base
  include Addressable
  include Taggable
end

class Company < ActiveRecord::Base
  include Addressable
  include Taggable
end
```

Addresses와 Companies는 모두 주소 지정이 가능하고 태그 지정이 가능하도록 개선되었습니다.

```ruby
module Addressable
  has_one :address
end
```

어라, 레일즈에서는 이렇게 할 수 없네요. 이렇게는 어떨까요.

```ruby
module Addressable
  def self.included(klazz)  # klazz 는 이 module 이 include 된 class 객체임.
    klazz.class_eval do
      has_one :address
    end
  end
end
```

이제, 각 클래스들은 Address 와 `has_one` 관계를 가지는 Addressable 모듈을 include 하고 있습니다. 이 때 데이터베이스는 어떻게 생겨있을까요?

addresses (id, street, city, state, person_id, company_id)  
저런, 우리가 시작했던 지점으로 되돌아와 버렸군요. `Address` 는 `Person` 및 `Company` 와 동시에 `belongs_to` 관계를 가질 수 없습니다. 이제 다형성 관계가 필요해졌습니다.

와, 여행을 했네요. 무엇을 배웠을까요? 처음에 우리는 다형성 관계를 사용하는 것에서 시작했습니다. 그리고 나서 리펙토링 하기로 결정한 뒤, Addressable 과 Taggable 클래스를 상속해 사용하도록 모델링하는 방식을 시도해 보았습니다. 하지만 우리는 `Person` 와 `Company` 가 모두 `Taggable` 또한 필요하게 되었을 때 문제에 부딪혔습니다. 그래서 우리는 인터페이스/다중상속의 루비 버전인 모듈을 사용하게 되었습니다. 그랬더니 우리는 처음 시작 지점으로 돌아와버리게 되었습니다.

레일즈가 오직 STI를 사용한 상속 계층구조 매핑만을 허용하고 루비가 Addressable 과 Taggable 같은 "클래스를 사용한 인터페이스와 모델링을 하는 것"의 제약을 가지고 있다는 것이 이 이야기의 교훈이었죠. 자 그러면 그 대신에 다형성 관계를 사용하고 관계의 이름으로 "다형적인 인터페이스"를 사용하게 해봅시다:

```ruby
class Address < ActiveRecord::Base
  # 다음은 Addressable을 사용할 곳입니다.
  belongs_to :addressable, :polymorphic => true
end

class Tagging < ActiveRecord::Base
  # 다음은 Taggable을 사용할 곳입니다.
  belongs_to :taggable, :polymorphic => true
end
```

Addressable과 Taggable 같은 인터페이스를 사용하여 모델링하는 것을 좋아합니다. 저는 다음 자바 클래스를 정말 좋아합니다.

```java
public interface Addressable {
  Address getAddress ();

  void setAddress (Address address);
}

public class Address {
  private String street;
  private String city;
  private State state;
  private Zipcode zipcode;
  private Addressable addressable;    // 누군가 Addressable을 구현 할겁니다.

  // getter 와 setter 등등 변수들.
}

public class Person implements Addressable {
  private Address address;

  public Address getAddress () {
    return address;
  }

  public void setAddress (Address address) {
    this.address = address;
  }
}

public class Company implements Addressable {
  private Address address;

  public Address getAddress () {
    return address;
  }

  public void setAddress (Address address) {
    this.address = address;
  }
}
```

그리고 Taggable은:

```java
public interface Taggable {
  Collection getTaggings ();
}

public class Tagging {
  private Taggable taggable;  // 누군가 Taggable을 구현 할겁니다.
  private Tag tag;

  // 다른 변수들..
}

public class Person implements Addressable, Taggable {
  // 이 위에서 Addressable 구현됨.
  private Collection taggings;

  public Collection getTaggings () {
    return taggings;
  }
}

public class Company implements Addressable, Taggable {
  // 이 위에서 Addressable 구현됨.
  private Collection taggings;

  public Collection getTaggings () {
    return taggings;
  }
}
```

Java/.Net 세계의 ORM 라이브러리들에서는 3가지 스키마 매핑 방법을 모두 제공하는 반면에 레일즈에서는 오직 STI만을 제공합니다.

다른 두 개의 상속 스키마 매핑이 레일즈의 "엔터프라이즈" 버전인 것도 아니고, DHH에게 그 구현이 어려웠던 것도 아닙니다. 그보다는 루비의 인터페이스 제약으로 인해 다른 2 가지 스키마가 인터페이스를 지원하는 언어만큼 강력하지 못하기 때문이었습니다. 위의 예제에서와 같이 Addressable 까지는 정상적으로 작동하지만 곧이어 Taggable 즉 다른 인터페이스가 도입된 것 처럼 말이죠.

그리고 Rails 형태의 "다형성 관계"는 Java/.Net ORM 라이브러리에서는 발견되지 않습니다. 물론 루비가 다중 상속의 한 형태로 모듈을 사용하는 것은 뭔가 독특하기는 합니다. "다형성 관계"도 그 중 하나죠. 인기있는 플러그인인 acts_as_taggable 의 소스를 확인해보세요. 이 플러그인은 다형성 연관을 사용하며 모듈을 통해  인스턴스 및 클래스 메소드와 같은 모든 공유하려는 동작을 include 합니다. 꽤 좋은 방법인 것 같습니다. 클래스를 정의할 때 acts_as_taggable 을 호출하기만 하면 모든 동작을 무료로 얻을 수 있으니까요.

죄송하게도 제가 만든 인터페이스가 그립네요. 위의 Java 코드를 보면, 이렇게 말할런지도 모르겠습니다. "그래 그런데 Addressable 과 Taggable 이 Person과 Company 클래스에서 정확히 똑같이 구현된걸 봐. 이건 DRY 하지 않잖아. 모듈이 있는 루비에서는 그냥 한 번만 구현하고 Person과 Company 클래스에 include 하기만 하면 된다고". 이는 사실입니다만, 만약 누군가 이 모듈을 변경하기로 결정한다면, 그것을 include 한 클래스들은 고장이 날 수도 있습니다. 이건 "깨지기 쉬운 베이스 클래스"라는 또다른 문제입니다. 정확히 그것을 복제한다는 사실은 저 또한 좋아하지만, Java 코드라면 $RAILS_ROOT/vendor/plugins 에 접근할 필요 없이 Person 과 Company 클래스들에서 구현 코드를 모두 확인할 수 있다는 장점도 있습니다.

루비가 일반적으로 Java/C#의 인터페이스를 사용할 수 있게되면 더 자연스럽고 나을 것 같기는 합니다. Rails의 다형성 관계가 아직은 새로운 것이기 때문에, Rails 커뮤니티 전반적으로 분명 혼란스러워 하고 있기도 합니다.
