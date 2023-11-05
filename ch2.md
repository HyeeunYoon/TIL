# CH2 ) OpenCV로 시작하는 컴퓨터 비전
    
>## 객체지향 잘 활용하기
- 객체는 .을 찍어 자신이 가진 함수를 능동적으로 호출
  - 멤버 함수 or 메서드라 불림

- 객체 확인하기
  - type(a) : a가 어떤 클래스에 속하는지 알 수 있음
  - dir(a) : a가 가진 메서드를 모두 알 수 있음
  - help(a) : a가 지닌 메서드의 정보(무슨 일을 하는지, 어떻게 사용하는지)를 알 수 있음
    - help(a.sort()) : sort()의 정보만 알 수 있음

   - numpy.ndarray : numpy라이브러리에서 제공하는 다차원 배열 객체
     
            import numpy as np
            a=np.array([4,5,0,1,2,3,6,7,8,9,10,11])
            print(a)
            print(type(a))
            print(a.shape)
            a.sort() #sort() -> 정렬 함수
            print(a)
            b=np.array([-4,-3,-2.3,12.9,8.99,10.1,-1.2])
            b.sort()
            print(b)
            c=np.array(['one','two','three','four','five','six','seven'])
            c.sort()
             print(c)

            
>## 영상을 읽고 표시하기

    #영상 파일을 읽고 윈도우에 디스플레이하기 
    import cv2 as cv
    import sys

    img=cv.imread('catt.jpg') #영상 읽기

    if img is None:
      sys.exit('Not find file')
    
    cv.imshow('Image Display',img) #윈도우에 영상 표시
    #cv.waitKey(10000) :10초를 기다리는 것
    cv.destroyAllWindows() #모든 윈도우를 닫음


>## pixel (화소)

- 화소(pixel) 
  - 영상을 구성하는 한 점
  - 화소의 위치 : (행 좌표, 열 좌표)(r,c)로 표기
      - 행 좌표는 y축, 열 좌표는 x축에 해당
  - 왼쪽 위를 원점으로 간주
  - 채널은 RGB가 아닌 BGR 순서
  - img.shape = (948,1434,3)인 경우
    -  ![Alt text](image.png)  

           #black_cat 으로 해본 결과 img.shape = (784, 649, 3)
           import cv2 as cv
           import sys

           img=cv.imread('catt.jpg') #영상 읽기

           print(type(img))
           print(img.shape)
           print(img[0,0,0],img[0,0,1],img[0,0,2]) #(0,0)화소 조사
           print(img[340,300,0],img[340,300,1],img[340,300,2]) #(340,300)화소 조사


>## 영상 형태 변환 & 크기 축소
    import cv2 as cv
    import sys

    img=cv.imread('catt.jpg') #영상 읽기

    if img is None:
      sys.exit('Not found file')
    
    #BGR 컬러 영상을 명암 영상으로 변환
    gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)

    #반으로 축소
    gray_small=cv.resize(gray,dsize=(0,0),fx=0.5,fy=0.5)

    #영상을 파일에 저장 
    cv.imwrite('catt_gray.jpg',gray) #영상을 파일에 저장 
    cv.imwrite('catt_gray_small.jpg',gray_small)

    cv.imshow('Color image',img)
    cv.imshow('Gray image',gray)
    cv.imshow('Gray image small',gray_small)

    cv.waitKey()
    cv.destroyAllWindows()

- cvtColor 함수 : 컬러 영상 -> 명암 영상
     -  cvtColor(입력 영상, 이미지를 그레이스케일로 변환)
     -  컬러 변환을 위한 일반화 식 
        -  0.299와 같은 수는 상수이고 R,G,B는 현재 여기서 98,104,162에 해당
         ![Alt text](image-1.png)
     - cv.COLOR_BGR2GRAY : BGR로 표현된 컬러 영상을 명암으로 변환하라고 지시
- resize 함수 : 영상의 크기를 변환
  - cv.resize(입력 영상, 변환할 크기 지정)
    - dsize : 변환할 크기 지정 (dsize=(0,0)인 경우 fx,fy를 따른다는 의미)
    - fx=0.5, fy=0.5 : 가로와 세로 방향 모두 반으로 축소

