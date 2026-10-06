---
name: circle-to-search
description: "Enable Google's native Circle to Search on non-Pixel/non-Samsung Android phones (Xiaomi/HyperOS/MIUI, POCO, Redmi, Motorola) without root: install CTSLauncher and grant it WRITE_SECURE_SETTINGS over ADB (on-phone via Wireless debugging + Shizuku + aShell, or from a PC over USB), then bind it to power-button hold, back tap or a Quick Settings tile. Use when the user wants Circle to Search, screen search or translate-the-screen back after Google Assistant was replaced by Gemini, or hits 'Error: Stream closed' on HyperOS. Triggers: «Circle to Search», «обвести и найти», «поиск по экрану», «вернуть ассистента Google», «CTSLauncher», «pm grant WRITE_SECURE_SETTINGS», «Stream closed Xiaomi»."
---

# Circle to Search на любом Android через ADB (CTSLauncher)

Инструкция для ИИ-агента. Гайд для людей со скриншотами: [README](https://github.com/Eniggman/circle-to-search#readme).

Суть: Circle to Search уже встроен в приложение Google. Открытое приложение **CTSLauncher** (`com.chiller3.ctslauncher`) вызывает его системным интентом, но для этого ему нужно право `WRITE_SECURE_SETTINGS`, которое выдаётся только через ADB. Root и смена прошивки не нужны.

## Важно для агента

- Телефон физически у человека. Все действия на экране телефона (настройки, подтверждения, коды сопряжения) делает **человек**. Ты даёшь пошаговые указания и проверяешь результат командами.
- **Перед любой ADB-командой, которая что-то меняет на телефоне, коротко скажи, что она делает, и получи «да».** Это касается `pm grant`, `pm uninstall`, `pm disable-user` и `settings put`.
- Включение «Отладки по USB (настройки безопасности)» на Xiaomi требует SIM-карты и входа в Mi-аккаунт и показывает предупреждения безопасности. Объясни это человеку, решение за ним.

## Шаг 0. Что установить на телефон (делает человек)

1. CTSLauncher: https://github.com/chenxiaolong/CTSLauncher/releases
2. Для способа без ПК: Shizuku (https://github.com/RikkaApps/Shizuku/releases) и aShell You (https://github.com/DP-Hridayan/aShellYou/releases) или Termux.
3. Приложение Google должно быть установлено и обновлено.

## Шаг 1. Режим разработчика (делает человек)

1. «Настройки → О телефоне», 7 раз нажать на «Версия ОС» (HyperOS/MIUI).
2. «Настройки → Расширенные настройки → Для разработчиков», включить:
   - «Отладка по USB»;
   - «Отладка по Wi-Fi» (для способа без ПК);
   - **«Отладка по USB (настройки безопасности)»**: на Xiaomi/HyperOS это обязательно, иначе будет `Error: Stream closed`.

## Способ A: с компьютера по USB (когда агент работает на ПК)

1. Проверь, что ADB установлен: `adb version`. Если его нет, попроси человека поставить Android SDK Platform-Tools.
2. Попроси подключить телефон кабелем и на телефоне нажать «Разрешить отладку по USB» (с галочкой «Всегда разрешать»).
3. Проверь, что устройство видно:
   ```bash
   adb devices            # статус должен быть "device", а не "unauthorized"
   adb shell pm list packages | grep ctslauncher   # должен вывести package:com.chiller3.ctslauncher
   ```
   В Windows вместо `grep` используй `findstr`.
4. После подтверждения человека выдай право:
   ```bash
   adb shell pm grant com.chiller3.ctslauncher android.permission.WRITE_SECURE_SETTINGS
   ```
   При успехе команда ничего не выводит (код возврата 0).

## Способ B: на самом телефоне, без ПК (Wi-Fi + Shizuku + aShell)

Агент не может выполнять команды сам, поэтому веди человека по шагам:
1. «Отладка по Wi-Fi → Подключить устройство с помощью кода сопряжения». Код и порт ввести в уведомлении Shizuku (или в aShell).
2. В Shizuku нажать «Запустить» в блоке «Запуск через отладку по Wi-Fi» и дождаться статуса «Shizuku запущен». В «Авторизованных приложениях» разрешить aShell.
3. В aShell выбрать карточку **«Локальный ADB»** (не «Беспроводная отладка»: Shizuku уже работает, повторное сопряжение не нужно) и нажать «Начать».
4. Ввести команду (без `adb shell`):
   ```bash
   pm grant com.chiller3.ctslauncher android.permission.WRITE_SECURE_SETTINGS
   ```

## Шаг 2. Проверка, что право выдано

```bash
adb shell dumpsys package com.chiller3.ctslauncher | grep WRITE_SECURE_SETTINGS
# ожидается: android.permission.WRITE_SECURE_SETTINGS: granted=true
```
В способе B попроси человека выполнить в aShell ту же команду без `adb shell`.

## Шаг 3. Как вызывать (делает человек в настройках)

1. **Удержание кнопки питания 0.5 с**: «Расширенные настройки → Функции жестов → Запуск Google Ассистента» (или «Нажатие кнопки питания») → «Удерживайте кнопку питания 0.5 с». Затем «Приложения → Приложения по умолчанию → Цифровой ассистент» → выбрать **CTSLauncher**. Другой вариант: оставить Google или Gemini и включить в CTSLauncher опцию авто-переключения ассистента (*Auto-switch assistant app*).
2. **Back Tap**: «Функции жестов → Постукивание по задней панели» → двойное или тройное → CTSLauncher.
3. **Плитка в шторке**: редактирование плиток → перетащить плитку Circle to Search / CTS.

Свайп из нижнего угла **не будет** вызывать CTSLauncher: в HyperOS и POCO Launcher он привязан к голосовой службе (`voice_interaction_service`), а CTSLauncher — это Activity. Не трать время на этот вариант и предложи один из трёх способов выше.

## Шаг 4. Финальная проверка

Попроси человека открыть любое приложение и сработать выбранным жестом. Должен появиться оверлей Circle to Search (обвести, найти, перевести).

## Частые ошибки

| Симптом | Причина и решение |
|---|---|
| `Error: Stream closed` при `pm grant` | Xiaomi/HyperOS блокирует выдачу защищённых прав. Нужно вставить SIM и включить мобильный интернет, войти в Mi-аккаунт, включить «Отладка по USB (настройки безопасности)» (3 окна с таймером, «Далее» и «Принять») и повторить команду |
| `adb devices` показывает `unauthorized` | На телефоне не подтверждён запрос отладки: переподключи кабель и попроси нажать «Разрешить» |
| `Unknown package: com.chiller3.ctslauncher` | CTSLauncher не установлен: вернись к шагу 0 |
| Shizuku не стартует | Wi-Fi отладка выключилась (после смены сети или перезагрузки): включи её снова и повтори сопряжение |
| Жест не срабатывает | CTSLauncher не выбран цифровым ассистентом, или используется угловой свайп (не поддерживается) |

## Опционально: твики из README (только по явной просьбе)

В README есть бонусные команды. **Каждую выполняй только после явного согласия человека** и объясняй последствия:
- Удаление системных приложений Xiaomi для текущего пользователя: `pm uninstall -k --user 0 com.miui.msa.global`, `com.miui.analytics`, `com.xiaomi.mipicks`, `com.xiaomi.glgm`. Вернуть пакет можно командой `cmd package install-existing <пакет>`.
- Фиксация 120 Гц: `settings put system min_refresh_rate 120` и `settings put system user_refresh_rate 120`. Это повышает расход батареи.
- Отключение Joyose (троттлинг в играх): `pm disable-user --user 0 com.xiaomi.joyose`, вернуть: `pm enable com.xiaomi.joyose`. Без троттлинга телефон сильнее греется.
