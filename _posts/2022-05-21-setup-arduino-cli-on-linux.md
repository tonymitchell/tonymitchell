Using Arduino-cli...


## Installation

```shell
# apt install arduino

#in WSL then install usbip to access USB devices in the host
#sudo apt install linux-tools-virtual hwdata
#sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*/usbip 20
```




You can using the board details command to figure out the fully qualified board name you want...
e.g. 
```
arduino-cli board listall
```
shows
```
Board Name                       FQBN
ATtiny24/44/84                   attiny:avr:ATtinyX4
ATtiny25/45/85                   attiny:avr:ATtinyX5
Adafruit Circuit Playground      arduino:avr:circuitplay32u4cat
Arduino BT                       arduino:avr:bt
Arduino Duemilanove or Diecimila arduino:avr:diecimila
Arduino Esplora                  arduino:avr:esplora
Arduino Ethernet                 arduino:avr:ethernet
Arduino Fio                      arduino:avr:fio
Arduino Gemma                    arduino:avr:gemma
Arduino Industrial 101           arduino:avr:chiwawa
Arduino Leonardo                 arduino:avr:leonardo
Arduino Leonardo ETH             arduino:avr:leonardoeth
Arduino Mega ADK                 arduino:avr:megaADK
Arduino Mega or Mega 2560        arduino:avr:mega
Arduino Micro                    arduino:avr:micro
Arduino Mini                     arduino:avr:mini
Arduino NG or older              arduino:avr:atmegang
Arduino Nano                     arduino:avr:nano
Arduino Pro or Pro Mini          arduino:avr:pro
Arduino Robot Control            arduino:avr:robotControl
Arduino Robot Motor              arduino:avr:robotMotor
Arduino Uno                      arduino:avr:uno
Arduino Uno WiFi                 arduino:avr:unowifi
Arduino Yún                      arduino:avr:yun
Arduino Yún Mini                 arduino:avr:yunmini
LilyPad Arduino                  arduino:avr:lilypad
LilyPad Arduino USB              arduino:avr:LilyPadUSB
Linino One                       arduino:avr:one
```

I know I want ATtiny85... but the board option `attiny:avr:ATtinyX5` isn't specific enough.  So let's check the board details:

```
arduino-cli board details attiny:avr:ATtinyX5
```
shows
```
Board name:            ATtiny25/45/85
FQBN:                  attiny:avr:ATtinyX5
Board version:         1.0.2

Package name:          attiny
Package maintainer:    David A. Mellis
Package URL:           https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json
Package website:       https://github.com/damellis/attiny

Platform name:         attiny
Platform category:     attiny
Platform architecture: avr
Platform URL:          https://github.com/damellis/attiny/archive/6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform file name:    6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform size (bytes): 5913
Platform checksum:     SHA-256:1654a8968fea7d599255bd18430786f9c84340606e7a678b9cf9a3cd49d94ad7

Option:                Processor                                                                                                cpu
                       ATtiny25                                                                                            ✔    cpu=attiny25
                       ATtiny45                                                                                                 cpu=attiny45
                       ATtiny85                                                                                                 cpu=attiny85
Option:                Clock                                                                                                    clock
                       Internal 1 MHz                                                                                      ✔    clock=internal1
                       Internal 8 MHz                                                                                           clock=internal8
                       Internal 16 MHz                                                                                          clock=internal16
                       External 8 MHz                                                                                           clock=external8
                       External 16 MHz                                                                                          clock=external16
                       External 20 MHz                                                                                          clock=external20
Programmers:           Id                                                                                                  Name
```

Notice the cpu and clock options? Notice has ATtiny25 and internal 1 MHz are selected?  Great if you want that, but what if you don't?  You can add addition options and use the same board details command to confirm you've got it right...


```
arduino-cli board details attiny:avr:ATtinyX5:cpu=attiny85,clock=internal8
```
shows
```
Board name:            ATtiny25/45/85
FQBN:                  attiny:avr:ATtinyX5
Board version:         1.0.2

Package name:          attiny
Package maintainer:    David A. Mellis
Package URL:           https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json
Package website:       https://github.com/damellis/attiny

Platform name:         attiny
Platform category:     attiny
Platform architecture: avr
Platform URL:          https://github.com/damellis/attiny/archive/6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform file name:    6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform size (bytes): 5913
Platform checksum:     SHA-256:1654a8968fea7d599255bd18430786f9c84340606e7a678b9cf9a3cd49d94ad7

Option:                Processor                                                                                                cpu
                       ATtiny25                                                                                                 cpu=attiny25
                       ATtiny45                                                                                                 cpu=attiny45
                       ATtiny85                                                                                            ✔    cpu=attiny85
Option:                Clock                                                                                                    clock
                       Internal 1 MHz                                                                                           clock=internal1
                       Internal 8 MHz                                                                                      ✔    clock=internal8
                       Internal 16 MHz                                                                                          clock=internal16
                       External 8 MHz                                                                                           clock=external8
                       External 16 MHz                                                                                          clock=external16
                       External 20 MHz                                                                                          clock=external20
Programmers:           Id                                                                                                  Name
```


