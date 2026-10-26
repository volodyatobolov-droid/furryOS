[README.md](https://github.com/user-attachments/files/33967833/README.md)
# furryOS

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
