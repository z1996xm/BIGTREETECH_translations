# MCU 명령

이 문서는 Klipper "호스트" 소프트웨어에서 전송되고 Klipper 마이크로 컨트롤러 소프트웨어에 의해 처리되는 저수준 마이크로 컨트롤러 명령에 대한 정보를 제공합니다. 이 문서가 모든 MCU 명령어를 커버하지는 않습니다.

이 문서는 저수준 마이크로 컨트롤러 명령을 이해하는 데 관심이 있는 개발자에게 유용할 수 있습니다.

명령어 형식 및 전송에 대한 자세한 내용은 [프로토콜](Protocol.md) 문서를 참조하십시오. 여기에 있는 명령은 "printf" 스타일 구문을 사용하여 설명됩니다. 해당 형식에 익숙하지 않은 사용자를 위해 '%...' 시퀀스가 표시되는 경우 실제 정수로 대체되어야 합니다. 예를 들어 "count=%c"가 있는 설명은 "count=10"이라는 텍스트로 대체될 수 있습니다. "열거"로 간주되는 매개변수 (위의 프로토콜 문서 참조) 는 마이크로 컨트롤러의 정수 값으로 자동 변환되는 문자열 값을 사용합니다. 이것은 "pin" (또는 "_pin" 접미사가 있는) 이라는 매개변수가 일반적입니다.

## 시작 명령

마이크로 컨트롤러 및 주변 장치를 구성하기 위해 특정 일회성 작업을 수행해야 할 수도 있습니다. 이 섹션에서는 해당 용도로 사용할 수 있는 일반적인 명령을 나열합니다. 대부분의 마이크로 컨트롤러 명령과 달리 이러한 명령은 수신되는 즉시 실행되며 특정 설정이 필요하지 않습니다.

일반적인 시작 명령:

* `set_digital_out pin=%u value=%c` : 이 명령은 주어진 핀을 디지털 출력 GPIO로 즉시 구성하고 이를 low level (값=0) 또는 high level (값=1)로 설정합니다. 이 명령은 LED의 초기 값을 구성하고 스테퍼 드라이버 마이크로 스테핑 핀의 초기 값을 구성하는 데 유용할 수 있습니다.
* `set_pwm_out pin=%u cycle_ticks=%u value=%hu` : 이 명령은 주어진 수의 cycle_ticks로 하드웨어 기반 PWM(펄스 폭 변조)을 사용하도록 지정된 핀을 즉시 구성합니다. "cycle_ticks"는 각 전원 켜기 및 전원 끄기 주기가 지속되어야 하는 MCU 클록 틱 수입니다. cycle_ticks 값 1은 가능한 가장 빠른 주기 시간을 요청하는 데 사용할 수 있습니다. "값" 매개변수는 0에서 255 사이이며 0은 완전히 꺼진 상태를 나타내고 255는 완전히 켜진 상태를 나타냅니다. 이 명령은 CPU 및 노즐 냉각 팬을 활성화하는 데 유용할 수 있습니다.

## 저수준 마이크로 컨트롤러 구성

마이크로 컨트롤러의 대부분의 명령은 성공적으로 호출되기 전에 초기 설정이 필요합니다. 이 섹션에서는 구성 프로세스의 개요를 제공합니다. 이 섹션과 다음 섹션은 Klipper의 내부 세부 정보에 관심이 있는 개발자만 관심을 가질 것입니다.

호스트가 마이크로 컨트롤러에 처음 연결할 때 항상 data dictionary를 얻는 것으로 시작합니다 (자세한 내용은 [프로토콜](Protocol.md) 참조). data dictionary 을 얻은 후 호스트는 마이크로 컨트롤러가 "configured" 상태인지 확인하고 그렇지 않은 경우 Configuration 합니다. Configuration 에는 다음 단계가 포함됩니다:

