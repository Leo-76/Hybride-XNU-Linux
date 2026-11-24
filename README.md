# Noyau hybride XNU / Linux :

Quatre essais autour de PureDarwin (Darwin 20.5, XNU x86_64) et de Linux. **Tout a été vu sous QEMU seulement. Rien n'est testé sur un PC réel.** Chaque dossier de `captures/` correspond à un essai.

## 1. Programmes Darwin (Mach-O) sur un noyau Linux
`captures/darwin-sur-linux/` : captures réelles de terminal (fenêtres du Mac, avec leur titre).

Le chargeur `mldr` lance des binaires PureDarwin sur un vrai noyau Linux (`uname -sm` : `Linux x86_64`).
- `01` : `bash` 5.3 de PureDarwin en terminal interactif : tubes, tâche de fond, `jobs`, `kill %1`.
- `02` : fils d'exécution et `kqueue`.
- `03` : connexion `ssh` dont le shell de connexion est le `bash` Darwin ; il boucle, puis `clang` de Darwin compile un programme et le lance (`hello depuis clang Darwin`, `rc=0`).
- `04` : le même `bash` sans SSH, lancé au démarrage avec le journal série ; `exec` d'un autre Mach-O, sous-shell, puis `clang` et le binaire produit (`rc=0`).

Les images de cette section remplacent des captures précédentes qui étaient des rendus, pas de vrais écrans.

## 2. Programmes UNIX V7/x86 sur le noyau hybride
`captures/v7-sur-xnu/`

Des binaires `a.out` de V7 (compilés dans V7) tournent sous PureDarwin grâce à un noyau patché et un petit chargeur. Capture : `hello` affiche `hi`, `psh -c 'echo ok $((6*7))'` affiche `ok 42`. Selon le rapport, `pipe`, `exec` et `stat` donnent aussi les bonnes sorties.

## 3. Debian sur le noyau hybride
`captures/debian-sur-xnu/`

Programmes Linux (ELF) de Debian 13 sous XNU : login `root`, `ls`, `uname`, `ps`.
- 112 commandes sur 116 passent dans `bash` (capture 20). Restent en échec : `which`, `python3` (non installé), `tty`, `coproc/fifo`.
- `systemd` 257 démarre jusqu'à « Welcome to Debian GNU/Linux 13 » (capture 16), puis se bloque. **Pas terminé.**
- L'ISO « GNU/XNU » démarre jusqu'à `login:` (capture 21).

## 4. Une image qui regroupe PureDarwin, V7 et Debian
`captures/iso/`

Une image de 946 Mo avec un menu GRUB. En UEFI elle démarre **comme un disque ou une clé USB**, pas comme un CD : en `-cdrom`, XNU ne trouve pas sa racine.

## 5. Shells (bash Darwin, bash Linux, psh) sur le noyau hybride
`captures/shells-sur-xnu/`

Même script de session (`session-shell.sh`), tapé à la main au prompt `#` de la console, sur un noyau propre (sans compteurs de diagnostic). QEMU seulement.
- `01` : `bash` 5.3.9 **Darwin** (Mach-O). `uname`, version, tableau, arithmétique (`42 2`), tube, `jobs` (`[1]+ Running sleep 1 &`), `rc=3`, `cd /tmp`, `ulimit -n` (`256`), `trap` (`sig`).
- `02` : `bash` **Linux** (ELF, le même script). Mêmes sorties, sauf `ulimit -n` qui donne `1024` au lieu de `256`. Cause non expliquée.
- `03` : `psh` (shell POSIX maison), binaire Darwin puis binaire Linux, même petit script : `42`, `lo 5`, `data`, `rc=3`, `/tmp`, `sig`, `[a b]`, `123`.

Ce que ça ne montre pas : le script n'est pas la suite de tests complète de `psh` (`make test` n'a pas été rejoué sur ce noyau) ; `cat: /etc/hostname: No such file or directory` apparaît dans la capture 01 (fichier absent de l'image, sans lien avec le shell) ; un seul passage par shell.

## Limites connues
- Pas de test sur matériel réel : clavier, pilotes USB et démarrage sur clé restent inconnus.
- Les captures sont des écrans de terminal QEMU ; les chiffres viennent des rapports des essais et ne sont pas tous rejoués ici.
- Ce dépôt ne contient ni binaires ni images : seulement ce texte et des captures.
- PureDarwin, XNU et Debian gardent leurs propres licences.
