---
title: "ShellScript(3)"
categories:
  - ShellScript
tags:
  - shellscript


---

# 8/19 Shellscript

---



- 디렉토리 생성 프로그램

  ```shell
  #!/bin/bash
  for DIR in {1..10}
  do
          mkdir dir$DIR && echo "dir$DIR created !"
          for SUBDIR in {1..10}
          do
                  mkdir dir$DIR/subdir$SUBDIR && echo "dir$DIR / subdir$SUBDIR created !"
          done
  done
  ```



- 계산기 프로그램

  ```shell
  #!/bin/bash
  echo "***** Simple calculator *****"
  echo -n "Input Num1 : "
  read NUM1
  echo -n "Input Num2 : "
  read NUM2
  echo "Select Operator Number"
  echo -n "1) +  2) -  3) x  4) /  : "
  read OPER
  
  echo "*****************************"
  echo "Input OK. calculating..."
  echo "*****************************"
  
  case $OPER in
          1) echo "$NUM1 + $NUM2 = `expr $NUM1 + $NUM2`"
          ;;
          2) echo "$NUM1 - $NUM2 = `expr $NUM1 - $NUM2`"
          ;;
          3) echo "$NUM1 x $NUM2 = `expr $NUM1 \* $NUM2`"
          ;;
          4) echo "$NUM1 / $NUM2 = `echo "scale=3; $NUM1/$NUM2" | bc`"
          ;;
          *) echo "uncorrect operator."
          ;;
  esac
  ```



- 사용자 추가 프로그램

  ```shell
  #!/bin/bash 
  echo -n "Enter basename : "
  read BASENAME
  echo -n "Start number : "
  read STARTNUM
  echo -n " User conut : "
  read USERCNT
  
  (( USERCNT-- ))
  
  for ((i=0;i<$USERCNT;i++))
  do
          USERNAME="$BASENAME`expr $STARTNUM + $i`"
          useradd -m -d /home/$USERNAME $USERNAME && echo "$USERNAME user created..."
          echo "$USERNAME:Passw0rd" | chpasswd && echo "$USERNAME password set to P@ssword"
  done
  ```



- 파일 점검 프로그램

  ```shell
  #!/bin/bash
    
  #check file : test.txt
  
  #backup file check
  if [ -f test.txt.bak ]
  then
          #exist
          echo "backup file found."
  else
          #none
          cp test.txt test.txt.bak
          echo "backup file created."
          exit 0
  fi
  
  # file exist check
  if [ -f test.txt ]
  then
          #exist
          echo "backup file found."
  else
          #none
          cp test.txt.bak test.txt
          echo "orginial file not found!. file recovered."
          exit 0
  fi
  
  #compare file check
  if diff test.txt test.txt.bak > /dev/null 2>&1 # 명령어 실행결과를 커맨드창에 표시하지 않도록 
                                                  # diff의 결과값은 0,1로 확인한다. echo $?
  then
          echo "file check complete!"
  else
          echo "file changed!"
  fi
  ## hash check command - md5sum, sha1sum
  ```

  

   