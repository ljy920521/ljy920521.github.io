## 8/18 Shellscript

---

### 스크립트 실행 방법

1. shell command 이용

   - bash script_name

2. 스크립트 내 shebang을 이용 (#!bin/bash)

   - ```
     $ cat test.sh
     #!/bin/bash
     echo hello
     $ . ./test.sh
     echo hello
     ```

3. dot command 이용

   - . ./script_name

     

### 디버그 모드

- 1번 실행 방법에서 옵션 추가해서 사용

- 옵션

  - -x  : action에 중점

  - -v  : coding에 중점

  - -f  : 메타문자 사용금지(*, ?)

    ```
    $ cat test4.sh
    #!/bin/bash
    ls /etc/*.conf
    
    $ bash test4.sh
    /etc/adduser.conf          /etc/gai.conf         /etc/logrotate.conf           ...
    $ bash -f test4.sh
    ls: cannot access '/etc/*.conf': No such file or directory
    ```

- 스크립트 내에서 디버그 설정/해제

  - set -x : 디버그 모드 활성화

  - set +x : 디버그 모드 비활성화

    

### 환경변수

- initialization File(초기화 파일)
  - /etc/profile
  - $HOME/.profile

- env : 환경변수 확인

  ```
  env | grep HOME
  ```

- prompt 변수

  - PS1 : 주 프롬프트 $

    ```
    $ echo $PS1
    \[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$
    ```

  - PS2 : 부 프롬프트 >

    ```
    $ nslookup
    > 
    ```

  - PS3 : select #?

  - PS4 : debug +

- PATH 변수

  ```
  $ echo $PATH
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
  ```

- hash

  - 사용자가 사용한 명령어 목록 및 사용회수 저장

    ```
    $ hash
    hits    command
       2    /bin/bash
       3    /usr/bin/vi
       1    /usr/bin/env
    ...
    
    $ hash -r # 초기화
    ```

- Exit Status

  ```
  $ echo $?	# 0: 정상 실행 후 종료, 1: 비정상
  ```

  

### Command Line

- 명령어 동시 수행

  ```
  $ date; cal
  2020. 08. 18. (화) 11:17:41 KST
        8월 2020         
  일 월 화 수 목 금 토  
                  1  
  ...
  ```

- 명령어 조건부 실행

  - [명령어1] && [명령어2] : 명령어1의 실행결과가 참인 경우 명령어2를 실행
  - [명령어1] || [명령어2] : 명령어1의 실행결과가 거짓인 경우 명령어2를 실행

  ```
  $ grep 'root' /etc/passwd && echo '**root found**'
  root:x:0:0:root:/root:/bin/bash
  **root found**
  $ grep 'abc' /etc/passwd && echo '**root found**'
  student@CCCR15:~$
  
  $ grep 'root' /etc/passwd || echo '**not found**'
  root:x:0:0:root:/root:/bin/bash
  $ grep 'abc' /etc/passwd || echo '**not found**'
  **not found**
  ```

- 명령어 grouping

  ```
  $ (date; cal) > datecal.txt
  ```

  

### 변수

- 일반적인 변수 선언

  - 변수는 기본적으로 문자열 형태로 처리됨 - 산술연산은 expr

  ```
  student@CCCR15:~$ NUM=100
  ```

- Quotation

  - ```
    ' ' : 모든 특수문자를 무시하는 문자열
    " " : 일부 특수문자를 인식하는 문자열 ($, \, `)
    ` ` : 내부 명령어를 실행하고 실행 결과를 출력
    ```

    ```
    $ echo 'My Home is $HOME'
    My Home is $HOME
    $ echo 'Today is `date`'
    Today is `date`
    $ echo "My Home is $HOME"
    My Home is /
    $ echo "Today is `date`"
    Today is 2016년 7월  4일 월요일 오후 02시 45분 16초
    $ echo "\$HOME is $HOME"
    $HOME is /
    ```

- 변수에 다른 변수의 값을 추가

  ```
  student@CCCR15:~$ PATH=$PATH:/home/student
  student@CCCR15:~$ echo $PATH
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/student
  ```

- 위치매개변수(position parameter)

  - 사용자가 명령어 실행 시 명령어 뒤에 인자로 붙이는 값

  ```
  ./test.sh	red   blue   green
     $0        $1    $2      $3
     
  $ cat test.sh
  echo $0
  echo $1
  echo $2
  echo $3
  
  student@CCCR15:~$ ./test.sh -a -b /tmp
  ./test.sh
  -a
  -b
  /tmp
  
  student@CCCR15:~$ set a b c d e f g h i j k l
  student@CCCR15:~$ echo $1
  a
            ...
  student@CCCR15:~$ echo $10
  a0								# $1을 a로 표기하고 0이 그대로 옴
  student@CCCR15:~$ echo ${10}
  j
  student@CCCR15:~$ echo $#
  12								# 위치매개변수의 총 갯수 ($0 제외)
  student@CCCR15:~$ echo $@
  a b c d e f g h i j k l			# 위치매개변수의 총 갯수 ($0 제외)
  student@CCCR15:~$ echo \$$#
  $12								# $를 문자그대로 쓰기위해 \ 사용
  ```

- 산술연산

  ```
  expr 10 + 3
  expr 10 '*' 3
  expr 10 / 3
  
  $ echo "scale=3; 10/3" | bc	## 소수점 단위 표기위해서 bc 명령어 사용
  3.333
  ```

- read

  ```
  $ cat read.sh
  #!/bin/bash
  echo -n "input number : "
  read number
  
  $ . ./test.sh
  input number : 11
  $ echo $number
  11
  ```



### grep

- 특징
  - 문자열 검색에 사용됨

  - 정규화표현식(Regular Expression, regexp, regexr)

  - 찾은 내용을 화면에 출력한다

  - 파일을 검색해도 파일의 내용은 변경되지 않음

  - ASCII텍스트, 스크립트 등 사용자가 내용을 확인할 수 있는 파일만 검색가능
    (실행파일 등 Binary 파일은 strings 명령어를 사용하여 검색)
    
    

- 사용법
  1. grep [옵션] '[정규화표현식]' [검색대상]

  2. [명령어] | grep [옵션] '[정규화표현식]'

     - 파이프라인(Pipeline, ' | ' )

       - 좌측 명령어의 실행결과 발생한 표준 출력을 우측 명령어의 표준 입력으로 전달
         ex) # df -h | grep '/export/home

         

- 옵션
  -i : 대소문자를 구분하지 않고 검색
  -l : 검색 결과를 출력하지 않고, 찾은 파일 이름만 출력
  -n : 검색된 내용 앞에 줄 수 표시
  -v : 해당 패턴이 없는 라인을 출력
  -c : 해당 패턴이 들어있는 라인의 개수를 출력
  -w : 단일 단어 패턴만 검색

  

- grep 사용 예

  1. 사용자 검색
  grep '[사용자이름]' /etc/passwd
  cat /etc/passwd | grep '[사용자이름]'

  2. 프로세스 검색
  ps -ef | grep '[프로세스이름]'

  3. 네트워크 연결 검색
  nestat -an | grep 'ESTABLISHED'



### 정규표현식

```
^pattern : 패턴으로 시작되는 라인을 검색
pattern$ : 패턴으로 끝나는 라인을 검색
\<pattern : 패턴으로 시작되는 단어를 검색
pattern\> : 패턴으로 끝나는 단어를 검색