* `get_config` : 호스트는 마이크로 컨트롤러가 이미 구성되어 있는지 확인하는 것으로 시작합니다. 마이크로 컨트롤러는 "config" 응답 메시지로 이 명령에 응답합니다. 마이크로 컨트롤러 소프트웨어는 전원을 켤 때 항상 configuration 되지 않은 상태로 시작됩니다. 호스트가 configuration 프로세스를 완료할 때까지 이 상태를 유지합니다 (finalize_config 명령으로 실행). 마이크로 컨트롤러가 이전 세션에서 이미 configuration 되어 있고 원하는 설정으로 구성된 경우 호스트에서 추가 작업이 필요하지 않으며 구성 프로세스가 성공적으로 종료됩니다.
* `allocate_oids count=%c` : 이 명령은 호스트에 필요한 최대 object-id (oid) 수를 마이크로 컨트롤러에 알리기 위해 실행됩니다. 이 명령은 한 번만 실행하는 것이 유효합니다. oid 는 각 스테퍼, 각 엔드스톱 및 예약 가능한 각 gpio 핀에 할당된 정수 식별자입니다. 호스트는 하드웨어를 작동하는데 필요한 oid 수를 미리 결정하고 이를 마이크로 컨트롤러에 전달하여 oid 에서 내부 개체로의 매핑을 저장하기에 충분한 메모리를 할당할 수 있습니다.
* `config_XXX oid=%c ...` : 전통적으로 "config_" 접두사로 시작하는 모든 명령은 새 마이크로 컨트롤러 개체를 만들고 지정된 oid 를 할당합니다. 예를 들어 config_digital_out 명령은 지정된 핀을 디지털 출력 GPIO 로 구성하고 호스트가 주어진 GPIO 에 대한 변경을 예약하는데 사용할 수 있는 내부 개체를 생성합니다. config 명령에 전달된 oid 매개변수는 호스트에 의해 선택되며 0 과 assign_oids 명령에 제공된 최대 개수 사이에 있어야 합니다. config 명령은 마이크로 컨트롤러가 구성된 상태가 아닐 때 (즉, 호스트가 finalize_config를 전송하기 전) 및 assign_oids 명령이 전송된 후에만 실행할 수 있습니다.
* `finalize_config crc=%u` : finalize_config 명령은 마이크로 컨트롤러를 구성되지 않은 상태에서 구성된 상태로 전환합니다. 마이크로 컨트롤러에 전달된 crc 매개변수는 저장되고 "config" 응답 메시지로 호스트에 다시 제공됩니다. 관례에 따라 호스트는 요청할 구성의 32비트 CRC를 가져오고 후속 통신 세션이 시작될 때 마이크로 컨트롤러에 저장된 CRC가 원하는 CRC와 정확히 일치하는지 확인합니다. CRC가 일치하지 않으면 호스트는 마이크로 컨트롤러가 호스트가 원하는 상태로 구성되지 않았음을 알고 있습니다.

### 일반적인 마이크로 컨트롤러 오브젝트

이 섹션에는 일반적으로 사용되는 몇 가지 config 명령이 나열되어 있습니다.

* `config_digital_out oid=%c pin=%u value=%c default_value=%c max_duration=%u` : 이 명령은 주어진 GPIO '핀'에 대한 내부 마이크로 컨트롤러 객체를 만듭니다. 디지털 출력 모드에서 구성되고 'value'(낮음은 0, 높음은 1)로 지정된 초기 값으로 설정됩니다. digital_out 객체를 생성하면 호스트가 지정된 시간에 지정된 핀에 대한 GPIO 업데이트를 예약할 수 있습니다 (아래에 설명된 queue_digital_out 명령 참조). 마이크로 컨트롤러 소프트웨어가 종료 모드로 전환되면 구성된 모든 digital_out 개체가 'default_value'로 설정됩니다. 'max_duration' 매개변수는 안전 검사를 구현하는 데 사용됩니다. 0이 아니면 호스트가 추가 업데이트 없이 지정된 GPIO를 기본값이 아닌 값으로 설정할 수 있는 최대 클록 틱 수입니다. 예를 들어 default_value가 0이고 max_duration이 16000이면 호스트가 gpio를 1의 값으로 설정하면 16000 클럭 틱 내에서 gpio 핀에 대한 또 다른 업데이트(0 또는 1)를 예약해야 합니다. 이 안전 기능을 히터 핀과 함께 사용하여 호스트가 히터를 활성화한 다음 오프라인으로 전환하지 않도록 할 수 있습니다.
* `config_pwm_out oid=%c pin=%u cycle_ticks=%u value=%hu default_value=%hu max_duration=%u` : 이 명령은 호스트가 업데이트를 예약할 수 있는 하드웨어 기반 PWM 핀에 대한 내부 개체를 만듭니다. 사용법은 config_digital_out과 유사합니다. 매개변수 설명은 'set_pwm_out' 및 'config_digital_out' 명령에 대한 설명을 참조하세요.
* `config_analog_in oid=%c pin=%u` : 이 명령은 아날로그 입력 샘플링 모드에서 핀을 구성하는 데 사용됩니다. 일단 구성되면 query_analog_in 명령을 사용하여 일정한 간격으로 핀을 샘플링할 수 있습니다(아래 참조).
* `config_stepper oid=%c step_pin=%c dir_pin=%c invert_step=%c step_pulse_ticks=%u` : This command creates an internal stepper object. The 'step_pin' and 'dir_pin' parameters specify the step and direction pins respectively; this command will configure them in digital output mode. The 'invert_step' parameter specifies whether a step occurs on a rising edge (invert_step=0) or falling edge (invert_step=1). The 'step_pulse_ticks' parameter specifies the minimum duration of the step pulse. If the mcu exports the constant 'STEPPER_BOTH_EDGE=1' then setting step_pulse_ticks=0 and invert_step=-1 will setup for stepping on both the rising and falling edges of the step pin.
* `config_endstop oid=%c pin=%c pull_up=%c stepper_count=%c` : 이 명령은 내부 "endstop" 개체를 만듭니다. 엔드스톱 핀을 지정하고 "호밍" 작업을 활성화하는 데 사용됩니다 (아래 endstop_home 명령 참조). 이 명령은 디지털 입력 모드에서 지정된 핀을 구성합니다. 'pull_up' 매개변수는 핀에 대한 하드웨어 제공 풀업 저항(사용 가능한 경우)이 활성화되는지 여부를 결정합니다. 'stepper_count' 매개변수는 이 엔드스톱이 원점 복귀 작업 동안 중지해야 할 수 있는 최대 스테퍼 수를 지정합니다(아래 endstop_home 참조).
* `config_spi oid=%c bus=%u pin=%u mode=%u rate=%u shutdown_msg=%*s` : 이 명령은 내부 SPI 개체를 만듭니다. spi_transfer 및 spi_send 명령과 함께 사용됩니다 (아래 참조). "버스"는 사용할 SPI 버스를 식별합니다(마이크로 컨트롤러에 사용 가능한 SPI 버스가 두 개 이상 있는 경우). "핀"은 장치의 칩 선택(CS) 핀을 지정합니다. "모드"는 SPI 모드입니다 (0과 3 사이여야 함). "rate" 매개변수는 SPI 버스 속도(초당 사이클)를 지정합니다. 마지막으로 "shutdown_msg"는 마이크로 컨트롤러가 종료 상태가 될 경우 지정된 장치에 보내는 SPI 명령입니다.
* `config_spi_without_cs oid=%c bus=%u mode=%u rate=%u shutdown_msg=%*s` : 이 명령은 config_spi와 유사하지만 CS 핀 정의가 없습니다. 칩 선택 라인이 없는 SPI 장치에 유용합니다.

