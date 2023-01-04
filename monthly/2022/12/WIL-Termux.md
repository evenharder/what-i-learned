> Termux is an **Android terminal emulator and Linux environment app** that works directly with no rooting or setup required. A minimal base system is installed automatically - additional packages are available using the APT package manager.
> 
> - Termux official homepage

# 계기
원래는 Play Store에 있는 IDE로 개발을 하고 있었는데 하나씩 나사가 빠져 있거나 개인적인 요구사항을 만족하지 못했다. 폰트 변경이 어렵거나, format 형식을 지정할 수 없거나, 컴파일을 할 때 온라인 환경을 요구하는 등 다양한 애로사항이 있었다. 이러던 와중 [code-server](https://github.com/coder/code-server) 를 발견했다. VS Code를 localhost로 띄워서 돌릴 수 있다니 상당히 구미가 당겼다. 이를 실행하기위한 터미널 환경을 찾던 중 [Termux](https://termux.dev/en/) 를 발견했다. 평도 좋고 기능도 많아보여 설치해보았다.

# Termux 설치 과정
RTFM(Read the f-word manual)은 모두가 당연하다 여기면서도 전혀 예상치 못한 순간에 뒤통수를 한 번씩 치고 가는 마법의 문장이다. [termux-app](https://github.com/termux/termux-app)의 설치 관련 유의사항으로 Play Store쪽은 obsolete하니까 사용하지 말라는 문구가 있었는데, 설치 전에 이 readme를 발견했음에도 불구하고 그쪽으로 설치하고 말았다. 다행히 제거에 큰 문제는 없었고 곧바로 [F-Droid](https://f-droid.org/en/packages/com.termux)에서 설치했다. GitHub에도 release가 있긴 한데 후술한 Termux:API랑 호환 문제가 있었기 때문에 권장하지 않는다.

termux-app의 README에도 명시되어 있지만 Android 12 이상에선 실행 가능한 프로세스 수에 제한이 있고 CPU를 과도하게 사용하는 프로세스가 종료될 수 있다. [해당 링크](https://github.com/termux/termux-app/issues/2366#issuecomment-1237468220)를 참고하여 Termux가 백그라운드에서도 종료되지 않게 작업해주어야 한다. 나는 DontKillMyApp을 다운받아 앱이 알려준 그대로 Termux의 배터리 최적화를 비활성화했다. 다행스럽게도 Samsung DeX를 사용하면서도 Termux는 잘 살아있었다.

# proot-distro 설치
설치 후 열면 반가운 shell이 나온다. 여기서 작업을 해도 되지만, [proot-distro](https://github.com/termux/proot-distro)로 일부 Linux 배포판을 공식적으로 사용할 수 있다. 다음 명령으로 설치할 수 있다. repo README에도 나와있지만 root 권한으로 실행하면 안 된다.

```bash
~ $ pkg upgrade
~ $ pkg install proot-distro
~ $ proot-distro login ubuntu
```

`.bashrc` 같은 파일에 `alias run="proot-distro login ubuntu"` 와 같이 추가해주면 편리하다. 정말 기본판이기 때문에 `man`, `file`, `vim`, `gcc`, `tmux` 등도 직접 설치가 필요하다.

# Termux Style 사용
Termux를 며칠간 잘 사용하다가 글꼴과 색상을 바꾸고 싶어 구글링을 한 결과 [Termux Style](https://github.com/adi1090x/termux-style)이라는 코드를 발견했다. 개인적으로 애용하는 [Ubuntu Monospace](https://design.ubuntu.com/font/)도 있어 곧바로 적용했다.

# Termux:API 설치
Termux에서 작성한 코드를 클립보드에 복사하려하니 `xclip`이 잘 작동하지 않았다. 해결책을 찾던 와중 [Termux:API](https://github.com/termux/termux-api)를 발견했다. Android API 기능을 shell script로 옮겨놓은 느낌이다. 역시 [F-Droid에서](https://f-droid.org/en/packages/com.termux.api/) 다운로드할 수 있다. 설치하고 `pkg install termux-api`를 수행하면 다양한 `termux-*` 꼴의 명령이 추가된다. 클립보드에 복사해넣는 명령은 `termux-clipboard-set`이고, stdin을 통해 넣어주면 된다. IDE에서 파일의 내용을 통째로 복사할 상황이 많아 다음과 같이 script function을 만들어줬다.

```bash
#!/bin/sh -e

cl () {
  termux-clipboard-set < "$1"
}

```

# Troubleshooting
가끔씩 업데이트나 설치를 할 때마다 `pkg`의 mirror repo가 꼬일 때가 있었는데, 그럴 때는 `termux-change-repo`를 살포시 실행하면 된다.

# To-do
Termux는 minimal하게 유지할 것 같다. 주 개발환경이 아닐 뿐더러 굳이 터미널을 화려하게 구성할 필요를 못 느꼈기 때문이다.

# 결론
Termux를 통해 개발환경을 구축할 수 있는 길이 무궁무진해졌다. 결과적으로 Termux를 설치한 이유인 code-server 구동은 vim 학습으로 인해 취소되었지만 덕분에 매 순간 삽질을 하며 한 단계 성장할 수 있었다. 또 컴퓨터 OS는 Windows다보니 최근에 Linux를 자주 사용하진 않았는데 이렇게 계속 경험을 쌓을 수 있어서 다행이다. Thank you Termux!

