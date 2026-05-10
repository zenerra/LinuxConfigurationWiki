# LinuxConfigurationWiki

---

## 1. Lemezek ellenőrzése és formázása

* **`lsblk`**: A csatlakoztatott lemezek és tárolók (SDC, SDD, SDE) listázása.
* **`sudo cfdisk /dev/sdc`**: Az SDC lemez particionálása `cfdisk` segítségével. A folyamat során:
  * GPT partíciós tábla kiválasztása.
  * 50 GB-os (`/dev/sdc1`) és 10 GB-os (`/dev/sdc2`) partíciók létrehozása.
  * A partíciók típusának módosítása `Linux RAID` típusra.
  * A módosítások mentése (`Write`).

## 2. A partíciók átmásolása a többi lemezre

* **`sudo sfdisk -d /dev/sdc | sudo sfdisk /dev/sdd`**: A partíciós tábla átmásolása az SDC-ről az SDD-re.
* **`sudo sfdisk -d /dev/sdc | sudo sfdisk /dev/sde`**: A partíciós tábla átmásolása az SDC-ről az SDE-re.
* **`lsblk`**: A lemezek és partíciók újbóli ellenőrzése.

## 3. A RAID tömbök (MD50, MD10) létrehozása

* **`sudo mdadm --create /dev/md50 --level=1 --raid-devices=2 /dev/sdc1 /dev/sdd1 --spare-devices=1 /dev/sde1`**: Az MD50 (50 GB) RAID1 tömb létrehozása 2 aktív lemezzel és 1 tartalék lemezzel.
* **`sudo mdadm --create /dev/md10 --level=1 --raid-devices=2 /dev/sdc2 /dev/sdd2 --spare-devices=1 /dev/sde2`**: Az MD10 (10 GB) RAID1 tömb létrehozása 2 aktív lemezzel és 1 tartalék lemezzel.
* **`cat /proc/mdstat`**: A RAID szinkronizáció és állapotának lekérdezése.

## 4. Fájlrendszerek létrehozása (Ext4)

* **`sudo mkfs.ext4 /dev/md50`**: Ext4 fájlrendszer létrehozása az MD50 tömbön.
* **`sudo mkfs.ext4 /dev/md10`**: Ext4 fájlrendszer létrehozása az MD10 tömbön.

## 5. Mappák létrehozása és felcsatolás (Mount)

* **`sudo mkdir /backup50`**: Mappa létrehozása az MD50 csatolásához.
* **`sudo mkdir /backup10`**: Mappa létrehozása az MD10 csatolásához.
* **`sudo mount /dev/md50 /backup50`**: Az MD50 tömb felcsatolása.
* **`sudo mount /dev/md10 /backup10`**: Az MD10 tömb felcsatolása.
* **`lsblk`** (vagy **`df -h`**): A sikeres felcsatolások ellenőrzése.

## 6. Automatikus felcsatolás (Fstab konfigurálása)

* **`sudo cp /etc/mdadm/mdadm.conf /etc/mdadm/mdadm.conf.bak`**: Biztonsági másolat készítése az `mdadm.conf` fájlról.
* **`sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf`**: Az új RAID tömbök adatainak mentése az `mdadm.conf` fájlba.
* **`sudo update-initramfs -u`**: Az initramfs frissítése az új RAID konfigurációval.
* **`sudo nano /etc/fstab`**: Az `/etc/fstab` szerkesztése az automatikus csatoláshoz (hozzáadandó sorok: `/dev/md50 /backup50 ext4 defaults 0 0` és `/dev/md10 /backup10 ext4 defaults 0 0`).
* **`sudo mount -a`**: Az fstab módosítások ellenőrzése hibák nélkül.

## 7. Biztonsági másolat és fájlkezelés

* **`sudo cp -R /etc/skel /backup50/etc_skel_mentes`**: Az `/etc/skel` (vagy a feladatban meghatározott `/etc` mappa) átmásolása az MD50 tömbre.
* **`sudo cp -R /etc/ssh /backup10/ssh_mentes`**: Az `/etc/ssh` (vagy más meghatározott mappa) átmásolása az MD10 tömbre.
* **`reboot`**: A rendszer újraindítása az fstab és a RAID megfelelő betöltődésének ellenőrzésére.

---

## 8. Fájlrendszer, könyvtárak és szövegkezelés
Ezek a parancsok a fájlok mozgatásához, linkeléséhez és tartalmuk szűréséhez elengedhetetlenek.

