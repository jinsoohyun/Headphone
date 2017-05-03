## 2017 Angstrom CTF Writeup
<p align=right><strong> Forensic - Headphones </strong></p>

### Exercise
forensic 100 point의 문제입니다. headphone 을 얻었으며, traffic 속에서 flag 를 찾아내야 합니다.
```
Defund got gaming headphones.
Get the flag from this traffic.
```
![Exercise](/assets/Exercise.png)
<p align="center">[그림1] Exercise</p>

<br>
### Solution

> traffic file을 다운받아보면, pcap 형식의 파일임을 알 수 있으며, wireshark를 통해서 해당 packet을 열었을 때, 아래와 같이 `USB Device packet` 통신을 확인할 수 있습니다.

![packet1](/assets/packet1.png)
<p align="center">[그림2] Traffic Packet Check</p>

<br>
> 최근 위 문제와 같이 USB Packet Data에 관한 Forensic 문제가 종종 출제된 바가 있어 몇번 풀어본 경험이 있었습니다.

> 주로 Mouse, Keyboard 에 관련된 문제였었는데, 해당 문제는 Headphone 입력에 관한 문제로써 input, output이 또 다른 문제였습니다.

> 기존의 방식으로는 아래와 같이 `usb.capdata` 영역에 존재하는 data 값을 참조하였으나, Headphone Device Packet 에서는 해당 명령어로 참조할 경우 조회가 되지 않습니다.

<b>Fail Command</b> : `tshark -r headphones.pcap -T fields -e usb.capdata`

<br>
> USB Data Packet에 대한 Instruction을 다시금 참조하면서 주어진 Packet에 대해 살펴보면, `information` 항목에 `URB_ISOCHRONOUS out`라는 정보가 기재되어 있는 것을 참고하여, 패킷 내에서 아래와 같은 isochronous packet 헤더 내에 ISO Data가 존재하는 것을 확인할 수 있었습니다.


![ISO_data](/assets/ISO_data_knnscyo7y.png)
<p align="center">[그림3] ISO Data Packet</p>
<br>

> 위 내용과 관련하여 검색해본 결과 http://egloos.zum.com/jmyoon7546/v/552294 에서 아래와 같은 정보를 참고할 수 있었습니다.

> 일반적으로 Audio, Video Stream 정보를 포함하고 있다고 설명을 하고 있는데, 문제에서 Headphone 이라고 언급이 되어 있으므로, Audio 데이터가 전송되는 것으로 추측해볼 수 있습니다.

![usb packet information](</assets/usb packet information.png>)
<p align="center">[그림4] isochronous 내용 </p>

<br>
> 위 내용을 토대로, 해당 packet 내에서 audio stream data를 추출해내는 과정을 아래와 같이 진행하였습니다.

![Tshark_command](/assets/Tshark_command.png)
<p align="center">[그림5] iso.data 추출 </p>

<br>

> 추출한 데이터를 raw format에 맞게 고친 뒤, bin 파일로 저장한 결과 일정 패턴의 데이터 형식이 반복되는 구간과 함께 raw pcm data 임을 확인할 수 있었습니다.

![Export_data](/assets/Export_data.png)
<p align="center">[그림6] 추출 raw data </p>
<br>

> 문제에서 headphone data packet 임을 우선 알 수 있었고, 추출된 데이터를 확인한 결과 해당 raw data가 pcm data임을 추측해 볼 수 있었습니다. 따라서 rate 와 Hz 등을 맞춰 주는 과정을 통해서 raw pcm data 를 재생할 수 있었으며, 이 과정에서 보다 편리하게 `aplay` command에서 자체적으로 지원하는 format을 이용하여 출력 형태와 비율을 맞추어 재생할 수 있었습니다.

![aplay_help](/assets/aplay_help.png)
<p align="center">[그림7] aplay option </p>
<br>


> raw data 에서 2byte 주기로 반복되는 01 00 fe ff을 통해서 16-bit, signed little endian two's complement 방식임을 예측할 수 있습니다. 아래와 같이 aplay 옵션 중 stereo 방식을 이용하여, 재생한 결과 flag를 음성으로 알려주는 것을 확인할 수 있었습니다.

![aplay_play](/assets/aplay_play.png)
<p align="center">[그림8] result_data aplay 재생 </p>
<br>



**flag** : actf{e392157ea599c605b6d483042ff8d9fe}
