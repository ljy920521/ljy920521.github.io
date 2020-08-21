---
title: "ShellScript(2)"
categories:
  - ShellScript
tags:
  - shellscript


---

# 8/19 Shellscript

---

### awk

- 데이터 조작, 리포트 생성 등을 지원하는 도구

  

- 용어

  - 레코드 : 하나의 라인
  - 필드 : space, tab 등으로 구분된 레코드 내의 데이터
  - 레코드 구분자;RS(Record Separator) : 레코드와 레코드를 구분하는 구분기호
  - 필드 구분자;FS (Field Separator) : 필드와 필드를 구분하는 구분기호
  - ORS : Output RS. 출력형태의 레코드 구분자
  - OFS : Output FS. 출력형태의 필드 구분자
  - NF : Number of Field. 현재 레코드의 필드 개수
  - NR : Number of Record. 현재 처리중인 레코드의 번호
  - $0 : 전체 레코드
  - $n : 특정 레코드, $1, $2...

  

- 형식

  - nawk '/패턴/' [대상파일] : 특정 패턴을 찾고 출력 (= grep)
  -  nawk '{명령}' [대상파일] : 모든 레코드에 대하여 동일한 명령 수행
     nawk '/패턴/ {명령}' [대상파일] : 패턴을 찾고 찾은 패턴에 대하여 명령 수행

  

- 예시

  ```
  $ cat passwd
  root:x:0:0:root:/root:/bin/bash
  $ awk -F: '{print $3 "\t\t" $6}' passwd
  0               /root
  
  # 다중 구분
  $ awk -F'[ :\t]' '{print $1, $2, $3}' passwd
  
  
  #script 파일에서 awk command 사용
  $ cat employees2
  Tom Jones:4424:5/12/66:543354
  Mary Adams:5346:11/4/63:28765
  Sally Chang:1654:7/22/54:650000
  Billy Black:1683:9/23/44:336500
  
  $ cat > info
  /Tom/ { print "Tom's birthday is " $3 }
  /Mary/ { print NR, $0 }
  /^Sally/ { print "Hi Sally, " $1 "has a salary of $" $4 "."}
  
  $ nawk -F: -f info employees2 
  Tom's birthday is 5/12/66
  2 Mary Adams:5346:11/4/63:28765
  Hi Sally, Sally Chang has a salary of $650000.
  ```

  



#### 예제

- donors

  ```
  Mike Harrington:(510) 548-1278:250:100:175
  Christian Dobbins:(408) 538-2358:155:90:201
  Susan Dalsass:(206) 654-6279:250:60:50
  Archie McNichol:(206) 548-1348:250:100:175
  Jody Savage:(206) 548-1278:15:188:150
  Guy Quigley:(916) 343-6410:250:100:175
  Dan Savage:(406) 298-7744:450:300:275
  Nancy McNeil:(206) 548-1278:250:80:75
  John Goldenrod:(916) 348-4278:250:100:175
  Chet Main:(510) 548-5258:50:95:135
  Tom Savage:(408) 926-3456:250:168:200
  Elizabeth Stachelin:(916) 440-1763:175:75:300
  ```

1. 전화번호를 모두 출력
2. Dan의 전화번호만 출력
3. Susan의 성과 전화번호를 출력
4. C나 E로 시작하는 이름을 출력
5. 지역번호가 916인 사람들의 이름을 출력
6. Mike의 기부금을 출력. 액수는 $기호로 시작해야함
7. 아래 항을 포함하는 awk 스크립트를 작성하기
   - 성이 Savage인 사람들의 전체 이름과 전화번호 출력
   - Chet의 기부금을 출력
   - 첫째 달에 $250을 기부한 사람들의 전체 이름 출력



#### 답

1. ```
   $ nawk -F: '{print $2}' donors
   ```

2. ```
   $ nawk -F: '/Dan/ {print $2}' donors
   ```

3. ```
   $ nawk -F"[ :]" '/Susan/ {print $2, $3 " " $4}' donors
   ```

4. ```
   $ nawk -F'[ :]' '$1 ~ /^[CE]/ {print $1}' donors
   ```

5. ```
   $ nawk -F: '/(916)/ {print $1}' donors
   ```

6. ```
   $ nawk -F: '/Mike/ {print "$" $3+$4+$5}' donors
   ```

7. ```
   $ nawk -F: '/Savage/ {print $1 "\t" $2}' donors
   $ nawk -F: '/Chet/ {print "$"$3+$4+$5}' donors
   $ nawk -F: '$3 ~ /250/ {print $1}' donors
   ```



#### 연산자

- 산술연산자

  ```
  $ nawk '/test/ {print $1 + $2}' datafile
  ```

- 관계연산자

  ```
  $ nawk '$7 == 5 {print $1}' datafile
  $ nawk '$2 == "EA" {print $1}' datafile
  $ nawk '$2 >= 15 {print $1}' datafile
  ```

- 조건연산자

  ```
  $ nawk '{print($7>4 ? "high" $7:"low" $7)}' datafile
  ```

- 논리연산자

  ```
  $ nawk '$8>10 && or || $8<17' datafile
  $ nawk '!($8 == 13) {print $8}' datafile
  ```

- 범위연산자

  ```
  $ nawk '/^pattern/' , '/pattern^/'
  ```

#### BEGIN, END

- 시나리오(donors)

  ```
  BEGIN{ FS=":"; count=0; sum=0
  print "                               Donor List"
  print "============================================================================"
  print "Name\t\t\tPhone\t\t\tJan\tFeb\tMar\tAVG"
  print "============================================================================"
  }
  { sum = $3+$4+$5 }
  { print $1,"\t\t"$2,"\t"$3,"\t"$4,"\t"$5,"\t"sum/3 }
  { total += sum }
  { count++ }
  END{
  print "============================================================================"
  print "                               Summary"
  print "============================================================================"
  print "Total donate : $"total
  print "Avg : $"total/count
  print "============================================================================"
  }
  ```

  

  - 답

    ```
    ## 간격이 안맞는부분은 printf문 사용
    { printf "%-20s\t%s\t\t%d\t%d\t%d\t%.2f\n",$1,$2,$3,$4,$5,sum/3 }
    
    student@CCCR15:~/Desktop/shell$ nawk -f script.awks donors
                                   Donor List
    ==============================================================================
    Name                    Phone                   Jan     Feb     Mar     AVG
    ==============================================================================
    Mike Harrington         (510) 548-1278          250     100     175     175.00
    Christian Dobbins       (408) 538-2358          155     90      201     148.67
    Susan Dalsass           (206) 654-6279          250     60      50      120.00
    Archie McNichol         (206) 548-1348          250     100     175     175.00
    Jody Savage             (206) 548-1278          15      188     150     117.67
    Guy Quigley             (916) 343-6410          250     100     175     175.00
    Dan Savage              (406) 298-7744          450     300     275     341.67
    Nancy McNeil            (206) 548-1278          250     80      75      135.00
    John Goldenrod          (916) 348-4278          250     100     175     175.00
    Chet Main               (510) 548-5258          50      95      135     93.33
    Tom Savage              (408) 926-3456          250     168     200     206.00
    Elizabeth Stachelin     (916) 440-1763          175     75      300     183.33
    ==============================================================================
                                   Summary
    ==============================================================================
    Total donate : $6137
    Avg : $511.417
    ==============================================================================
    ```

    

  