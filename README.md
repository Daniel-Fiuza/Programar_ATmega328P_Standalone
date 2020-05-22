# Tutorial Configuração do ATmega328P Standalone com USBAsp no Ubuntu 18.04
## Configuração de Hardware
A imagem a seguir ilustra a configuração necessária para programar o ATmega328P standalone.
![](/Imagens/Standalone_atmega328p.jpg)

Já a conexão do microcontrolador com o gravador USBasp deve ser realizada da seguinte forma:
![](/Imagens/conexão_USBasp_atmega328p.png)
## Configurando Porta Serial no Ubuntu 18.04
Primeiro verifique porta serial:
```
sudo lsusb
```
Deve aparecer algo do tipo:

> Bus 001 Device 022: ID 16c0:05dc Van Ooijen Technische Informatica shared ID for use with libusb

Agora habilite (quase todas) as permissões para este dispositivo:
```
sudo chmod 666 /dev/bus/usb/001/022 
```
onde 001 é o barramento encontrado e 022 o dispositivo. Você pode ver as permissões concedidas:
```
ls -al /dev/bus/usb/001/022
```

## Usando AVRDUDE
Para verificar informaçes do dispostivo:
```
avrdude -c usbasp -p m328p -F -P usb -v
```
Para mais comandos do avrdude veja o [manual](/Documentos/Manual_avrdude.pdf).
### Resultado de um Microcontrolador Danificado
Devido algum problema no dispositivo o avrdude não consegue verificar a assinatura padrão do chip e a seguinte mensagem aparece:

> avrdude: warning: cannot set sck period. please check for usbasp firmware update.
avrdude: error: program enable: target doesn't answer. 1 
avrdude: initialization failed, rc=-1
avrdude: AVR device initialized and ready to accept instructions
avrdude: Device signature = 0x10c214
avrdude: Expected signature for ATmega328P is 1E 95 0F

> avrdude done.  Thank you.

Toda a conexão de hardware foi verificada, inclusive as tensões de alimentação e estava correta. Provavelmente, o chip foi queimado e não funciona mesmo nas configurações adequadas de hardware. Tentarei a programação paralela como método de recuperação.

#### Possíveis Causas para Problemas Apresentados:
* Problema de fiação
* A programação serial foi desativada (requer programador paralelo, tão improvável)
* O pino de redefinição foi desativado pelo fusível RSTDISBL
* O debugWire foi ativado por um depurador ou programado acidentalmente
* O relógio externo foi selecionado anteriormente através de fusíveis, mas nenhum relógio está presente
* O oscilador externo de cristal / cerâmica foi selecionado anteriormente através de fusíveis, mas nenhum cristal está conectado
* Microcontrolador AVR queimado
* Programador morto

### Resultado de um Microcontrolador Bom
Utilizando outro ATmega328P-PU (com a mesma configuração de hardware), porém completamente funcional, obtive o seguinte resultado:

>  avrdude: warning: cannot set sck period. please check for usbasp firmware update.
avrdude: AVR device initialized and ready to accept instructions

> Reading | ################################################## | 100% 0.00s

> avrdude: Device signature = 0x1e950f (probably m328p)

> avrdude: safemode: Fuses OK (E:FF, H:D9, L:F7)

> avrdude done.  Thank you.

### Informações sobre Fuses do ATMEGA328P
Você pode verificar no [datasheet](/Documentos/Datasheet_ATmega328P.pdf) (pág 242 - Fuse Bits).

Todos os ATmega328P vem com a assinatura: **0x1E 0x95 0x0F**, por padrão (pág 244).

A configuração padrão dos fusíveis no Atmega328P são: **LF:0x62 HF:0xD9 EF:0xFF**.

Ao utilizar um cristal oscilador externo (16 MHz), deve-se alterar os fusíveis para: **LF:0xF7 HF:0xD9 EF:0xFF**.

Caso necessite reprogramar os fusíveis pode-se utilizar a programação paralela (pág 245 - 27.6 Parallel Programming Parameters, Pin Mapping, and Commands e pág 252 - Figure 27-5. Programming the FUSES Waveforms).

### Configurando Fusível e Gravando Código no Microcontrolador
Como informado anteriormente, deve-se alterar apenas o fusível LF, pois os demais permanecem iguais:
```
avrdude -c usbasp -p m328p -P usb -b 19200 -u -U lfuse:w:0xf7:m
```
Esse comando escreve no low fuse o byte 0xf7 imediatamente. O argumento *'-u'* impede que os fusíveis sejam programados incorretamente caso não esteja utilizando uma fonte externa, e sim, a tensão do programador USBasp. Agora, podemos gravar o código no dispositivo com o seguinte comando:
```
avrdude -c usbasp -p m328p -P usb -b 19200 -U flash:w:Blink_pin_13.ino.with_bootloader.hex
```
Pronto! O código foi carregado e deve estar rodando no dispositivo.
Saída esperada desse comando:
> avrdude: warning: cannot set sck period. please check for usbasp firmware update.
avrdude: AVR device initialized and ready to accept instructions

> Reading | ################################################## | 100% 0.01s

> avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: warning: cannot set sck period. please check for usbasp firmware update.
avrdude: reading input file "Blink_pin_13.ino.with_bootloader.hex"
avrdude: input file Blink_pin_13.ino.with_bootloader.hex auto detected as Intel Hex
avrdude: writing flash (32768 bytes):

> Writing | ################################################## | 100% 0.60s

> avrdude: 32768 bytes of flash written
avrdude: verifying flash memory against Blink_pin_13.ino.with_bootloader.hex:
avrdude: load data flash data from input file Blink_pin_13.ino.with_bootloader.hex:
avrdude: input file Blink_pin_13.ino.with_bootloader.hex auto detected as Intel Hex
avrdude: input file Blink_pin_13.ino.with_bootloader.hex contains 32768 bytes
avrdude: reading on-chip flash data:

> Reading | ################################################## | 100% 0.52s

> avrdude: verifying ...
avrdude: 32768 bytes of flash verified

> avrdude done.  Thank you.

## Referências
[Gravando Arquivo HEX com avrdude](https://www.elecrom.com/avrdude-tutorial-burning-hex-files-using-usbasp-and-avrdude/)

[Configurar Fusíveis de um ATmega328P](https://www.instructables.com/id/How-to-change-fuse-bits-of-AVR-Atmega328p-8bit-mic/)