## 일반적인 명령

이 섹션에서는 일반적으로 사용되는 몇 가지 런타임 명령을 나열합니다. Klipper 에 대한 통찰력을 얻으려는 개발자에게만 관심이 있을 것입니다.

* `set_digital_out_pwm_cycle oid=%c cycle_ticks=%u` : 이 명령은 "소프트웨어 PWM"을 사용하도록 디지털 출력 핀(config_digital_out에 의해 생성됨)을 구성합니다. 'cycle_ticks'는 PWM 주기의 클록 틱 수입니다. 출력 스위칭은 마이크로 컨트롤러 소프트웨어에서 구현되기 때문에 'cycle_ticks'는 10ms 이상의 시간에 해당하는 것이 좋습니다.
* `queue_digital_out oid=%c clock=%u on_ticks=%u` : 이 명령은 주어진 클록 시간에 디지털 출력 GPIO 핀에 대한 변경을 예약합니다. 이 명령을 사용하려면 마이크로 컨트롤러 구성 중에 동일한 'oid' 매개변수를 사용하여 'config_digital_out' 명령을 실행해야 합니다.'set_digital_out_pwm_cycle'이 호출된 경우 'on_ticks'는 pwm 주기의 온 지속 시간 (클록 틱 단위)입니다. 그렇지 않으면 'on_ticks'는 0 (low voltage) 또는 1 (high voltage)이어야 합니다.
* `queue_pwm_out oid=%c clock=%u value=%hu` : 하드웨어 PWM 출력 핀에 대한 변경을 예약합니다. 자세한 내용은 'queue_digital_out' 및 'config_pwm_out' 명령을 참조하세요.
* `query_analog_in oid=%c clock=%u sample_ticks=%u sample_count=%c rest_ticks=%u min_value=%hu max_value=%hu` : 이 명령은 아날로그 입력 샘플의 반복 일정을 설정합니다. 이 명령을 사용하려면 마이크로 컨트롤러 구성 중에 동일한 'oid' 매개변수를 사용하여 'config_analog_in' 명령을 실행해야 합니다. 샘플은 '클럭' 시간부터 시작하고, 'rest_ticks' 클럭 틱마다 얻은 값을 보고하고, 'sample_count'번 오버샘플링하고, 'sample_ticks' 클럭 틱 수를 일시 중지합니다. 'min_value' 및 'max_value' 매개변수는 안전 기능을 구현합니다. 마이크로 컨트롤러 소프트웨어는 샘플링된 값(오버샘플링 후)이 항상 제공된 범위 사이에 있는지 확인합니다. 이것은 히터를 제어하는 서미스터에 부착된 핀과 함께 사용하기 위한 것입니다. 히터가 온도 범위 내에 있는지 확인하는 데 사용할 수 있습니다.
* `get_clock` : 이 명령은 마이크로 컨트롤러가 "clock" 응답 메시지를 생성하도록 합니다. 호스트는 이 명령을 1초에 한 번 전송하여 마이크로 컨트롤러 클록의 값을 얻고 호스트와 마이크로 컨트롤러 클록 간의 드리프트를 추정합니다.이를 통해 호스트는 마이크로 컨트롤러 clock을 정확하게 추정할 수 있습니다.