* **`mkdir -p`**: Könyvtárszerkezet létrehozására szolgál egy lépésben (pl. `/home/mars/egeszseg/backup/...` létrehozása maximum 4 lépésben).
* **`cp -r`**: Könyvtárak rekurzív másolására használatos, például amikor biztonsági mentést kell készíteni az `/etc` mappáról.
* **`mv`**: Fájlok és mappák mozgatására, illetve átnevezésére használható (pl. az `etc_mentes` átnevezése és visszamozgatása).
* **`rm -r`** és **`rm`**: Fájlok vagy teljes könyvtárak (a `-r` kapcsolóval) törlésére szolgál az eredeti helyükről.
* **`ln -s`**: Szimbolikus link (symlink) létrehozására használatos, például hogy a gyökérből is látszódjon egy mappa tartalma.
* **`ln`**: Hard link létrehozására alkalmas két fájl között (pl. a `passwd` fájl linkelése a `/tmp` mappába).
* **`cat`**: Fájlok tartalmának kiíratására vagy létrehozására használható a parancssorból.
* **`>`**, **`>>`**, **`2>`**, **`&>`** *(Átirányítások)*: Létfontosságúak az eredmények és hibák fájlba irányításához: a `>` felülírja, a `>>` hozzáfűzi a kimenetet, a `2>` a hibákat, az `&>` pedig a helyes eredményt és a hibát is egy fájlba tereli.
* **`sort`** és **`uniq`**: A szöveges tartalmak ábécé szerinti rendezésére és az ismétlődések kiszűrésére valók.
* **`wc -l`**: A sorok számának megszámlálására használható egy fájlon belül.
* **`grep`**: Szöveges szűrésre alkalmazható, például az időzónák közötti keresésre vagy üres sorok és kommentek eltüntetésére egy konfigurációs fájlból.
* **`sha512sum`**: Egy fájl vagy szöveg SHA-512 eljárással történő hash-elésére (ellenőrzőösszegének generálására) szolgál.

---

## 9. Rendszerinformációk és hálózatkezelés
A szerver alapvető állapotának és hálózati beállításainak lekérdezésére és módosítására vonatkozó parancsok.

* **`apt update`** és **`apt upgrade`**: Az operációs rendszer csomagjainak és magának a rendszernek a frissítésére szolgálnak egy lépésben.
* **`timedatectl`**: Az aktuális időzóna lekérdezésére, kilistázására és helyes értékre (pl. Budapest) történő beállítására alkalmazható.
* **`ip a`** (vagy **`ip addr`**): A jelenlegi IP címek és MAC azonosítók megjelenítésére szolgál.
* **`nano /etc/netplan/00-installer-config.yaml`** (vagy hasonló `.yaml` fájl): A hálózati csatoló (pl. az `enp0s8`) statikus IP, maszk, átjáró és DNS beállításainak elvégzésére használatos Ubuntu 20.04 alatt.
* **`uname -a`**: A kernel és az operációs rendszer adatainak kiíratására való.
* **`uptime`**: Megmutatja, hogy mióta van bekapcsolva a gép.
* **`htop`** és **`top`**: A memóriát vagy processzort leginkább terhelő, futó folyamatok kilistázására és azonosítására szolgáló programok.
* **`iptables`** (vagy **`ufw`**): NAT-olás beállítására a szerveren, hogy a kliens gép számára elérhetővé váljon a publikus hálózat.

---

## 10. Felhasználók és csoportok kezelése
A felhasználói fiókok és azok jogosultságainak menedzselése.

* **`su -`**: Lehetővé teszi, hogy átlépj egy másik felhasználó (pl. root) profiljába úgy, hogy a környezeti változók is az új felhasználóhoz igazodjanak (elfelejtve a régi maradványokat).
* **`useradd`**: Új felhasználó (pl. "venusz") létrehozására szolgál különböző paraméterek (home könyvtár, csoport, név) megadásával.
* **`usermod`**: Egy létező felhasználó fiókjának letiltására, engedélyezésére vagy új csoportba helyezésére használható.
* **`groupadd`**: Új csoport (pl. "naprendszer") létrehozására való.
* **`passwd`**: Egy felhasználó (pl. a root) jelszavának beállítására vagy módosítására szolgál.

---

## 11. Lemezezés, RAID és Szolgáltatások telepítése
Ezek a parancsok a szerver tárolókapacitásának és a nyújtott szolgáltatásoknak a menedzselését végzik.

* **`mdadm`**: RAID tömbök (pl. RAID-1) létrehozására, kezelésére, valamint meghibásodás utáni helyreállítására használatos.
* **`mount`**: A létrehozott RAID tömbök (vagy lemezek) megfelelő mappákba (pl. `/backup50`) történő becsatolására szolgál.
* **`nano /etc/fstab`**: Ennek a fájlnak a szerkesztésével érhető el, hogy a lemezek becsatolása újraindítás után is megmaradjon.
* **`apt install isc-dhcp-server`**: A DHCP szerver telepítésére szolgál, amelyet utána konfigurálni kell a megfelelő IP tartománnyal és átjáróval.
* **`apt install dnsmasq`**: A DNS szolgáltatás (névfeloldás) telepítésére és beállítására alkalmazandó program.
* **`apt install apache2 mysql-server php phpmyadmin`**: Ezekkel a parancsokkal telepíthető fel a LAMP (Linux, Apache, MySQL, PHP) szerver környezet és a phpMyAdmin felület.

```
