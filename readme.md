# Голосовой ассистент для управления умным домом на базе MajorDomo.ru #
***
**Возможности**
1. Запуск распознавания по любому ключевому слову (Привет Алиса, проснись дом, Ok Google и тд.)
2. Передача команды в МДМ
3. Ответ в терминал через sayReply
4. Подключение светодиодов (в разработке)
***Внимание!!!***
На данный момент протестировано на платах:
* ***Orange Pi Zero*** - Armbian Ubuntu_xenial_default ***ядро 3.4***
## Подготовка 
Скачать и записать образ системы.
После первой загрузки системы идет долгое обновление всех пакетов, проверить можно через команду htop, будет запущен процесс dkpg  

Если ситема на Armbian Xenial mainline kernel 4.14
Обновите всю систему через apt upgrade
нужно зайти в armbian-config -> System -> Hardware: выбрать analog-audio (пробелом)
запустить alsamixer, под каждым миксером проверить, что буквы ММ(XX) горят зеленым, если нет включить (клавиша M) 
за громкость звука отвечает 2 ползунка  Line Out и DAC 

по не понятным причинам, если выключить mic 1 (встроеный микрфон, то пропадает звук на первые 2-3 секунды) 

Сохранить настройки Alsa
```
sudo alsactl store 0
```

Отредактировать или создать файл /etc/asound.conf
Заранее проверить на каком усройстве микрофон и динмаик через команду 
aplay -l 
arecored -l
и отредактировать если нужно в hw:1,0 

```
sudo nano /etc/asound.conf

```
Вставить код 

```
pcm.!default {
type asym
playback.pcm "playback"
capture.pcm "capture"
}

pcm.playback {
type plug
slave.pcm "dmixed"
}

pcm.capture {
type plug
slave.pcm "array"
}

pcm.dmixed {
type dmix
slave.pcm "hw:0,0"
ipc_key 555555
}

pcm.array {
type dsnoop
slave {
pcm "hw:1,0"
channels 2
}
ipc_key 666666
}

```
сохранить и проверить вывод звука через команду  speaker-test



*************************************************
## **Установка** 
*************************************************
1. Откройте терминал и выполните команды
```
cd ~/
git clone https://github.com/devoff/mdmPiTerminal
cd mdmPiTerminal
chmod +x scripts/mdm-pi-installer.sh
./scripts/mdm-pi-installer.sh
```
Ждем пока установится SnowBoy и все зависимости. 

**************************************************
## **Первый запуск, подготовка терминала** 
**************************************************

```
chmod +x systemd/service-installer.sh
sudo ./systemd/service-installer.sh

sudo systemctl enable mdmpiterminal.service
sudo systemctl enable mdmpiterminalsayreply.service

sudo systemctl start mdmpiterminalsayreply.service 
```
При первом запуске терминал скажет свой IP адрес, который нужно будет добавить в МДМ (если не успели записать, воспользуйтесь командой ifconfig) 


*************************************************
## **Настройка терминала** 
************************************************

Заходим в Панель управления MajorDomo > Система > Маркет дополнений > Оборудование > MDM VoiceAssistant и устанавливаем модуль. 

После заходим - в Настройки > Терминалы > Добавить новую запись > Добавляем название и IP адрес терминала 



Переходим в Устройства >  MDM VoiceAssistant
Выбираем терминал, который только, что создали.
Выбираем Сервис синтеза речи:
Если есть API ключ от Яндекса, лучше выбрать Yandex TTS, если нет то Google
Чувствительность реагирования на ключевое слово - чем больше тем лучше слышит, но будет много ложных срабатываней.  
Сервис распознования речи - Выбираем Google, так же можно попробовать wit.ai или микрософт, но для них нужно получить API/

Сохраняем

*************************************************
## **Запись ключевого слова** 
*************************************************

Я бы рекомендовал записать образцы голоса сразу на терминале с тем микрофоном, который будет использоваться, либо записать на сайте snowboy
для этого нужно получиться пройти регистрация на сайте https://snowboy.kitt.ai/

В МДМ в настройках модуля, во вкладка запись ключевого слово нажимаем - ЗАПИСЬ, на терминале включится запись на 5 секунд и автоматически завершиться, 
нужно повторить эту процендуру 3 раза  после отправить на компиляцию. 

После записать дополнительные ключевые слова. 

*************************************************
## **Запуск терминала ** 
*************************************************

```
sudo systemctl start mdmpiterminal.service
```

*************************************************
## **Отладка** 
*************************************************
Для ручного запуска скриптом терминала - 


Остановить сервисы 

```
sudo service mdmpiterminal stop
sudo service mdmpiterminalsayreply stop
```

```
cd ~/
mdmPiTerminal/env/bin/python -u mdmPiTerminal/src/snowboy.py // для запуская сервиса распознования ключевого слова 
mdmPiTerminal/env/bin/python -u mdmPiTerminal/src/sayreply.py // Сервис для ответов от МДМ получение и обработка настроек. 
```

*************************************************
## **Решения проблем** 
*************************************************
1. Если не работает USB микрофон, попробуйте выдыренуть и вставить обратно, иногда это помогает. 
