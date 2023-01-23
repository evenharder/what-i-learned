shell script를 많이 짜보진 않았으나 배울, 아니 배우지 않고는 배길 수 없는 기회가 한 번 있었다. 이전에 회사에서 긴급하게 특정 상황에서 특정 동작을 요구하는 shell script를 짜야 하는 일이 있었는데, 왜인지는 모르겠으나 shell script를 거의 써본 적이 없는 나에게 과제가 떨어졌다. 절대 오류가 있어서는 안된다는 말을 듣고 이틀 밤낮을 끙끙거렸다. 문법과 coding convention부터 하나하나 검색해가며 코드를 짜고, 온갖 예외 사항을 처리하고, 모의 테스팅 환경도 구축해보고, shellcheck도 돌려봤다. 다행스럽게도 저 멀리 전파된 코드는 오류 없이 제 역할을 수행했다.

그리고 두 번째로 배울 기회가 찾아왔다. 바로 지난 12월의 C++ 컴파일 스크립트 작성이다. 정리가 늦어져서 이제야 글을 써본다. 

# 계기
vim을 배우며 C++ 코드를 많이 작성했는데, 매번 g++ 컴파일 명령어를 쓸 수는 없는 노릇이다. CMake를 쓸 수도 있겠지만 한 번 쓰고 마는 single source file에 CMake까지 필요한지는 모르겠어서, 컴파일 스크립트를 작성했다. 덕분에 shell script 문법을 오랜만에 다시 공부했다.

```bash
#!/bin/sh -e

run () {
  format_print () {
    echo "------ run: $* ------" # SC2145
  }
  exec_print () {
    format_print "$*"
    "$@" # eval not required, SC2294
  }
  if [ -n "$1" ]
  then
    case $1 in *.cpp|*.c )
      cpp_file="$1"
      ;;
    * )
      cpp_file="$1.cpp"
      ;;
    esac
    if [ ! -f "${cpp_file}" ]
    then
      echo "run: cpp file '${cpp_file}' not found"
      return 1
    fi
    bin_file="${cpp_file%.cpp}"
    if [ -n "$2" ]
    then
      case $2 in *.in )
        input_file="$2"
        ;;
      * )
        input_file="$2.in"
        ;;
      esac
    else
      input_file="${bin_file}.in"
    fi
    if [ ! -f "${input_file}" ]
    then
      echo "run: input file '${input_file}' not found"
      return 2
    fi
    exec_print g++ -O2 -g -Wall --std=c++17 -fsanitize=address -D__EVENHARDER "${cpp_file}" -o "${bin_file}"
    # redirection operator is not fed into exec_print...
    format_print "./${bin_file} < ${input_file}"
    "./${bin_file}" < "${input_file}"
  else
    echo 'usage: run cppfile [inputfile]'
    echo
    echo '-  cppfile:   exclude .cpp'
    echo '-  inputfile: exclude .in, default to cppfile'
  fi
  return 0
}

```

`run A .in`을 수행하면 미리 설정된 `g++` 컴파일 옵션으로 `A.cpp`를 컴파일 및 링크하여 `A` 실행 파일을 만든다. 곧바로 `.in` 파일을 표준 입력으로 `A`를 실행한다.

shell script는 언제 봐도 어렵다. 작성하면서 다음을 배웠다.
- shell script 문법을 많이 까먹었다는 사실.
- `case` 문법. syntax가 괴상하다.
- `${parameter%word}`의 shell parameter expansion. 왜 `%`가 저런 동작을 하는 걸까.
- `set -v` . `set`까지 쓰고 싶진 않아서 코드에 반영하진 않았다.
- 테스팅의 필요성. shell script 개발이 익숙치 않아서인지 실수가 많이 났다. 그럼 이런 script의 테스팅은 어떻게 진행해야 할까? test suite을 만든다면 어떻게 해야 할까? stderr 등을 capture해야 할까? 이쪽 공부도 하나의 topic이 될 수 있겠다.
- `shellcheck`의 강력함. 정적 분석 툴은 정말 대단하다.

CLI 환경에서 작업을 빈번히 하면 shell scripting은 정말 생산성을 극대화할 수 있다고 생각된다. 조금 더 익숙해져야겠다.