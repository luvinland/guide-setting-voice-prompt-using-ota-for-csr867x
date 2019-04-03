# configuration-vp-using-ota-for-csr867x

## Step 1. VP 데이터 파일 (\*.idx, \*.prm) 생성
1. Language 개수 → Event mapping → Generate  
![01](https://user-images.githubusercontent.com/26864945/55311980-5854da80-549f-11e9-9773-55d2b6e4e1a4.PNG)

1. `app\sink\image` 폴더에 생성된 `\header`, `\prompts`, `\refname` 폴더 이동 (잘라내기)
   1. 임의 지정 폴더로 이동 (잘라내기) (예. `app\sink\audioprompts`)
   
   1. Language 단위로 폴더 분리   
      예.) 4개 language 의 경우, `\01`, `\02`, `\03`, `\04` 폴더 생성  
      → 각각의 `\header`, `\prompts` 폴더 생성 및 데이터 이동. (`\01` : 0~15, `\02` : 16 ~ 31, …)  
      ![02](https://user-images.githubusercontent.com/26864945/55312009-67d42380-549f-11e9-8325-9265c007c2ad.PNG)

1. VP 데이터 폴더를 xuv 파일로 변경. (packing) : 아래 명령어 이용하여 4개 xuv 생성
   ```c
   \tools\bin\packfile.exe 01 ptn01.xuv
   \tools\bin\packfile.exe 02 ptn02.xuv
   \tools\bin\packfile.exe 03 ptn03.xuv
   \tools\bin\packfile.exe 04 ptn04.xuv
   ```

## Step 2. 기본 파티션 생성.
1. 파티션 설정 파일 작성. `vp.ptn`
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

1. 외부 메모리 파티션 write. `vp.ptn` 참조
   ```c
   nvscmd.exe –usb 0 burn vp.ptn all
   ```

1. `Sink_upgrade.c` (`logicalPartitions[]` 배열 수정)
   ```c
   static const UPGRADE_UPGRADABLE_PARTITION_T logicalPartitions[] = {
      UPGRADE_PARTITION_DOUBLE(0x1000,0x1004,MOUNTED),  // 첫 번째 언어 공간
      UPGRADE_PARTITION_DOUBLE(0x1001,0x1005,MOUNTED),
      UPGRADE_PARTITION_DOUBLE(0x1002,0x1006,MOUNTED),
      UPGRADE_PARTITION_DOUBLE(0x1003,0x1007,MOUNTED)
   };
   ```

## Step 3. XIDE sink project 설정 변경.
1. Define symbols : `ENABLE_SQIFVP`

1. OTA App. GAIA Control 버전에 따라 GAIA SPP 활성화 필요함.

   1. GAIA Control v3 의 경우 XIDE GAIA SPP feature : `Enabled`
   
   1. `Gaia_private.h` (define 추가, line 15) → VM lib. clean → build.
      ```c
      #define GAIA_TRANSPORT_NO_RFCOMM 1
      #define GAIA_TRANSPORT_SPP 1
      ```
   
   1. SPP 방식이 속도가 빠름.

## Step 4. ADK Source 수정.
1. `Sink_audio_prompts.c`
   ```c
   #if 0 /* Jace. PSKEY_FSTAB 1000 파티션 mount 를 위해 panic() 무시 처리함. */
   if(!PartitionMountFilesystem(PARTITION_SERIAL_FLASH, theSink.audio_prompt_language , PARTITION_LOWER_PRIORITY))
        Panic();
   #else
   PartitionMountFilesystem(PARTITION_SERIAL_FLASH, theSink.audio_prompt_language , PARTITION_LOWER_PRIORITY);
   #endif
   ```

1. `Main.c` / `Sink_private.h` / `Sink_upgrade.c`
   > _Upgrade complete 후, Reboot 시 “전원이 켜집니다” VP 재생을 위해 해당 파티션을 마운트 시킨 후에는 Upgrade commit 시 이전 파티션을 삭제하지 못하여, 추후 재차 Upgrade 시 파티션 접근 에러 발생함. 이를 보완하기 위하여, complete 후 Reboot 시 “전원이 켜집니다” VP 재생하지 않도록 보완 코드 삽입._

## Step 5. PSKEY 변경.
1. `USR5` : 4~7번 파티션 비어있음, VP index 결정시 참조함.
   ```c
   &028F = 00F0 0000
   ```
   
1. `USR7` : 4개 국어, 64개 문장
   ```c
   &0291 = 0008 0010 0004 0009 000B 0005 0022 0040 0000 0000 0F0F 0000
   ```
   
1. `USR26` : 통화버튼 짧게, Select Audio Prompt Language Mode
   ```c
   &02A4 = 2b08 0000 6008 4708 0008 2008 4808 0010 2008 4902 0008 2008 4a09 0008 2008 4a0a 0008 2008 4a0c 0008 2008 4b02 0010 2008 4c09 0010 2008 4c0a 0010 2008 4c0c 0010 2008 5704 0008 2000 5804 0010 2000 4602 0000 6000 d108 000a 3fff d208 000c 3fff d008 000a 3fff d308 0100 3fff d408 0100 3fff d008 000c 3fff 0000 0000 0000 0000 0000 0000
   ```
   
1. `USR30` : 16개 VP 에 대한 이벤트
   ```c
   &02A8 = 4001 0000 3fff 4701 0001 3fff 4715 0002 3fff 470f 0003 3fff 470e 0004 3fff 470d 0005 3fff 4704 0006 3ffe 4003 0007 3fff 400b 0008 3fff 4009 0009 3fff 4008 000a 3fff 4717 000b 3fff 4070 000c 3fff 4071 000d 3fff 4705 000e 3fff 4002 000f 3fff
   ```
   
1. `FSTAB` : 우선순위에 따른 파티션 설정 (내부0파티션, 외부0파티션, 외부1, 외부2, 외부3)
   ```c
   &25E6 = 0000 1000 1001 1002 1003
   ```