. : 아무 글자나 한 글자

b* : 패턴이 0번 이상 반복되는 부분을 검색 
ex) ab*c => abc abbc abbbc ac | abfffc(x)
     
.* : 길이 제한 없이 아무 글자 패턴 검색
ex) a.*c => abffffc, ac, abc, abbbc, ahjakdhkjahsc, a   c
 
[ ] : 포함되는 글자 중 한 글자
ex) [abcdexz!]

[A-Z] : 지정된 범위 중 한 글자
ex) [A-Z] : [ABCDEF....XYZ] [a-zA-Z0-9]

[^A-Z] : 지정된 범위에 해당하지 않는 한 글자
ex) [^A-Z] : [ABCDE....XYZ]가 아닌 패턴 검색

\ : Escape Sequence. 뒤 문자를 특수문자가 아닌 일반문자로 사용

Hello 에 정규화표현식으로 [^A-Z] 검색하면?
# grep '[^A-Z]' testword
Hello
# grep -v '[A-Z]' testword
# 

datafile 에서 다음 검색 입력하였을 때 결과 예상 후 실행
# grep '[정규화표현식]' datafile

NW	: NW 패턴을 검색
^n	: n으로 시작하는 라인 검색
4$	: 4로 끝나는 라인 검색
TB Savage (홑따옴표 없이) : TB 패턴을 Savage 와 datafile에서 검색
TB Savage (홑따옴표 치고) : TB Savage 패턴을 datafile에서 검색
5\..	: 5. 으로 시작하고 한 글자가 이어지는 패턴 검색
\.5	: .5인 패턴 검색
^[we]	: w이거나 e인 패턴으로 시작하는 라인
[^0-9]	: 0-9가 아닌 한글자 패턴 검색
	A-Za-z!@#$%^
