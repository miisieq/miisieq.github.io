---
layout: post
title: "Jak spartycjonować i sformatować dysk twardy na Linuxie"
tag: linux debian
date: 2021-01-22
---

Formatowanie nowego dysku w przypadku korzystania ze środowiska graficznego jest bardzo intuicyjne. Jednak się komplikuje w przypadku wykonania tej czynności tylko za pomocą powłoki systemowej. W poniższym wpisie pokażę dodać partycję, sformatować ją i zamontować na nowo zakupionym dysku twardym. Do demonstracji użyjemy systemu operacyjnego Debian 10 (Buster) Server.

## Identyfikator dysku
W celu ustalenia identyfikatora dysku, który chcemy sformatować, uzyjemy narzędzia [lsblk](https://man7.org/linux/man-pages/man8/lsblk.8.html), które wypisuje urządzenia blokowe.

```shell
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465,8G  0 disk
└─sda1   8:1    0 465,8G  0 part /mnt/storage
sdb      8:16   0 953,9G  0 disk
sdc      8:32   1  28,7G  0 disk
├─sdc1   8:33   1  27,7G  0 part /
├─sdc2   8:34   1     1K  0 part
└─sdc5   8:37   1   975M  0 part [SWAP]
```

Na pierwszy rzut oka widzimy, że urządzenia *sda* oraz *sdc* posiadają partycje, więc wiemy, że interesuje nasz nowy dysk, który nie ma partycji i jest oznaczony identyfikatorem *sdb*.

## Krok 1: Partycjonowanie

Gdy już znamy identyfikator dysku, możemy przystąpić do partycjonowania.

1. Uruchamiamy narzędzie [fdisk](https://man7.org/linux/man-pages/man8/fdisk.8.html), które posłuży nam do stworzenia tablicy partycji.
1. Wpisujemy polecenie *n*, aby przejść w tryb tworzenia nowej partycji.
1. Na wszystkie pytania odpowiadamy domyślną wartością, czyli wciskamy *ENTER*.
1. Aby zapisać zmiany w tablicy partycji, wciskamy *w*.

```shell
# fdisk /dev/sdb
Witamy w programie fdisk (util-linux 2.33.1).
Zmiany pozostaną tylko w pamięci do chwili ich zapisania.
Przed użyciem polecenia zapisu prosimy o ostrożność.

Urządzenie nie zawiera żadnej znanej tablicy partycji.
Utworzono nową etykietę dysku DOS z identyfikatorem dysku 0xba0a61ef.

Polecenie (m wyświetla pomoc): n
Typ partycji
    p   główna (głównych 0, rozszerzonych 0, wolnych 4)
    e   rozszerzona (kontener na partycje logiczne)
Wybór (domyślnie p): p
Numer partycji (1-4, domyślnie 1):
Pierwszy sektor (2048-2000409263, domyślnie 2048):
Ostatni sektor, +/-sektorów lub +/-rozmiar{K,M,G,T,P} (2048-2000409263, domyślnie 2000409263):

Utworzono nową partycję 1 typu 'Linux' o rozmiarze 953,9 GiB.

Polecenie (m wyświetla pomoc): w
Tablica partycji została zmodyfikowana.
Wywoływanie ioctl() w celu ponownego odczytu tablicy partycji.
Synchronizacja dysków.
```

## Krok 2: Formatowanie

Aby sformatować partycję do formatu ext4 używamy narzędzia [mkfs](https://linux.die.net/man/8/mkfs.ext4).

```shell
# mkfs.ext4 /dev/sdb1
mke2fs 1.44.5 (15-Dec-2018)
Discarding device blocks: done
Creating filesystem with 250050902 4k blocks and 62513152 inodes
Filesystem UUID: 6c819f03-bfe3-45e6-b5a8-fc659295c20e
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
    102400000, 214990848
  
Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

## Krok 3: Montowanie partycji

```shell
# mkdir /mnt/backup
# mount /dev/sdb1 /mnt/backup
```

## Krok 4: Automatyczne montowanie partycji

Aby partycja była automatycznie montowana w systemie, powinniśmy najpierw ustalić jej UUID. W tym celu znowu użyjemy narzędzia [lsblk](https://man7.org/linux/man-pages/man8/lsblk.8.html), ale tym razem z dodatkowym parametrem:
```shell
# lsblk -f /dev/sdb1
NAME FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sdb1 ext4         6c819f03-bfe3-45e6-b5a8-fc659295c20e  890,1G     0% /mnt/backup
```

Gdy już ustalilismy UUID, otwieramy plik do edycji:
```shell
# vim /etc/fstab
```

i na końcu pliku dodajemy poniższą linię:
```shell
UUID=6c819f03-bfe3-45e6-b5a8-fc659295c20e	/mnt/backup	ext4	defaults	0	0
```

## Krok 5: Restart systemu (opcjonalny)
Na koniec możemy zrestartować system, aby upewnić się, że konfiguracja jest poprawna, a nowo dodana partycja poprawnie się zamontuje.

W celu bezpiecznego restartu systemu użyjemy narzędzia [shutdown](https://linux.die.net/man/8/shutdown):
```shell
# shutdown -R
```