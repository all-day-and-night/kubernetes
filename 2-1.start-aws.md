start aws
==========


1. log aws

> 우선 aws에 회원가입을 하고 신용카드를 연결한다.

> 프리티어를 사용하면 무료로 사용할 수 있지만 무료로 사용하더라도 신용카드를 연결해야 한다.

![image](https://user-images.githubusercontent.com/94096054/152725762-34dcaa43-b6dc-485c-91cb-2f2b32090b5d.png)


![image](https://user-images.githubusercontent.com/94096054/152725850-33e4a07f-38a7-48f8-8e3d-050e0a065db0.png)


2. EC2로 가상 인스턴스 생성하기

* 로그인을 하게 되면 다음과 같은 페이지를 볼 수 있다.

![image](https://user-images.githubusercontent.com/94096054/152726020-0927cae2-20ea-47f2-95d6-25504a63d7d6.png)


* 이 때 instances로 들어가 Launch instance를 클릭한다.

![image](https://user-images.githubusercontent.com/94096054/152726154-a0004c3f-4b16-4c88-bd72-533ee6309d5f.png)


* 이후 다음과 같은 페이지를 볼 수 있는데, Free tier로 사용할 수 있는 운영체제를 선택하면 무료로 사용할 수 있다.

![image](https://user-images.githubusercontent.com/94096054/152726295-b99182c5-2cb9-4323-9011-b7ab940a98b1.png)

> 쿠버네티스 실습을 위해 Ubuntu Server 20.04 LTS를 선택했다.

![image](https://user-images.githubusercontent.com/94096054/152726369-8b043b0d-45b3-42b3-b2ca-1564ddf1df21.png)

* 다음으로는 인스턴스의 타입을 선택한다. 

> CPU, 주기억장치, 보조기억장치 등의 자원을 필요에 따라 선택하면 되지만, 프리티어를 지원하는 것은 "t2 micro"밖에 없기에 선택했다.

![image](https://user-images.githubusercontent.com/94096054/152726500-5e73ed22-95aa-4797-b442-84d88a581c85.png)

* Next 버튼을 클릭 한 후 인스턴스의 자세한 사항을 기입하고 넘어간다. 

> 단순 실습을 위해 생성하기 때문에 설정을 건들지 않고 생성했다.

> 추후에 서비스를 배포할 때 더 자세히 정리해야겠다.

![image](https://user-images.githubusercontent.com/94096054/152726657-3c90296e-3329-4fe0-994d-6c0b4227ec64.png)

* Storage를 설정한다.

> 필요에 따라 Storage를 추가하면 되지만, 그대로 수행했다.

![image](https://user-images.githubusercontent.com/94096054/152726819-241511c2-743a-41b6-906d-c4d4ab6af482.png)

* 이후 tag를 추가한다. 

> tag의 이름으로 인스턴스를 관리할 수 있다. 

> 예를 들어, key = Name, Value = Webserver 로 설정할 수 있다. 

> 명시적인 tag Name으로 인스턴스를 분리해서 사용하면 된다.

![image](https://user-images.githubusercontent.com/94096054/152727202-bdcdf45f-1bea-4cfc-9928-3e2b41c926e2.png)

* Security 설정

> 외부에서 원격 접속하기 위해 ssh key를 생성하기로 결정한다.

![image](https://user-images.githubusercontent.com/94096054/152727460-251ed493-9ea6-4887-ba07-a90360ce4cc4.png)


* 마무리

> 모든 설정이 완료되면 생성할 인스턴스의 정보를 확인하고 Launch를 눌러 인스턴스를 생성한다.

![image](https://user-images.githubusercontent.com/94096054/152727509-b906a245-6023-4dfb-8342-bddd5039b2f7.png)

> 이후 ssh key를 생성하는 창이 띄워지는데, RSA를 선택한 후 key 이름을 적고 다운로드한다.

> 외부에서 접속할 때 계속해서 사용하기 때문에 적절한 디렉토리에 저장하여 사용한다.

![image](https://user-images.githubusercontent.com/94096054/152727586-61b13be8-86f3-436e-8308-2db8a00d56eb.png)



## Elastic Ip 설정

> 생성한 인스턴스를 닫고 열 때마다 public ip가 바뀌기 때문에 원격접속시 불편함을 겪을 수 있다.

> 이 때 Elastic IP를 사용하여 Elastic Ip와 인스턴스를 연결하여 고정된 IP로 원격접속을 할 수 있다.

> Elastic IP의 경우 유료이기 때문에 사용시 고려해야 하지만, 한 달에 4000원 정도 청구된다고 하니 사용해 볼 만하다.

> 사용하지 않더라도 인스턴스 수행시 할당 받는 public ip로 연결하면 된다. 

![image](https://user-images.githubusercontent.com/94096054/152727873-39731581-a447-43e6-b2d0-64cc61fc26a4.png)

* 우측 상단의 "Allocate Elastic IP address" 클릭시 

![image](https://user-images.githubusercontent.com/94096054/152733388-818c886e-74b0-4ec6-9c07-7706771f2e1b.png)

> 이러한 페이지를 볼 수 있는데 Allocate를 클릭하면 "Elastic IP"가 생성된다.

* 우측 상단의 "Actions"를 보면 Associate Elastic IP Address가 있다. 

> 클릭하면 다음과 같은 페이지를 볼 수 있다.

![image](https://user-images.githubusercontent.com/94096054/152733570-75c3d5ee-47aa-4e05-97d1-4a579b08c961.png)

> 연결하고자 하는 인스턴스를 선택하여 Associate하면 연결된다.


* 인스턴스 Elastic IP 할당 확인

> Instance page로 들어가서 확인해보면 연결한 인스턴스에 Elastic IP가 연결된 것을 볼 수 있다. 