-v [0-9]	: -v옵션에 의하여 0-9가 들어있지 않은 라인 검색
[A-Z][A-Z] [A-Z]	: 대문자 두글자, 공백, 대문자 한글자 패턴 검색
ss*	: s가 하나 있고 s가 0번 이상 반복
\<north	:  north 패턴 중 앞이 분리된 단어인 형태 검색
\<north\> : north 패턴 중 앞뒤가 분리된 단어인 형태 검색
 
+			a~z 중 한문자로 시작하고 +뒤에 단어로 끝나는 단어가 있는 라인출력
x | y
? 앞에 문자가 1개이거나 없는 라인 출력
( | ) searches나 seraching 이 포험된 라인 출력
```



### sed

- 특징
  - 입력받은 내용을 그대로 출력
  - 패턴 검색 등을 통해 특정 라인에 편집 가능
  - 비 대화형 편집기 (대화형 편집기 : vi)
  - 파일 입력시 파일 내용이 변경되지 않음
  - **라인** 단위로 처리됨

- 패턴 스페이스
  - 라인 단위로 처리할 때 패턴스페이스에 집어넣고 처리
  - 패턴스페이스에서 처리한 후 표준 출력으로 내보냄
  - -n 옵션을 사용해서 출력을 억제할 수 있음

- 홀딩 버퍼
  - -h 옵션으로 사용, 패턴 스페이스에서 특정 라인을 지정하여 저장하는 버퍼
  - -G 옵션 사용으로 출력 가능

#### Command

- p : 해당 라인 표준 출력으로 내보냄.
- d : 해당 라인 삭제
- s : 교체(Substitute). 패턴을 찾고, 찾은 패턴을 변경
- q : 패턴에 해당되는 라인에서 sed 종료
- r : 파일에서 내용을 읽어온 후 스트림에 삽입
- a\ : 새로운 라인을 추가 (아래) append
- i\ : 새로운 라인을 추가 (위) insert
- c\ : 해당 라인의 내용을 변경. change
- w : 파일로 저장
- !  : 해당되지 않는 패턴 검색
- g : s와 함께 사용. g플래그 미사용 시 s명령어로 패턴 교체 시 첫 번째 패턴만 교체되고 이후의 패턴은 건너뜀. ex) s/Mary/Jane/g

#### Command option

- -e : 다중 실행
- -f : 스크립트를 불러와서 sed 실행
- -n : 기본 출력 억제
- -i : sed 로 실행한 내용을 원본에 수정사항 반영

#### Command 예제

- sample datafile

  ```
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  eastern         EA      TB Savage               4.4     .84     5       20
  northeast       NE      AM Main Jr.             5.1     .94     3       13
  north           NO      Margot Weber            4.5     .89     5        9
  central         CT      Ann Stephens            5.7     .94     5       13
  ```

- command

  ```
  $ sed '/north/p' dafafile
  northwest       NW      Charles Main            3.0     .98     3       34
  northwest       NW      Charles Main            3.0     .98     3       34
  ...
  northeast       NE      AM Main Jr.             5.1     .94     3       13
  northeast       NE      AM Main Jr.             5.1     .94     3       13
  north           NO      Margot Weber            4.5     .89     5        9
  north           NO      Margot Weber            4.5     .89     5        9
  ...
  
  $ sed '1,5p' datafile
  northwest       NW      Charles Main            3.0     .98     3       34
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southern        SO      Suan Chin               5.1     .95     4       15
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  ...
  
  $ sed -n '/west',/east/p' datafile
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  ## west 키워드 시작부터 east 키워드가 마지막으로 나올때까지
  
  cf)
  $ cat datafile > datafile2; cat datafile >> datafile2
  $ sed -n '/west',/east/p' datafile2
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  
  $ sed '3d' datafile
  ```

```
# 범위지정

