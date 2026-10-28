[README.md](https://github.com/user-attachments/files/34067597/README.md)
[README.md](https://github.com/user-attachments/files/33967833/README.md)
# furryOS
# furryOS

**Arch-based · Hyprland · pink paws**

[![GitHub](https://img.shields.io/badge/GitHub-repo-181717?logo=github)](https://github.com/volodyatobolov-droid/furryOS)
[![Discord](https://img.shields.io/badge/Discord-сообщество-5865F2?logo=discord&logoColor=white)](https://discord.gg/48aRJRcy6S)

Арчеподобный дистрибутив. ver 1 — брендированный live/install ISO
с Hyprland. Пакеты берём из Arch `core`/`extra`, а хотелки — из AUR

## Требования к хосту

```bash
sudo pacman -S --needed archiso squashfs-tools mtools dosfstools libisoburn
```

Для макета интерфейса: `qt6-declarative`.

## Сборка

```bash
./build.sh
```

Результат: `out/furryos-YYYY.MM.DD-x86_64.iso`

`build.sh` каждый раз пересобирает профиль `iso/furryos/` из ванильного
`releng` + наш оверлей. **Руками профиль не правим** — правки идут в
`iso/overlay/` и `scripts/init-profile.sh`.

## Тест

```bash
# BIOS
sudo run_archiso -i out/furryos-*.iso
# UEFI
sudo run_archiso -u -i out/furryos-*.iso
```

## Запись на флешку

```bash
sudo dd if=out/furryos-*.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

## Что внутри

| Путь | Зачем |
|---|---|
| `build.sh` | сборка ISO |
| `scripts/init-profile.sh` | генерация профиля archiso из `releng` + оверлей |
| `scripts/preview-ui.sh` | живой макет установщика (QML, без ISO) |
| `scripts/preview/` | заглушки Calamares API для макета |
| `scripts/dev-session.sh` | вложенный Hyprland для разработки дотфайлов |
| `scripts/vm.sh` | запуск ISO/диска в QEMU (KVM или TCG) |
| `CHECKPOINT.md` | где остановились и что делать дальше |
| `iso/packages.extra` | доп. пакеты поверх releng |
| `iso/overlay/airootfs/` | файлы, которые лягут в корень live-системы |
| `iso/overlay/airootfs/etc/skel/` | дефолтные конфиги (Hyprland, waybar, kitty) |
| `iso/overlay/airootfs/etc/calamares/` | конфиг и брендинг установщика |
| `iso/overlay/airootfs/usr/lib/furryos/` | свой модуль `furryos-pacstrap` |

## Live-пользователь

- логин: `furry` (автологин на tty1, сразу Hyprland)
- пароль: пустой
- `sudo` без пароля (только в live-режиме)

## Calamares — установка настоящая

Ставим через **pacstrap** (нужен интернет), а не unpackfs: так в системе
оказываются свежие пакеты из Arch, а ISO не пухнет.

Порядок:

| Шаг | Что делает |
|---|---|
| `welcome` | проверки: ОЗУ, диск, **интернет** (красный пункт = дальше нельзя) |
| `locale`, `keyboard`, `partition`, `users`, `summary` | обычный мастер |
| `furryos-pacstrap` | свой Python-модуль: `pacstrap` базы + брендинг |
| `machineid`, `locale`, `localecfg`, `fstab`, `initcpio` | штатные модули |
| `users`, `networkcfg`, `displaymanager` | юзер, сеть, sddm |
| `grubcfg`, `bootloader` | ванильный GRUB (UEFI и BIOS) |

Свой модуль лежит в `iso/overlay/airootfs/usr/lib/furryos/calamares/modules/furryos-pacstrap/`
и подключён через `modules-search`, поэтому `cachyos-calamares` его не перетирает.

Сеть:

- в live включён `NetworkManager` (networkd/iwd/resolved выключены);
- `furryos-netcheck` до запуска установщика проверяет интернет и при
  отсутствии открывает `nmtui` в kitty;
- значок сети в waybar тоже открывает `nmtui`.

Запуск установщика:

- автоматически при логине (`exec-once` в `hyprland.conf`)
- вручную: `sudo furryos-installer` или `SUPER+I`

## Предпросмотр интерфейса (без ISO и без Calamares)

```bash
./scripts/preview-ui.sh
```

Открывает окно 1180×720 с **настоящими** `calamares-sidebar.qml` и
`calamares-navigation.qml`. Шаги в сайдбаре и кнопки «Назад / Поехали /
Выход» кликаются, страницы пока пустые.

Нужен `qt6-declarative` (там `qml6`). Скриншот: `docs/preview-installer.png`.

## Дотфайлы без VM

Вложенный Hyprland с конфигами прямо из репозитория:

```bash
./scripts/dev-session.sh          # запустить окном
./scripts/dev-session.sh --check  # только проверить конфиг
./scripts/dev-session.sh --push   # вернуть правки в репо
./scripts/dev-session.sh --pull   # обновить копию из репо
```

Копия конфигов живёт в `~/.cache/furryos-dev`, репозиторий не трогается
до `--push`.

## VM

```bash
sudo pacman -S --needed qemu-full edk2-ovmf
./scripts/vm.sh install out/furryos-*.iso   # загрузка с ISO и установка
./scripts/vm.sh run                         # загрузка установленного диска
```

Диск: `vm/furryos.qcow2` (30 ГБ). SSH: `ssh -p 2222 <user>@localhost`.
Общая папка с репо внутри гостя: `mount -t 9p -o trans=virtio furryos /mnt`.
Без `/dev/kvm` QEMU работает в TCG — медленно; включи VT-x в BIOS.

## Конфиги и обновление

Дефолтные конфиги (Hyprland, waybar, kitty, fastfetch) лежат в
`iso/overlay/airootfs/etc/skel/.config` и попадают в систему через
модуль `users` (Calamares копирует `/etc/skel` в `/home/<user>`).

Обновить их из GitHub без пересборки ISO:

```bash
sudo furryos-config-update
```

Скрипт качает `iso/overlay/airootfs/etc/skel` из репозитория
`FURRYOS_CONFIG_REPO` (по умолчанию `volodyatobolov-droid/furryOS`),
делает бэкап `~/.config-backup-YYYYMMDD-HHMMSS` и раскладывает свежее.

## Ограничения

- В `cachyos-calamares` нет переводов (`.qm`), поэтому системные страницы
  Calamares — на английском. Наши сайдбар/навигация/заставка — на русском.
- `pacstrap` требует интернет: без сети установка не начнётся.
- Secure Boot не поддерживаем (нужна своя подпись/шима).

## Картинка при загрузке (Plymouth)

Тема: `iso/overlay/airootfs/usr/share/plymouth/themes/furryos/`
(белая лапа на тёмном фоне, модуль `two-step`). Скрин: `docs/preview-boot.png`.

Как включено:

- пакет `plymouth` (в live и в target);
- хук `plymouth` в `HOOKS` — в live через `archiso.conf`, в установленной
  системе через `/etc/mkinitcpio.conf.d/10-furryos.conf`;
- `quiet splash vt.global_cursor_default=0` в параметрах ядра
  (live — `kernel_params_x86_64`, target — `furryos-grubcfg.conf`);
- `/etc/plymouth/plymouthd.conf` с `Theme=furryos`.

Поменять картинку: заменить `throbber-00.png` / `animation-00.png`
(и их 4-значные копии) в теме и пересобрать initramfs:

```bash
sudo mkinitcpio -P
```

Про «лого ядра»: встроенный Tux (`logo_linux_clut224.ppm`) виден только
до Plymouth и только на framebuffer-консоли. Чтобы его заменить, нужно
пересобирать пакет `linux` с подменённым файлом в `drivers/video/logo/` —
это отдельная ветка. Plymouth закрывает этот этап полностью.

## Лого

Paw-лого лежит в `iso/overlay/airootfs/usr/share/furryos/logo.txt`
(симметричное, 30 колонок). Генерится из картинки:

```bash
./scripts/paw2blocks.py logo.png -w 30 -m
```


## Про AUR

1. **chaotic-aur** — готовые бинарники из AUR, подключается двумя строками в `pacman.conf`.
2. **paru** — собирает AUR локально (долго, зато всё свежее).
3. **свой `[furryos]`** — CI собирает AUR-пакеты, подписывает и раздаёт. Это уже «свой репо, наполненный из AUR».

При раздаче собранных AUR-пакетов не забываем про лицензии (GPL → исходники).

Арчеподобный дистрибутив. ver 1 — брендированный live/install ISO
с Hyprland. Пакеты берём из Arch `core`/`extra`, а хотелки — из AUR

## Требования к хосту

```bash
sudo pacman -S --needed archiso squashfs-tools mtools dosfstools libisoburn
```

## Сборка

```bash
./build.sh
```

Результат: `out/furryos-YYYY.MM.DD-x86_64.iso`

`build.sh` каждый раз пересобирает профиль `iso/furryos/` из ванильного
`releng` + наш оверлей. **Руками профиль не правим** — правки идут в
`iso/overlay/` и `scripts/init-profile.sh`.

## Тест

```bash
# BIOS
sudo run_archiso -i out/furryos-*.iso
# UEFI
sudo run_archiso -u -i out/furryos-*.iso
```

## Запись на флешку

```bash
sudo dd if=out/furryos-*.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

## Что внутри

| Путь | Зачем |
|---|---|
| `build.sh` | сборка ISO |
| `scripts/init-profile.sh` | генерация профиля archiso из `releng` + оверлей |
| `iso/packages.extra` | доп. пакеты поверх releng |
| `iso/overlay/airootfs/` | файлы, которые лягут в корень live-системы |
| `iso/overlay/airootfs/etc/skel/` | дефолтные конфиги (Hyprland, waybar, kitty) |

## Live-пользователь

- логин: `furry` (автологин на tty1, сразу Hyprland)
- пароль: пустой
- `sudo` без пароля (только в live-режиме)

## Calamares

`cachyos-calamares` 3.4.2 ставится **из репы CachyOS, только на время сборки**
(в `/etc/pacman.conf` самого ISO этот блок не попадает — проверено).

Сейчас это **болванка**: мастер проходит welcome → locale → keyboard →
partition → users → summary → finished, но **диск не трогает** (в `exec`
стоит `shellprocess`-заглушка). Реальную установку вайрим следующим шагом.

Запуск:

- автоматически при логине (`exec-once` в `hyprland.conf`)
- вручную: `sudo furryos-installer` или `SUPER+I`

## Лого

Paw-лого лежит в `iso/overlay/airootfs/usr/share/furryos/logo.txt`
(симметричное, 30 колонок). Генерится из картинки:

```bash
./scripts/paw2blocks.py logo.png -w 30 -m
```


## Про AUR

AUR — это PKGBUILD'ы, а не бинарники. Варианты:

1. **chaotic-aur** — готовые бинарники из AUR, подключается двумя строками в `pacman.conf`.
2. **paru** — собирает AUR локально (долго, зато всё свежее).
3. **свой `[furryos]`** — CI собирает AUR-пакеты, подписывает и раздаёт. Это уже «свой репо, наполненный из AUR».

При раздаче собранных AUR-пакетов не забываем про лицензии (GPL → исходники).
