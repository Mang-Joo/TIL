# 컴파일언어와 인터프리터 언어

## 컴파일 언어와 인터프리터 언어

### 컴파일 언어

컴파일 언어란 컴파일러에 의해 소스 코드 전체가 저급 언어로 변환되어 실행되는 언어입니다.\
대표적 컴파일 언어로는 C언어가 있습니다.\
또한 컴파일을 하게 될 때 컴파일러가 소스 코드 내에서 오류를 하나라도 발견하면 해당 소스 코드는 컴파일에 의해 실패하게 됩니다.

컴파일 언어의 가장 큰 장점은 실행 속도가 빠르다는 것입니다. 컴파일된 실행 파일은 기계어로 변환되어 있기 때문에, CPU에서 바로 실행될 수 있습니다. 또한, 컴파일된 실행 파일은 보안성이 높기 때문에, 다른 사람이 소스 코드를 볼 수 없어 보안 문제가 발생할 확률이 낮습니다.

하지만 컴파일 언어는 실행 파일을 만들어야 하기 때문에 실행 파일의 크기가 크고, 실행 파일을 만드는 과정이 필요합니다. 또한, 소스 코드를 수정할 때마다 컴파일 과정이 필요하기 때문에, 개발 시간이 더 많이 소요될 수 있습니다. 또한, 다양한 플랫폼에서 실행 파일을 만들어야 하는 경우에는, 각각의 플랫폼에 맞게 컴파일 과정을 거쳐야 합니다.

### 인터프리터 언어

인터프리터 언어는 소스 코드를 직접 실행하는 언어입니다. 컴파일 언어와 달리 소스 코드를 바이너리 코드로 변환하지 않고, 인터프리터에 의해 한 줄씩 해석되어 실행됩니다. 대표적인 인터프리터 언어로는 Python, JavaScript 등이 있습니다.

인터프리터 언어의 가장 큰 장점은 소스 코드를 수정할 때마다 바로 실행 결과를 확인할 수 있다는 것입니다. 컴파일 언어와 달리 실행 파일이 없기 때문에, 소스 코드를 수정하면 바로 실행 결과가 반영됩니다. 또한, 다양한 플랫폼에서 소스 코드를 그대로 실행할 수 있습니다.

하지만 인터프리터 언어는 컴파일러에 비해 속도가 느리다는 것이 가장 큰 단점입니다. 소스 코드가 실행되기 때문에, 컴파일 언어에 비해 실행 속도가 느릴 수 있습니다. 또한, 보안성이 낮기 때문에, 소스 코드를 쉽게 볼 수 있어 보안 문제가 발생할 확률이 높습니다.

따라서, 컴파일 언어와 인터프리터 언어는 각각의 특성에 따라 사용 용도가 다르며, 적절한 상황에서 선택하여 사용해야 합니다. 만약 실행 속도가 중요한 경우에는 컴파일 언어를, 소스 코드를 자주 수정해야 하거나 다양한 플랫폼에서 실행해야 하는 경우에는 인터프리터 언어를 선택하는 것이 좋습니다.

> 자바는 인터프리터와 컴파일이 동시에 수행됩니다.\
> 해당 내용은 JVM을 다룰 때 더 자세히 다뤄보겠습니다.