1) n,m : n줄부터 m줄까지
	ex) 50,60 : 50번째 줄부터 60번째 줄 까지
	sed -n '50,60p' datafile
	sed '1,5d' datafile

2) 패턴검색 : /정규표현식/
	ex) sed -n '/a..e?/p' datafile

3) 패턴으로 범위지정 : /패턴1/,/패턴2/
	ex) sed -n '/3/,/5/p' line

# Sed Print Command

sed '/north/p' datafile
=> 기본적으로 전체를 출력하면서 north 있는 패턴은 한번 더 출력
sed '1,5p' datafile
=> 기본적으로 전체를 출력하고 1~5줄을 한번 더 출력
sed -n '/north/p' datafile
=> 기본 출력을 억제하고 north 패턴이 있는 라인만 출력
sed -n '1,5p' datafile
=> 기본 출력을 억제하고 1~5줄만 출력
sed -n '/west/,/east/p' datafile
=> west 패턴부터 시작해서 east 패턴 찾을 때 까지 출력
sed -n '5,/^northeast/p' datafile
=> 5번째 줄부터 ^northeast 패턴 찾을때 까지 출력
sed '/north/!p' datafile	
=> 전체를 출력하며 north 패턴을 찾지 못한 라인을 한번 더 출력

참고) sed -n '1,5p' datafile = sed '5q' datafile

# Sed Delete Command

sed '3d' datafile
=> 3번째 줄일 경우 지움
sed '/north/d' datafile
=> north 패턴을 찾을 경우 지움
sed '2,5d' datafile
=> 2~5줄에 해당할 경우 지움
sed '/north/,/south/d' datafile
=> north 패턴부터 south 패턴까지 지움


# Sed Substitute command

sed 's/3/A/' datafile
=> 라인별로 3을 찾고 첫번째 3을 A로 변경 후 다름 라인 진행
sed 's/3/A/g' datafile
=> 라인별로 전체 3을 찾아서 A로 변경
sed 's/west/north/g' datafile
=> west를 north로 변경 
sed -n 's/^west/north/p' datafile
=> ^west를 찾아서 west를 north로 변경 후 변경한 라인 출력
sed 's/[0-9][0-9]$/&.5/' datafile
=> 숫자2자리로 끝나는 패턴 검색 후 해당 패턴 뒤에 .5 삽입
sed -n 's/Hemenway/Jones/gp' datafile
=> 기본출력을 억제하고 Hemenway를 찾아서 Jones로 바꾸고 바꾼 라인 출력
sed 's/[0-9]$//' datafile
=> 맨 끝자리가 숫자인 패턴을 검색하고 마지막 숫자를 삭제
sed 's/\<[0-9]\>//' datafile
=> 독립된 단어 형태의 숫자 한 글자를 삭제

