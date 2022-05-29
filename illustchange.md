게임 내에서 일러스트를 바꿀 방법에 대하여 생각해보았습니다


```
ld l,$BEA_AkuwaraiA1,80
```

문제편은 위와 같이 `ld`라는 **사용자 정의 함수**를 통해 기본적인 스탠딩의 경로를 얻고, 설정에 따라 동인판, 콘솔판 일러 경로로 바꾼 뒤 `_ld` 함수를 호출하는 방식을 사용하고 있습니다.

```
_ld r,":b;sprites\bea\1\bea_a11_1_warai1.png",23
```

반면 전개편은 그냥 `_ld`함수에다가 스탠딩의 완전한 경로를 그냥 보내서 호출합니다

보면 파일 이름도 경로도 차이가 있습니다

전개편 스크립트에는 `ld` 사용자 정의 함수가 `ld_old`라는 이름으로 남아있지만 이 폴더구조/파일이름의 변경으로 사실상 쓸모가 없습니다

다행히 전개편 동인판 일러스트 패치는 위의 동인 콘솔판 일러스트에서 파일구조가 전혀 바뀌지 않았습니다

따라서 저희는 `ld_old`와 `ld_inner` 사용자 정의 함수를 수정함을 통해 사용할 수 있게 만들 수 있습니다.


일단 동인판 일러스트를 다음의 폴더경로로 합니다.

```
C:\Program Files (x86)\Steam\steamapps\common\Umineko Chiru\doujin\sprites
```

그리고 `%is_old_sprites`변수는 기본적으로 0의 값을 가지는데 0과 1을 토글하는 설정을 만듭니다.


그리고 `*ld_old`와 `*ld_inner` 정의 부분을 이렇게 수정합니다.

```
*ld_inner

    notif %is_old_sprites = 1 goto *ld_inner_postold
    mov $witchh12, ":b;"
    add $witchh12, $witchh2
    add $witchh12, ".png"
    mov $witchh10, $witchh2
    add $witchh10, ".png"

    fileexist %witchh11, $witchh10
    if %witchh11 = 1 goto *ld_inner_postold
        mov $witchh12, ":b;doujin\"
        add $witchh12, $witchh2
        add $witchh12, ".bmp"
    *ld_inner_postold

    if %is_old_sprites = 1 jumpf
    mov $witchh12, ":b;"
    add $witchh12, $witchh2
    add $witchh12, ".png"
    ~
return

*ld_old
    getparam %witchh1,$witchh2,%witchh3  ;立ち絵位置（l左　C中央　ｒ右）、立ち絵、表示方法

    gosub *ld_inner

    mov $witchh15,$witchh12
    mov %witchh16,%witchh3
    if %hide_new_sprites = 1 && %is_old_sprites = 0 mov $witchh15,":r;bmp\placeholder.png" : mov %witchh16,1

    ;左の画像が前に来るようにする。
    notif %witchh1 = l jumpf
    _ld l,$witchh15,%witchh3 : mov $last_l, $witchh2
    ~
    if %witchh1 = c _ld c,$witchh15,%witchh16 : mov $last_c, $witchh2
    if %witchh1 = r _ld r,$witchh15,%witchh16 : mov $last_r, $witchh2

return
```

그리고 마지막으로 스탠딩 호출을 다음과 같이 변경합니다.

```
ld_old r,"sprites\bea\1\bea_a11_1_warai1.png",23
```

테스트를 거쳐야하지만 기본적으로 이 원리를 통해 일러스트 변경을 할 수 있다고 생각합니다