### 스테퍼 명령

* `queue_step oid=%c interval=%u count=%hu add=%hi` : 이 명령은 각 단계 사이의 '간격' 클럭 틱 수와 함께 지정된 스테퍼에 대한 '카운트' 단계 수를 예약합니다. 첫 번째 단계는 주어진 스테퍼에 대해 마지막으로 예약된 단계 이후의 클럭 틱의 '간격' 수입니다. '추가'가 0이 아닌 경우 간격은 각 단계 후에 '추가' 양만큼 조정됩니다. 이 명령은 지정된 간격/카운트/추가 시퀀스를 스테퍼별 대기열에 추가합니다. 정상 작동 중에 이러한 시퀀스가 수백 개 있을 수 있습니다. 새 시퀀스가 대기열 끝에 추가되고 각 시퀀스가 '카운트' 단계 수를 완료할 때 대기열 앞에서 팝됩니다. 이 시스템을 통해 마이크로 컨트롤러는 잠재적으로 수십만 단계를 대기열에 넣을 수 있습니다. 모두 안정적이고 예측 가능한 일정 시간으로 이루어집니다.
* `set_next_step_dir oid=%c dir=%c` : 이 명령은 다음 queue_step 명령이 사용할 dir_pin 값을 지정합니다.
* `reset_step_clock oid=%c clock=%u` : 일반적으로 단계 타이밍은 주어진 스테퍼의 마지막 단계에 상대적입니다. 이 명령은 다음 단계가 제공된 '시계' 시간을 기준으로 하도록 시계를 재설정합니다.호스트는 일반적으로 인쇄 시작 시에만 이 명령을 보냅니다.
* `stepper_get_position oid=%c` : 이 명령은 마이크로 컨트롤러가 스테퍼의 현재 위치와 함께 "stepper_position" 응답 메시지를 생성하도록 합니다. 위치는 dir=1 로 생성된 총 단계 수에서 dir=0 으로 생성된 총 단계 수를 뺀 것입니다.
* `endstop_home oid=%c clock=%u sample_ticks=%u sample_count=%c rest_ticks=%u pin_value=%c` : 이 명령은 스테퍼 "호밍" 작업 중에 사용됩니다. 이 명령을 사용하려면 마이크로 컨트롤러 구성 중에 동일한 'oid' 매개변수를 사용하여 'config_endstop' 명령을 실행해야 합니다. 이 명령이 호출되면 마이크로 컨트롤러는 'rest_ticks' 클록 틱마다 엔드스톱 핀을 샘플링하고 'pin_value'와 같은 값이 있는지 확인합니다. 값이 일치하면 (그리고 'sample_count' 추가 샘플이 'sample_ticks' 간격으로 계속 일치하면) 연결된 스테퍼의 이동 대기열이 지워지고 스테퍼가 즉시 중지됩니다. 호스트는 이 명령을 사용하여 homing을 구현합니다. 호스트는 endstop에 endstop 트리거에 대한 샘플링을 지시한 다음 일련의 queue_step 명령을 실행하여 스테퍼를 endstop으로 이동시킵니다. 스테퍼가 엔드스톱에 도달하면 트리거가 감지되고 이동이 중지되며 호스트에 알림이 표시됩니다.

### Move queue

각 queue_step 명령은 마이크로 컨트롤러 "move queue"의 항목을 활용합니다. 이 큐는 "finalize_config" 명령을 수신할 때 할당되며 "config" 응답 메시지에 사용 가능한 큐 항목 수를 보고합니다.

queue_step 명령을 보내기 전에 큐에 사용 가능한 공간이 있는지 확인하는 것은 호스트의 책임입니다. 호스트는 각 queue_step 명령이 완료될 때를 계산하고 그에 따라 새 queue_step 명령을 예약하여 이를 수행합니다.

### SPI 명령

* `spi_transfer oid=%c data=%*s` : 이 명령은 마이크로 컨트롤러가 'oid'로 지정된 spi 장치에 '데이터'를 보내도록 하고 전송 중에 반환된 데이터로 "spi_transfer_response" 응답 메시지를 생성합니다.
* `spi_send oid=%c data=%*s` : 이 명령은 "spi_transfer"와 유사하지만 "spi_transfer_response" 메시지를 생성하지 않습니다.
