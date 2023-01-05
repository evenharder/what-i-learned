사실, PowerShell은 이전에 거의 사용해본 적이 없었다. shell script은 Unix 계열에서만 조금 다루어봤고, verb-noun 형태의 명령이 이질적으로 느껴졌기 때문이다. 하지만 이번 기회에 아주 살짝 배워본 결과 입문이 아주 어려운 script language는 아니라고 느꼈다.

# 그래서 PowerShell이 뭐지?
사실, 아직도 PowerShell의 기원이나 원리는 잘 모르겠다. [MS 공식 문서](https://learn.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.3)에서는 "a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework"라고 부르고 있다. 조금 찾아보니 .NET 기반이었고 (5.1까지), cmdlet을 호출할 수 있고, 앞에서 언급했듯 shell, language, framework를 한데 묶어서 부르는 개념이고, 명령어가 길다 (verbose). 잘 와닿진 않지만 Windows의 대표 shell이니만큼 다른 컴포넌트와 잘 상호작용할 수 있지 않을까?

# PowerShell 입문 이유
겨우 함수 하나 만들어놓고 학습을 그만둔 상태라 입문이라 하기도 뭐한 수준이지만, 만든 이유는 Competitve Programming을 할 때 컴파일을 빠르게 하기 위해서였다. VS Code의 Run / Debug는 debugger 연동으로 인해 느리고, NeoVim plugin은 파일 경로를 이상하게 받아왔다. VS Code의 terminal이 어차피 PowerShell이다보니 이전에 만든 shell script를 PowerShell로 포팅해서 `Test-Cpp` 같은 함수로 만들어보고자 했다.

# 작성한 PowerShell 함수 'Test-Cpp'

코드를 살펴보며 PowerShell Function의 기초 원리를 복습해보고자 한다. 참고로 PowerShell은 알파벳 대소문자를 구분하지 않는다. Windows 파일시스템부터 알파벳 대소문자를 일반적으로 구분하지 않아서일까?

```PowerShell
function Test-Cpp {
  param (
    [string]$CppFileName,
    [string]$InputFileName
  )
  if ([string]::IsNullOrEmpty($CppFileName)) {
    Write-Output 'Test-Cpp $CppFileName $InputFileName'
    return 
  }
  if (-not($CppFileName -like "*.cpp")) {
    $CppFileName = $CppFileName + ".cpp"
  }
  if (-not(Test-Path -Path $CppFileName -PathType Leaf)) {
    throw [System.IO.FileNotFoundException] "Cpp file [$CppFileName] not found."
  }
  # if ([string]::IsNullOrEmpty($InputFileName)) {
  #   $InputFileName = (Get-Item $CppFileName).BaseName;
  # }
  if (-not($InputFileName -like "*.in")) {
    $InputFileName = $InputFileName + ".in"
  }
  if (-not(Test-Path -Path $InputFileName -PathType Leaf)) {
    throw [System.IO.FileNotFoundException] "Input file [$InputFileName] not found."
  }
  $CppExecName = "./$((Get-Item $CppFileName).BaseName).exe"
  $MyCommand = "g++ -O2 -g -Wall -std=c++17 -D__EVENHARDER $CppFileName -o $CppExecName" 
  Write-Output "------ Run: $MyCommand ------"
  Invoke-Expression $MyCommand
  Write-Output "------ Run $CppExecName < $InputFileName"
  Get-Content $InputFileName | & $CppExecName
  Write-Output ""
}
```

PowerShell의 함수는 통상적으로 _verb-noun_ 꼴을 준수해야 하며, 앞의 _verb_ 에 쓸 수 있는 동사도 정해져 있다 (`Get-Verb` cmdlet으로 확인할 수 있다). 그래서 함수 이름을 `Test-Cpp`로 지었으며, 안에서 사용하는 함수도 `Write-Output`, `Invoke-Expression`, `Get-Content` 등 _verb-noun_ 표현법을 충실히 따르고 있다.

PowerShell에서의 함수 선언은 `function <FunctionName> { ... }` 꼴이다. 매개변수는 그 밑에 열거했는데, `function <FunctionName>(param1, param2, ...) { ... }` 꼴로도 작성해도 된다. bash처럼 매개변수명 및 변수명은 `$`가 맨 앞에 붙는다. `[string]`은 해당 매개변수가 `String` 형으로 주어져야 한다는 뜻이다. type의 의미가 매우 약한 bash와 다르게 PowerShell은 [type이 다양하다](https://learn.microsoft.com/en-us/powershell/scripting/lang-spec/chapter-04). 

```PowerShell
if ([string]::IsNullOrEmpty($CppFileName)) {
    Write-Output 'Test-Cpp $CppFileName $InputFileName'
    return 
  }
```

`if ([string]::IsNullOrEmpty($CppFileName))`의 역할은 bash에서의 `if [ -z "$1"`과 동일하다. `Write-Output`은 `echo`의 역할을 하며 PowerShell에서 `echo`로도 alias되어 있다.

```PowerShell
  if (-not($CppFileName -like "*.cpp")) {
    $CppFileName = $CppFileName + ".cpp"
  }
```

bash의 비슷한 문법으로 `in`이 있다.

```PowerShell
  if (-not(Test-Path -Path $CppFileName -PathType Leaf)) {
    throw [System.IO.FileNotFoundException] "Cpp file [$CppFileName] not found."
```

조건문은 `$CppFileName`이라는 파일이 존재하지 않는지를 확인한다. `if [ ! -f "${cpp_file}" ]`과 동일하다.

```PowerShell
  $CppExecName = "./$((Get-Item $CppFileName).BaseName).exe"
```

`$CppFileName`의 디렉토리와 확장자를 제외한 파일명을 `.BaseName`으로 구한 다음 `.exe`를 뒤에 붙인다. `$CppFileName`의 확장자는 `.cpp`이기 때문에 bash에서 basename을 구하는 구문은 `bin_file="${cpp_file%.cpp}"`에 대응되며, [`basename`](https://man7.org/linux/man-pages/man1/basename.1.html)을 사용해도 된다.

```PowerShell
Invoke-Expression $MyCommand
```

`eval`류의 명령은 늘 조심해서 도입해야 하지만 개인적인 용도로 목표가 명확하게 사용하는 코드라 이 정도는 괜찮지 않을까 싶다.

```PowerShell
Get-Content $InputFileName | & $CppExecName
```

bash에서의 `"./${bin_file}" < "${input_file}"` 와 역할이 동일하다. PowerShell에서의 `&`는 해당 파일을 실행하라는 뜻이다.

# PowerShell Profile
script를 작성했으니 `.bashrc` 같은 설정파일에 함수를 import하면 되는데, 찾아보니 PowerShell에서는 [Profile](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles)을 통해서 사용자 및 호스트에 따라 환경을 설정한다. 다음 명령으로 현재 사용자의 모든 접속 확경에 영향을 미치는 profile 파일을 만들 수 있다. 해당 경로가 존재하지 않으면 폴더 또한 생성된다.

```PowerShell
New-Item -ItemType File -Path $PROFILE.CurrentUserAllHosts -Force
```

이렇게 생성된 `Profile.ps1`에 다음을 추가하였다.

```PowerShell
Import-Module "$(Split-Path -Path $PROFILE)\Modules\CppRunTool\CppRunTool.psm1" -Force
```

`CppRunTool.psm1`은 위에서 작성한 `Test-Cpp` 함수가 있는 스크립트이다. 스크립트 모듈은 `ps1` 확장자 대신 `psm1`를 사용하며, `Import-Module`로 함수를 import할 때는 파일명과 폴더명이 동일해야 한다. 이를 추가하면 PowerShell 구동시 `Test-Cpp`를 쓸 수 있어야겠지만 다음과 같이 에러 메시지가 나온다.

```PowerShell
. : 이 시스템에서 스크립트를 실행할 수 없으므로 <user directory path>\Documents\WindowsPowerShell\profile.ps1 파일을
로드할 수 없습니다. 자세한 내용은 about_Execution_Policies(https://go.microsoft.com/fwlink/?LinkID=135170)를
참조하십시오.
위치 줄:1 문자:3
+ . '<user directory path>\Documents\WindowsPowerShell\profile.ps1'
+   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : 보안 오류: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

권한문제로 인해 PowerShell 스크립트를 실행할 수 없다는 뜻이다. 다음과 같이 [실행정책](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy)을 변경해준다. 단, 이 명령은 관리자 권한으로 실행해야 한다.

```PowerShell
Set-ExecutionPolicy RemoteSigned
```

`RemoteSigned`로 정책을 바꾸면 로컬 스크립트와 외부 인터넷에서 다운로드받은 서명된(signed) 스크립트를 실행할 수 있다. 보안 문제가 있을까 염려했지만 개발함에 있어 일반적인 설정으로 보인다.

최종적으로 PowerShell을 다시 구동하면 `Test-Cpp`를 사용할 수 있다.

# 결론
PowerShell을 간단히 써보면서 새로운 분야를 도전할 때 겁먹지 말아야겠다는 생각을 했다. 공식 문서와 구글 검색 신공을 통해 기본적인 틀은 짜낼 수 있었고 하나씩 해내다보니 성취감도 상당했다. Windows에서 script 측면에서의 자동화가 필요할 때 언제든 다시 찾을 언어로 보인다.
