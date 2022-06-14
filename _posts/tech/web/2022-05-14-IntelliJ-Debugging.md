---
layout: post
title: "IntelliJ Debugging "
category: tech
tags: web
image:
  path:   https://user-images.githubusercontent.com/44282342/172939243-84531676-aee7-48de-85f3-304d40af1f7f.png
---

이번 포스팅은 **IntelliJ에서의 Debugging**을 알아보려한다.

Step Over, Step Into 정도의 기능만 사용했었지만, 좀 더 자세히 알아보려한다.

* unordered toc
{:toc .large-only}

## Break Point
---

![BreakPoint](https://user-images.githubusercontent.com/44282342/168419934-28b7f045-c55f-4cff-9346-9521404513f4.png)

확인하고 싶은 라인에서 해당 부분을 클릭 또는 `ctrl + F8` 단축키를 이용하여 **Break Point**를 추가한다.

![condition](https://user-images.githubusercontent.com/44282342/168420003-46b55247-5108-4f66-b9a3-099b599de381.png)

Condition에 조건을 등록하여 **특정 상황에만 break**가 되도록 설정할 수 있다.

## 다양한 디버깅 기능
---

### **Resume (F9)**

![resume](https://user-images.githubusercontent.com/44282342/168420098-c0b596b1-8031-40f2-91e0-1ad5274f4b67.png)

**다음 Break Point로 이동**

![image](https://user-images.githubusercontent.com/44282342/168420177-442a895b-877c-492b-a490-972800f64f97.png)

여러 Break Point가 있을 때, 필요없는 **중간과정을 생략**하고 다음 Break Point로 넘어갈 수 있다.

### **Step Over (F8)**

![image](https://user-images.githubusercontent.com/44282342/168420348-7f8b9ff0-bb2f-41b1-be87-b1d9636a377f.png)

**현재 Line에서 다음 Line으로 이동**

![image](https://user-images.githubusercontent.com/44282342/168420487-ffb5c73b-387f-4ae9-93a9-700381de07b9.png)

다만 Line 내에 걸려있는 **동작(메서드 등)은 모두 실행** 후 다음 Line으로 넘어간다.

### **Step Into (F7)**

![image](https://user-images.githubusercontent.com/44282342/168420567-11d07075-6d68-45c0-ab4c-13f21848311e.png)

**현재 Line에서 실행하려는 동작(메서드 등)으로 이동한다.**

![image](https://user-images.githubusercontent.com/44282342/168420657-ae6bec3d-314f-4d22-8f90-f2b6c777b5c1.png)

해당 Line에서 실행하려는 동작은 test2 참조변수의 method() 메서드이다.

![image](https://user-images.githubusercontent.com/44282342/168420674-82698095-f820-4035-9fe4-bfdce881f75a.png)

`F7`로 해당 메서드로 진입을 하게 된다.

### Force Step Into (alt + shift + F7)

![image](https://user-images.githubusercontent.com/44282342/168420764-c7d74c42-85eb-4c89-8194-22f6a82a2605.png)

Step Into와 유사한 동작이지만, `getter`, `setter`와 같은 메서드을 Skip하는 설정을 해놓았다면 해당 기능은 **Skip을 무시하고 진입한다.**

### Step Out (shift + F8)

![image](https://user-images.githubusercontent.com/44282342/168420855-340c8ba5-a4ff-4822-94be-b837155c0f6e.png)

현재 Line에서 **Step Into로 호출한 Line으로 복귀**

![image](https://user-images.githubusercontent.com/44282342/168420909-27b3e5bc-38e3-4a1c-ad7c-7eed3b7dd554.png)

Step Into를 실행하여 현재 메서드를 수행하고 있다.

![image](https://user-images.githubusercontent.com/44282342/168420990-129d703b-dd28-4997-8b7a-172d0591369a.png)

`shift + F8`을 누르면 **남은 기능을 모두 수행하고** 호출한 Line으로 **다시 복귀한다.** 
즉, Step Into로 들어와 필요한 부분을 확인한 후 빠져나가고 싶을 때 이 기능을 사용한다.

### Run to Cursor (alt + F9)

![image](https://user-images.githubusercontent.com/44282342/168421157-586051e1-ae71-47e6-8157-d391758c75f2.png)

**포커스 되어있는 Line으로 이동**

![Animation](https://user-images.githubusercontent.com/44282342/168421425-2750cdb9-51cf-4017-983f-eefc7b715317.gif)

원하는 위치로 이동하고 싶을 떄, 해당 Line에 커서를 위치시킨다. 다만 **중간에 Break Point가 껴있다면 Break Point로 이동된다.**

### Evaluate (alt + F8)

![image](https://user-images.githubusercontent.com/44282342/168421737-7bf1b2f3-73de-45e5-9239-3d0ce9c27bca.png)

**Break된 라인에서 실행 가능한 동작들을 수행해볼 수 있다.**

![image](https://user-images.githubusercontent.com/44282342/168421720-a3e9bfcd-0607-4441-858e-ec4934544ccb.png)

Expression에 원하는 동작을 입력하고, 매개변수까지 바꾸어가며 미리 수행해볼 수 있다.  
`ctrl + alt + F8`로는 코드에서 간단하게(Quick) 미리보기로 가능하다.

### Watch

![image](https://user-images.githubusercontent.com/44282342/168421833-d4cc8279-6b75-457d-876a-d7b9a217c968.png)

Evaluate와 동일한 기능을 수행한다.  
다만 Watch는 필요할 떄 계속 수행하는 Evaluate와 달리 **따로 분리된 공간에 등록되어 계속 확인**할 수 있다는 장점이 있다.

### Call Stack

![Animation](https://user-images.githubusercontent.com/44282342/168421945-fc11bac2-46d5-42e1-9bca-99d334364dc0.gif)

해당 Line에 올 때까지 호출된 call stack이 출력된다.  
이전에 **어떤 매개변수가 넘어왔는지** 확인할 수 있으며, `Variables`와 `Watches`로 **값과 코드를 확인하고 예측**할 수 있다.