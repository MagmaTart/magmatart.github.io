---
layout: article
title:  "SendInput : 키보드/마우스 입력 이벤트 강제 발생"
date:   2016-12-02 12:00:00 -0400
modify_date: 2016-12-02 12:00:00 -0400
tags:
- C
- Win32
category: 
- Development
use_math: true
---

Win32 API의 SendInput 함수에 대해 알아보겠습니다.

<!--more-->
-----
`SendInput` 함수는 실제 마우스를 입력하고 키보드를 타이핑하는 것과 같은 동작을 가상으로 강제 발생시킬 수 있게 만들어줍니다. 실제 클릭하지 않아도 지정된 위치에서 클릭을 계속 반복하는 것과 같은 동작을 수행할 수 있습니다. 이를 통해 키보드/마우스 매크로 프로그램도 만들 수 있습니다.

`SendInput` 함수를 사용하기 위해서는 발생시킬 이벤트를 지정하는 `INPUT` 구조체를 이용해야 합니다. `INPUT` 구조체의 생김새는 아래와 같습니다. MSDN 출처입니다.
```c
typedef struct tagINPUT {
  DWORD type;
  union {
    MOUSEINPUT    mi;
    KEYBDINPUT    ki;
    HARDWAREINPUT hi;
  };
} INPUT, *PINPUT;
```

마우스 클릭 등의 마우스 이벤트를 발생시킬 때는 `mi` 구조체를 사용하고, 키보드 이벤트를 발생시킬 때는 `ki` 구조체를 사용합니다.

`INPUT` 구조체 안에, 발생시킬 입력 이벤트의 내용을 기술합니다. 마우스 이벤트임을 명시적으로 표시해주고, 관련 값들을 설정해줍니다.
```c
INPUT input;
input.type = INPUT_MOUSE;
input.mi.dx = 0;
input.mi.dy = 0;
input.mi.mouseData = 0;
input.mi.time = 0;
```

그리고, `MOUSEINPUT`의 `dwFlags` 속성을 이용하여 이벤트를 정의해주고, `SendInput` 함수로 이벤트를 운영체제에 전달합니다.
```c
input.mi.dwFlags = MOUSEEVENTF_LEFTDOWN;
SendInput(1, &input, sizeof(input));
Sleep(1);

input.mi.dwFlags = MOUSEEVENTF_LEFTUP;
SendInput(1, &input, sizeof(input));
Sleep(1);
```

`dwFlags`에 순서대로 마우스 왼쪽 버튼 누르기, 마우스 왼쪽 버튼 떼기 이벤트를 지정해주고 있고, 이벤트가 지정된 `INPUT` 구조체를 `SendInput` 함수에 넘겨 실제 이벤트를 발생시키게 됩니다.

마우스 이벤트를 예제로 해서, `SendInput` 함수의 기본적인 사용법을 알아보았습니다.
더 자세한 내용은 MSDN 공식 문서를 참조해주세요.

[SendInput 함수(winuser.h)](https://msdn.microsoft.com/ko-kr/library/windows/desktop/ms646310(v=vs.85).aspx)