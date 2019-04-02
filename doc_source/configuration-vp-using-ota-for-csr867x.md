# configuration-vp-using-ota-for-csr867x

### VP 데이터 파일 (\*.idx, \*.prm) 생성.
   1. Language 개수 → Event mapping → Generate
   ![01](https://user-images.githubusercontent.com/26864945/55311980-5854da80-549f-11e9-9773-55d2b6e4e1a4.PNG)
   1. app\sink\image 폴더에 생성된 \header, \prompts, \refname 폴더 이동. (잘라내기)
      1. 임의 지정 폴더로 이동. (잘라내기)  (예. app\sink\audioprompts)
      1. Language 단위로 폴더 분리.\
      예.) 4개 language 의 경우, \01, \02, \03, \04 폴더 생성 → 각각의 \header, \prompts 폴더 생성 및 데이터 이동. (\01 : 0~15, \02 : 16 ~ 31, …)
      