a\ : 새로운 라인을 추가 (아래) append
# sed '10a\
10줄 아래에 추가됩니다' datafile

i\ : 새로운 라인을 추가 (위) insert
ex) # sed '1i\
제목입니다' datafile

c\ : 해당 라인의 내용을 변경. change
ex) # sed '/northeast/c\
northeast는 폐선입니다' datafile

# Sed File Input/Output

r : 파일의 내용을 읽어와서 찾은 부분에 삽입
ex) sed '/eastern/r message' datafile
message 파일의 내용을 eastern 패턴 아래에 삽입

w : 찾은 내용을 파일로 저장
ex) sed '/north/w northfile' datafile
= grep 'north' datafile > northfile
= sed -n '/north/p' datafile > northfile

# sed 다중 편집
sed -e '[작업1]' -e '[작업2]' [대상파일] 

# cat > fruits
orange
apple
banana

orange -> apple
apple -> banana

apple
banana
banana

#sed -e 's/apple/orange/' -e 's/orange/banana/' fruits
banana
banana
banana

다중 실행시 앞 명령의 실행결과가 뒤 명령에 영향을 줄 수 있으므로 영향을 파악하고 실행
```



#### sed script

- command 끝에 space나 text가 오면 안됨

- sample

  ```
  student@CCCR15:~/Desktop/shell$ cat > adding
  /Lewis/a\
  Lewis is the TOP Salespersion for April!\
  Lewis is mobing to the southen district next month.\
  CONGRATULATIONS!
  
  /Margot/c\
     ****************************\
     MARGOT HAS RETIRED \
     ****************************
  1i\
  EMPLOYEE DATABASE
  $d
  ^C
  student@CCCR15:~/Desktop/shell$ vi adding 
  student@CCCR15:~/Desktop/shell$ sed -f adding datafile
  EMPLOYEE DATABASE
  northwest       NW      Charles Main            3.0     .98     3       34
  western         WE      Sharon Gray             5.3     .97     5       23
  southwest       SW      Lewis Dalsass           2.7     .8      2       18
  Lewis is the TOP Salespersion for April!
  Lewis is mobing to the southen district next month.
  CONGRATULATIONS!
  southern        SO      Suan Chin               5.1     .95     4       15
  southeast       SE      Patricia Hemenway       4.0     .7      4       17
  eastern         EA      TB Savage               4.4     .84     5       20
  northeast       NE      AM Main Jr.             5.1     .94     3       13
     *******************
     MARGOT HAS RETIRED 
     *******************
  ```

  

### sed 실습

#### Question

1. Jon의 이름을 Jonathan으로 교체
2. 처음 세 행을 삭제
3. Lane이 포함된 행을 삭제
4. 생일이 November 나 December인 사람들의 행 출력
5. Fred로 시작하는 행의 끝에 세 개의 별표(*)를 추가
6. Popeye의 생일을 11/14/46 으로 교체
7. 빈행 삭제
8. 아래와 같은 sed script를 작성
   - 첫 줄에 Persional File 제목으로 삽입
   - San Francisco에 거주하는 사람을 제거
   - 마지막 줄에 THE END 추가

- databook

  ```
  Steve Blenheim:238-923-7366:95 Latham Lane, Easton, PA 83755:11/12/56:20300
  Betty Boop:245-836-8357:635 Cutesy Lane, Hollywood, CA 91464:6/23/23:14500
  Igor Chevsky:385-375-8395:3567 Populus Place, Caldwell, NJ 23875:6/18/68:23400
  Norma Corder:397-857-2735:74 Pine Street, Dearborn, MI 23874:3/28/45:245700
  
  Jennifer Cowan:548-834-2348:583 Laurel Ave., Kingsville, TX 83745:10/1/35:58900
  Jon DeLoach:408-253-3122:123 Park St., San Jose, CA 04086:7/25/53:85100
  Karen Evich:284-758-2857:23 Edgecliff Place, Lincoln, NB 92086:7/25/53:85100
  Karen Evich:284-758-2867:23 Edgecliff Place, Lincoln, NB 92743:11/3/35:58200
  
  Karen Evich:284-758-2867:23 Edgecliff Place, Lincoln, NB 92743:11/3/35:58200
  Fred Fardbarkle:674-843-1385:20 Parak Lane, DeLuth, MN 23850:4/12/23:780900
  Fred Fardbarkle:674-843-1385:20 Parak Lane, DeLuth, MN 23850:4/12/23:780900
  Lori Gortz:327-832-5728:3465 Mirlo Street, Peabody, MA 34756:10/2/65:35200
  
  
  Paco Gutierrez:835-365-1284:454 Easy Street, Decatur, IL 75732:2/28/53:123500
  Ephram Hardy:293-259-5395:235 CarltonLane, Joliet, IL 73858:8/12/20:56700
  James Ikeda:834-938-8376:23445 Aster Ave., Allentown, NJ 83745:12/1/38:45000
  Barbara Kertz:385-573-8326:832 Ponce Drive, Gary, IN 83756:12/1/46:268500
  Lesley Kirstin:408-456-1234:4 Harvard Square, Boston, MA 02133:4/22/62:52600
  
  William Kopf:846-836-2837:6937 Ware Road, Milton, PA 93756:9/21/46:43500
  Sir Lancelot:837-835-8257:474 Camelot Boulevard, Bath, WY 28356:5/13/69:24500
  Jesse Neal:408-233-8971:45 Rose Terrace, San Francisco, CA 92303:2/3/36:25000
  Zippy Pinhead:834-823-8319:2356 Bizarro Ave., Farmount, IL 84357:1/1/67:89500
  
  Arthur Putie:923-835-8745:23 Wimp Lane, Kensington, DL 38758:8/31/69:126000
  Popeye Sailor:156-454-3322:945 Bluto Street, Anywhere, USA 29358:3/19/35:22350
  
  Jose Santiago:385-898-8357:38 Fife Way, Abilene, TX 39673:1/5/58:95600
  Tommy Savage:408-724-0140:1222 Oxbow Court, Sunnyvale, CA 94087:5/19/66:34200
  
  Yukio Takeshida:387-827-1095:13 Uno Lane, Ashville, NC 23556:7/1/29:57000
  Vinh Tranh:438-910-7449:8235 Maple Street, Wilmington, VM 29085:9/23/63:68900
  ```

  

#### Answer

```
1.
$ sed -n 's/Jon/Jonathan/p' databook
Jonathan DeLoach:408-253-3122:123 Park St., San Jose, CA 04086:7/25/53:85100
```

```
2.
$ sed '1,3d' databook
Norma Corder:397-857-2735:74 Pine Street, Dearborn, MI 23874:3/28/45:245700
...
```

```
3.
$ sed '/Lane/d' databook
```

```
4.
$ head -1 databook
Steve Blenheim:238-923-7366:95 Latham Lane, Easton, PA 83755:11/12/56:20300
# :11/12/56: 부분에서만 찾아야한다

$ sed -n '/:1[12]\//p' databook
```

```
5.
sed -n '/Fred/s/$/***/p' databook
# Fred를 찾아서 s(교체) 하겠다 -> $(라인의 마지막)을 ***로, p(바꾼내용을 출력)
```

```
6.
sed -n '/Popeye/s/1\?[0-9]\/[1-3]?[0-9]\/[0-9][0-9]/11\/14\/46/p' databook

?를 인식하려면 \ 역슬래시 필요
```

```
7.
$ sed '/^[]*$/d' databook
```

```
8.
$  sed_script.sh
1i\
Personal File\
==============
/San Francisco/d
$a\
==============\
```

