>## 웹 캠에서 비디오 읽기

    import cv2 as cv
    import sys

    # 카메라와 연결 시도
    cap = cv.VideoCapture(0, cv.CAP_DSHOW)

    if not cap.isOpened():
      sys.exit("Fail to link to the camera")

    # 비디오를 구성하는 프레임 획득
    while True:
      ret, frame = cap.read()

      if not ret:
          print("Fail to get frame, exiting loop")
          break

      cv.imshow('Video display', frame)

      key = cv.waitKey(1)
      if key == ord('q'):
          break


    #카메라와 연결 끊음
    cap release()
    cv.destroyAllWindows() 

  - while 문 : 10-20행을 무한 반복
     - while True : 무한 루프를 의미 


  - VideoCapture
    - 웹 캠과 연결 시도하는 함수
    - 첫 번째 인수 : 웹 캠 번호 (하나일 경우 0)
    - 두 번째 인수 : 비디오가 화면에 바로 출력 

 - ret,frame = cap.read()
   - read함수 : 호출한 순간의 프레임을 획득
   - ret에 획득 성공 여부 반환
   - frame 객체에 프레임 저장  

>## 비디오에서 수집한 영상을 이어 붙이기

### 1)#제외한 부분이 append를 위해 추가된 코드

    import cv2 as cv
    import numpy as np
    import sys

    #웹캠 연결하기 소스코드

    #cap = cv.VideoCapture(0,cv.CAP_DSHOW)

    #if not cap.isOpened():
    #    sys.exit('failed open your camera')

    frames = [] # 프레임 저장할 리스트

        #ret : 프레임 획득 성공 여부
        #frame : frame을 가져와서 저장해놓은 곳
    #while True:
    #    ret,frame = cap.read()

    #    if not ret:
    #       print('failed get your frame.exit loop')
    #        break

    #    cv.imshow('Display camera',frame)
    #    key = cv.waitKey(1)
    #    if key ==ord('c'):
    #        frames.append(frame)
    #    elif key ==ord('q'):
    #        break

    #cap.release()
    #cv.destroyAllWindows()

    #화면이 너무 커서 축소키셔봄
    for i in range(len(frames)):
       frames[i] = cv.resize(frames[i],dsize=(0,0),fx=0.5,fy=0.5)

    #frames 리스트에 프레임이 존재할 경우
    if len(frames)>0:
        #imgs에 0번 인덱스 프레임을 넣음으로써 초기화
        imgs = frames[0]
        for i in range(1,min(3,len(frames))):
            imgs=np.hstack((imgs,frames[i]))

        cv.imshow('collected imgaes',imgs)

    #    cv.waitKey()
    #    cv.destroyAllWindows()


- for i in range(1,min(3,len(frames))):
    - 1번 인덱스부터 frames의 길이와 3 중 더 작은 수를 채택하여 그만큼 반복
       - 3일 경우 총 3개의 프레임(1,2,3)을 이어붙이지만 3미만의 수 2개까지만 이어붙여준다(1,2 인덱스만)
       - ex) 5일 경우 총 3개의 프레임(1,2,3,4,5)을 이어붙이지만 5미만의 수 4개까지만 이어붙여준다
    - 현재 imgs 초기화 시 0번째 인덱스를 넣었으니 최종 출력 프레임 인덱스는 0,1,2

- np.hstack((A,B))
  - A와 B를 가로로 연결
- np.vstack((A,B))
  - A와 B를 세로로 연결
- imgs=np.hstack((imgs,frames[i])) 에서 괄호를 두번 작성하는 이유
  - 1번째 괄호 : np.hstack함수는 튜플이나 리스트를 전달 받기 때문에 함수에 보내기 전에 튜플or리스트를 만드는 역할 수행
  - 2번째 괄호 : np.hstack함수가 전달받은 튜플or리스트를 가로로 연결하기 위함

  ### 2) 우리의 frame이나 우리가 얻은 imgs의 R,G,B색깔 또는 shape을 볼 줄 알아야함 (아래 소스 코드)

      #frames 리스트 내에 있는 모든 사진의 shape(행,열,차원)을 아는 방법
      for i in range(len(frames)):
        print(frames[i].shpae)

      #frames[0]의 위치[0,0] RGB를 알아내는 방법
      print(frames[0][0,0]) #BGR 순서로 나옴

      #RGB 순서로 받는 법 
      B,G,R = frames[0][0,0] #각각의 B,G,R값을 받아내어
      print(R,G,B) #하나씩 출력