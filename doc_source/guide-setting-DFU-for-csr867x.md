# Get Started
CSR867x 이용한 DFU 적용 방법 가이드.

## Attatch File
[DFU_Config.zip]()

## File Info
1. `make_dfu.bat` "\ADK4.x.x\apps\sink\dfu\" copy 하고 실행. (Cf. 경로 확인할 것)

1. `STACK.psr` “\ADK4.x.x\apps\sink\" copy 하고 변경 없이 사용.

1. `CONFIG.psr` "\ADK4.x.x\apps\sink\" copy 하고 변경 하여 사용.

   1. USER 영역 아무거나 현재 사용하는 값으로 변경 후 저장.
   
## Procedure
1. 최종 버전 준비 후 보드에 Download.

1. SPI 연결된 상태에서 `make_dfu.bat` 실행.

1. "\ADK4.x.x\apps\sink\dfu\" 폴더에 `dump_dfu.dfu` 파일 생성 확인.

1. dfu 파일 준비 완료.

1. 다운로드 하고자 하는 세트의 USB cable 연결.

1. Enter DFU mode.

1. DFU Wizard 실행 진행.
