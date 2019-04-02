# configuration-vp-using-ota-for-csr867x

### VP 데이터 파일 (\*.idx, \*.prm) 생성
   1. Language 개수 → Event mapping → Generate
   ![01](https://user-images.githubusercontent.com/26864945/55311980-5854da80-549f-11e9-9773-55d2b6e4e1a4.PNG)
   1. app\sink\image 폴더에 생성된 \header, \prompts, \refname 폴더 이동. (잘라내기)
      1. 임의 지정 폴더로 이동. (잘라내기)  (예. app\sink\audioprompts)
      1. Language 단위로 폴더 분리
      예.) 4개 language 의 경우, \01, \02, \03, \04 폴더 생성
      → 각각의 \header, \prompts 폴더 생성 및 데이터 이동. (\01 : 0~15, \02 : 16 ~ 31, …)
      ![02](https://user-images.githubusercontent.com/26864945/55312009-67d42380-549f-11e9-8325-9265c007c2ad.PNG)
   1. VP 데이터 폴더를 xuv 파일로 변경. (packing)
      1. 아래 명령어 이용하여 4개 xuv 생성
         ```c
         \tools\bin\packfile.exe 01 ptn01.xuv
         \tools\bin\packfile.exe 02 ptn02.xuv
         \tools\bin\packfile.exe 03 ptn03.xuv
         \tools\bin\packfile.exe 04 ptn04.xuv
         ```

### 기본 파티션 생성.
   1. 파티션 설정 파일 작성. (vp.ptn)
      ```c
      0, 64K, RO, ptn01.xuv  # 첫 번째 언어 (위 1-C-i. 에서 생성한 xuv 파일)
      1, 64K, RO, ptn02.xuv
      2, 64K, RO, ptn03.xuv
      3, 64K, RO, ptn04.xuv
      4, 64K, RO, (none)     # 첫 번째 언어의 OTA 를 위한 공간
      5, 64K, RO, (none)
      6, 64K, RO, (none)
      7, 64K, RO, (none)
      ```
   1. 외부 메모리 초기화
      ```c
      nvscmd.exe –usb 0 erase
      ```
   1. 외부 메모리 파티션 write. (vp.ptn 참조)
      ```c
      nvscmd.exe –usb 0 burn vp.ptn all
      ```
   1. Sink_upgrade.c (logicalPartitions[] 배열 수정)
      ```c
      static const UPGRADE_UPGRADABLE_PARTITION_T logicalPartitions[] = {
         UPGRADE_PARTITION_DOUBLE(0x1000,0x1004,MOUNTED),  // 첫 번째 언어 공간
         UPGRADE_PARTITION_DOUBLE(0x1001,0x1005,MOUNTED),
         UPGRADE_PARTITION_DOUBLE(0x1002,0x1006,MOUNTED),
         UPGRADE_PARTITION_DOUBLE(0x1003,0x1007,MOUNTED)
      };
      ```
      
