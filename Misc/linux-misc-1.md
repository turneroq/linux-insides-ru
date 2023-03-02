Linux kernel development
================================================================================

Вступление
--------------------------------------------------------------------------------
Как вы знаете, год назад я начал серию [публикаций](https://0xax.github.io/categories/assembler/) о программировании на языке ассемблера под `x86_64` архитектуру. Я не написал ни одной строчки низкоуровнегого кода до того момента, кроме пары обучающих примеров как `Hello World` в университете. Это было давно, как я уже говорил, я совсем не писал низкоуровнегого кода. Некоторое время назад, я стал интересоваться этими вещами. Я понимал, что могу писать программы, но совсем не понимал как они устроены.

После написания небольшого количества ассемблерного кода, я стал понимать, как моя программа **приблизительно** выглядит после компиляции. Но все равно, я не понимал много других вещей. Например, что происходит, когда `syscall` инструкция выполняется в моем ассемблерном коде, что происходит, когда `printf` функция начинает исполняться или как моя программа общается с другими компьютерами по сети. [Ассемблер](https://ru.wikipedia.org/wiki/%D0%AF%D0%B7%D1%8B%D0%BA_%D0%B0%D1%81%D1%81%D0%B5%D0%BC%D0%B1%D0%BB%D0%B5%D1%80%D0%B0) не дает ответов на мои вопросы и я решил копать глубже. Я начал изучать исходный код ядра Linux и пробовал понять вещи, которыми я интересовался. Код ядра Linux не дал мне ответы на **все** мои вопросы, но мое понимание ядра и процессов связанных с ним улучшилось. 

Я пишу эту часть спустя 9,5 месяцев как я начал изучать код ядра Linux и публикую [первую](https://proninyaroslav.gitbooks.io/linux-insides-ru/content/Booting/linux-bootstrap-1.html) часть этой книги. Теперь она содержит 40 частей и это еще не конец. Я решил писать эту серию публикаций по большей части для себя. Как вы знаете, ядро Linux - очень огромная часть кода и легко забыть что делает, означает или реализует та или иная часть ядра.  Но вскоре [linux-insides](https://github.com/0xAX/linux-insides) репозиторий стал популярный и после 9 месяцев имеет `9096` звезд:

![github](images/github.png)

Как оказалось люди заинтересованы во внутренностях ядра Linux. Помимо этого, все время что я пишу `linux-insides`, я получаю много вопросов от разных людей о том, как внести свой вклад в разработку ядра Linux. Как правило, люди заинтересованы в участии в проектах с открытым исходным кодом, и ядро Linux не является исключением.

![google-linux](images/google_linux.png)

Итак, как оказалось люди заинтересованы в процессе разработки ядра Linux. Я думал, это будет странно, если книга про ядро Linux не будет содержать описания, как принять участие в разработке ядра, и поэтому я решил написать это. В этой части вы не найдете информации о том, почему вы должны быть заинтересованы в участии в разработке ядра Linux. Но если вы интересуетесь как начать разработку ядра, то эта часть для вас.

Итак, приступим.


Как начать c ядра Linux
---------------------------------------------------------------------------------

Во-первых, давайте рассмотрим, как получить, собрать и запустить ядро. Вы можете запустить свою кастомную сборку ядра двумя способами:

* Запустить ядро на виртуальной машине.
* Запустить ядро на реальном железе.

Я приведу описания обоих способов. До того как мы начнем делать что-либо с ядром Linux, мы должны получить его. Есть пара способов сделать это, в зависимости от вашей цели. Если вы просто хотите обновить текущую версию ядра на вашей компьютере, вы можете использовать гайд для вашего [дистибутива](https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D1%81%D1%82%D1%80%D0%B8%D0%B1%D1%83%D1%82%D0%B8%D0%B2_Linux).

В первом случае вам надо скачать новую версию ядра через [менеджер пакетов](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F_%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%B0%D0%BC%D0%B8). Например, чтобы обновить версию до `4.1` для [Ubuntu (Vivid Vervet)](http://releases.ubuntu.com/15.04/), надо выполнить следующие команды:

```
$ sudo add-apt-repository ppa:kernel-ppa/ppa
$ sudo apt-get update
```

После выполните эту команду:

```
$ apt-cache showpkg linux-headers
```

и выберите версию ядра, которая вам нужна. В следующий команде замените `${version}` на версию, которую вы выбрали в предыдущей команде:

```
$ sudo apt-get install linux-headers-${version} linux-headers-${version}-generic linux-image-${version}-generic --fix-missing
```

и перезагрузите вашу систему. После перезагрузки вы увидите новое ядро в меню [grub](https://ru.wikipedia.org/wiki/GNU_GRUB).

В другом случае, если вы заинтересованы в разработке ядра Linux, вам надо получить исходный код ядра. Вы можете найти его на  [kernel.org](https://kernel.org/) сайте. На самом деле процесс разработки ядра Linux полностью построен вокруг `git` [системы контроля версий](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F_%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D1%8F%D0%BC%D0%B8). Поэтому вы можете получить код с `kernel.org` с помощью следующей `git` команды:

```
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

Я не знаю насчет вас, но я предпочитаю `github`. Здесь есть [Зеркало](https://github.com/torvalds/linux) основного репозитория ядра Linux, поэтому вы можете клонировать его с помощью:

```
$ git clone git@github.com:torvalds/linux.git
```

Я использую мой собственный [fork](https://github.com/0xAX/linux) для разработки и когда я хочу подтянуть обновления из основного репозитория, я выполняю следующую команду:

```
$ git checkout master
$ git pull upstream master
```
Заметьте, что имя главного репозитория для удаленного доступа - `upstream`. Чтобы добавить новый удаленный доступ к главному репозиторию Linux, вы можете выполнить:

```
git remote add upstream git@github.com:torvalds/linux.git
```
После этого у вас будет 2 удаленных доступа:

```
~/dev/linux (master) $ git remote -v
origin	git@github.com:0xAX/linux.git (fetch)
origin	git@github.com:0xAX/linux.git (push)
upstream	https://github.com/torvalds/linux.git (fetch)
upstream	https://github.com/torvalds/linux.git (push)
```
Один из них вашего fork (`origin`) и другой главного репозитория (`upstream`).

Сейчас у нас есть локальная копия исходного кода ядра, теперь мы должны настроить и собрать его. Ядро может быть сконфигурировано несколькими способами. Самый простой - просто скопировать конфигурационный файл из уже установленного ядра, который находиться в директории `/boot`.

```
$ sudo cp /boot/config-$(uname -r) ~/dev/linux/.config
```

Если ваше текущее ядро ​​Linux было собрано с помощью доступа к файлу `/proc/config.gz`, вы можете скопировать актуальный файл конфигурации ядра с помощью этой команды:

```
$ cat /proc/config.gz | gunzip > ~/dev/linux/.config
```
Если вы не довольны со стандартной конфигурацией ядра, которая предоставлена разработчиками и maintainers вашего дистрибутива, вы можете вручную настроить ядро. Есть пара способов сделать это. Корневой [Makefile](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/Makefile) предоставляет набор targets, который позволяет конфигурировать. Например, `menuconfig` предоставляем меню-интерфейс для конфигурации ядра:

![menuconfig](images/menuconfig.png)

`defconfig` аргумент генерирует стандартный файл конфигурации для текущей архитектуры, в нашем примере [x86_64 defconfig](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/arch/x86/configs/x86_64_defconfig). Вы можете передать аргумент командрой строки - `ARCH`, чтобы сделать `make` для `defconfig` для конкретной архитектуры:

```
$ make ARCH=arm64 defconfig
```

`allnoconfig`, `allyesconfig` и `allmodconfig` аргументы позволяют вам сгенерировать новый конфигурационный файо, где все опции будут выключены, включены или включены как модули, соотвественно.`nconfig` аргумент командной строки, который предоставляет программу основанную на `ncurses` с меню для конфигурации:

![nconfig](images/nconfig.png)

И даже `randconfig` для создания случайного файла конфигурации. Я не буду писать как конфигурировать ядро Linux или какие опции включить, потому что это не имеет смысла делать так по двум причинам: во-первых, я не знаю вашего железа и во-вторых, если я знаю вашего железо, то остается единственная задача, понять как использовать программы для конфигурации ядра, и все они очень просты в использовании.

Хорошо, сейчас у нас есть исходный код ядра и его конфигурация. Следующий шаг - компиляция ядра. Самый простой способ скомпилировать ядро, это просто выполнить следующее:

```
$ make
scripts/kconfig/conf  --silentoldconfig Kconfig
#
# configuration written to .config
#
  CHK     include/config/kernel.release
  UPD     include/config/kernel.release
  CHK     include/generated/uapi/linux/version.h
  CHK     include/generated/utsrelease.h
  ...
  ...
  ...
  OBJCOPY arch/x86/boot/vmlinux.bin
  AS      arch/x86/boot/header.o
  LD      arch/x86/boot/setup.elf
  OBJCOPY arch/x86/boot/setup.bin
  BUILD   arch/x86/boot/bzImage
  Setup is 15740 bytes (padded to 15872 bytes).
System is 4342 kB
CRC 82703414
Kernel: arch/x86/boot/bzImage is ready  (#73)
```
Для ускорения скорости компиляции вы можете передать `-jN` аргумент командой строки в `make`, где `N` определяет число команд запущенных одновременно.

```
$ make -j8
```
Если вы хотите собрать ядро для архитектуры, которая отличается от вашей, самый простой способ - передать два аргумента:

* `ARCH` аргумент и архитектуру под которую собираете;
* `CROSS_COMPILER` аргумент и префикс tool кросс компилятора;

К примеру, если вы хотите скомпилировать ядро для [arm64](https://ru.wikipedia.org/wiki/ARM_(%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0)) со стандартным файлом конфигурации ядра, надо выполнить:

```
$ make -j4 ARCH=arm64 CROSS_COMPILER=aarch64-linux-gnu- defconfig
$ make -j4 ARCH=arm64 CROSS_COMPILER=aarch64-linux-gnu-
```

В результате компиляции мы можем увидеть сжатый образ ядра -`arch/x86/boot/bzImage`. Теперь, когда мы скомпилировали ядро, мы можем либо установить его на свой компьютер, либо просто запустить в эмуляторе.

Установка ядра Linux
--------------------------------------------------------------------------------

Как я уже писал, мы рассмотрим два способа как запустить новое ядро: в первом случае, мы может установить и запустить новую версию ядра на реальном железе и во втором, запустить на виртуальной машине. В предыдущем параграфе, мы видели как собирать ядро Linux из исходного кода и как результат мы можем получить сжатый образ ядра:

```
...
...
...
Kernel: arch/x86/boot/bzImage is ready  (#73)
```

После того, как мы получили образ [bzImage](https://en.wikipedia.org/wiki/Vmlinux#bzImage) нам надо установить `Заголовки`, `Модули` с помощью:

```
$ sudo make headers_install
$ sudo make modules_install
```

и само ядро:

```
$ sudo make install
```

С этого момента мы установили новую версию ядра Linux и сейчас мы должны сообщить `загрузчику` об этом. Конечно мы можем добавить это вручную, путем редактирования `/boot/grub2/grub.cfg` конфигурационного файла, но я предпочитаю использовать скрипт для этой цели. Я использую два разных дистрибутива Linux: Fedora и Ubuntu. Существует два различных способа обновить конфигурационный файл [grub](https://ru.wikipedia.org/wiki/GNU_GRUB). Я использую следующий скрипт для этого:

```shell
#!/bin/bash

source "term-colors"

DISTRIBUTIVE=$(cat /etc/*-release | grep NAME | head -1 | sed -n -e 's/NAME\=//p')
echo -e "Distributive: ${Green}${DISTRIBUTIVE}${Color_Off}"

if [[ "$DISTRIBUTIVE" == "Fedora" ]] ;
then
    su -c 'grub2-mkconfig -o /boot/grub2/grub.cfg'
else
    sudo update-grub
fi

echo "${Green}Done.${Color_Off}"
```

Это последний шаг установки ядра Linux и после этого вы можете перезагрузить вам компьютр и выбрать новую версию ядра при загрузке.

Второй случай - запуск ядра на виртуальной машине. Я предпочитаю [qemu](https://ru.wikipedia.org/wiki/QEMU). В первую очередь, нам надо собрать начальный ramdisk - [initrd](https://ru.wikipedia.org/wiki/Initrd) для этого. `initrd` - временная корневая файловая система, которая используется ядром Linux в процессе инициализации пока другие файловые системы не монтированы. Мы можем собрать `initrd` с помощью следующей команды:

Во-первых нам надо скачать [busybox](https://ru.wikipedia.org/wiki/BusyBox) и запустить `menuconfig` для его конфигурации:

```shell
$ mkdir initrd
$ cd initrd
$ curl http://busybox.net/downloads/busybox-1.23.2.tar.bz2 | tar xjf -
$ cd busybox-1.23.2/
$ make menuconfig
$ make -j4
```

`busybox` исполняемый файл - `/bin/busybox`, который содержит набор стандартных иструментов, такие как [coreutils]()

`busybox` is an executable file - `/bin/busybox` that contains a set of standard tools like [coreutils](https://en.wikipedia.org/wiki/GNU_Core_Utilities). In the `busysbox` menu we need to enable: `Build BusyBox as a static binary (no shared libs)` option:

![busysbox menu](http://i68.tinypic.com/11933bp.png)

We can find this menu in the:

```
Busybox Settings
--> Build Options
```

After this we exit from the `busysbox` configuration menu and execute following commands for building and installation of it:

```
$ make -j4
$ sudo make install
```

Now that `busybox` is installed, we can begin building our `initrd`. To do this, we go to the previous `initrd` directory and:

```
$ cd ..
$ mkdir -p initramfs
$ cd initramfs
$ mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
$ cp -av ../busybox-1.23.2/_install/* .
```

copy `busybox` fields to the `bin`, `sbin` and other directories. Now we need to create executable `init` file that will be executed as a first process in the system. My `init` file just mounts [procfs](https://en.wikipedia.org/wiki/Procfs) and [sysfs](https://en.wikipedia.org/wiki/Sysfs) filesystems and executed shell:

```shell
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys

exec /bin/sh
```

Now we can create an archive that will be our `initrd`:

```
$ find . -print0 | cpio --null -ov --format=newc | gzip -9 > ~/dev/initrd_x86_64.gz
```

We can now run our kernel in the virtual machine. As I already wrote I prefer [qemu](https://en.wikipedia.org/wiki/QEMU) for this. We can run our kernel with the following command:

```
$ qemu-system-x86_64 -snapshot -m 8GB -serial stdio -kernel ~/dev/linux/arch/x86_64/boot/bzImage -initrd ~/dev/initrd_x86_64.gz -append "root=/dev/sda1 ignore_loglevel"
```

![qemu](images/qemu.png)

From now we can run the Linux kernel in the virtual machine and this means that we can begin to change and test the kernel.

Consider using [ivandaviov/minimal](https://github.com/ivandavidov/minimal) or [Buildroot](https://buildroot.org/) to automate the process of generating initrd.

Getting started with the Linux Kernel Development
---------------------------------------------------------------------------------

The main point of this paragraph is to answer two questions: What to do and what not to do before sending your first patch to the Linux kernel. Please, do not confuse this `to do` with `todo`. I have no answer what you can fix in the Linux kernel. I just want to tell you my workflow during experimenting with the Linux kernel source code.

First of all I pull the latest updates from Linus's repo with the following commands:

```
$ git checkout master
$ git pull upstream master
```

After this my local repository with the Linux kernel source code is synced with the [mainline](https://github.com/torvalds/linux) repository. Now we can make some changes in the source code. As I already wrote, I have no advice for you where you can start and what `TODO` in the Linux kernel. But the best place for newbies is `staging` tree. In other words the set of drivers from the [drivers/staging](https://github.com/torvalds/linux/tree/master/drivers/staging). The maintainer of the `staging` tree is [Greg Kroah-Hartman](https://en.wikipedia.org/wiki/Greg_Kroah-Hartman) and the `staging` tree is that place where your trivial patch can be accepted. Let's look on a simple example that describes how to generate patch, check it and send to the [Linux kernel mail listing](https://lkml.org/).

If we look in the driver for the [Digi International EPCA PCI](https://github.com/torvalds/linux/tree/master/drivers/staging/dgap) based devices, we will see the `dgap_sindex` function on line 295:

```C
static char *dgap_sindex(char *string, char *group)
{
	char *ptr;

	if (!string || !group)
		return NULL;

	for (; *string; string++) {
		for (ptr = group; *ptr; ptr++) {
			if (*ptr == *string)
				return string;
		}
	}

	return NULL;
}
```

This function looks for a match of any character in the group and returns that position. During research of source code of the Linux kernel, I have noted that the [lib/string.c](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/lib/string.c#L473) source code file contains the implementation of the `strpbrk` function that does the same thing as `dgap_sinidex`. It is not a good idea to use a custom implementation of a function that already exists, so we can remove the `dgap_sindex` function from the [drivers/staging/dgap/dgap.c](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/drivers/staging/dgap/dgap.c) source code file and use the `strpbrk` instead.

First of all let's create new `git` branch based on the current master that synced with the Linux kernel mainline repo:

```
$ git checkout -b "dgap-remove-dgap_sindex"
```

And now we can replace the `dgap_sindex` with the `strpbrk`. After we did all changes we need to recompile the Linux kernel or just [dgap](https://github.com/torvalds/linux/tree/master/drivers/staging/dgap) directory. Do not forget to enable this driver in the kernel configuration. You can find it in the:

```
Device Drivers
--> Staging drivers
----> Digi EPCA PCI products
```

![dgap menu](images/dgap_menu.png)

Now is time to make commit. I'm using following combination for this:

```
$ git add .
$ git commit -s -v
```

After the last command an editor will be opened that will be chosen from `$GIT_EDITOR` or `$EDITOR` environment variable. The `-s` command line argument will add `Signed-off-by` line by the committer at the end of the commit log message. You can find this line in the end of each commit message, for example - [00cc1633](https://github.com/torvalds/linux/commit/00cc1633816de8c95f337608a1ea64e228faf771). The main point of this line is the tracking of who did a change. The `-v` option show unified diff between the HEAD commit and what would be committed at the bottom of the commit message. It is not necessary, but very useful sometimes. A couple of words about commit message. Actually a commit message consists from two parts:

The first part is on the first line and contains short description of changes. It starts from the `[PATCH]` prefix followed by a subsystem, driver or architecture name and after `:` symbol short description. In our case it will be something like this:

```
[PATCH] staging/dgap: Use strpbrk() instead of dgap_sindex()
```

After short description usually we have an empty line and full description of the commit. In our case it will be:

```
The <linux/string.h> provides strpbrk() function that does the same that the
dgap_sindex(). Let's use already defined function instead of writing custom.
```

And the `Sign-off-by` line in the end of the commit message. Note that each line of a commit message must no be longer than `80` symbols and commit message must describe your changes in details. Do not just write a commit message like: `Custom function removed`, you need to describe what you did and why. The patch reviewers must know what they review. Besides this commit messages in this view are very helpful. Each time when we can't understand something, we can use [git blame](http://git-scm.com/docs/git-blame) to read description of changes.

After we have committed changes time to generate patch. We can do it with the `format-patch` command:

```
$ git format-patch master
0001-staging-dgap-Use-strpbrk-instead-of-dgap_sindex.patch
```

We've passed name of the branch (`master` in this case) to the `format-patch` command that will generate a patch with the last changes that are in the `dgap-remove-dgap_sindex` branch and not are in the `master` branch. As you can note, the `format-patch` command generates file that contains last changes and has name that is based on the commit short description. If you want to generate a patch with the custom name, you can use `--stdout` option:

```
$ git format-patch master --stdout > dgap-patch-1.patch
```

The last step after we have generated our patch is to send it to the Linux kernel mailing list. Of course, you can use any email client, `git` provides a special command for this: `git send-email`. Before you send your patch, you need to know where to send it. Yes, you can just send it to the Linux kernel mailing list address which is `linux-kernel@vger.kernel.org`, but it is very likely that the patch will be ignored, because of the large flow of messages. The better choice would be to send the patch to the maintainers of the subsystem where you have made changes. To find the names of these maintainers use the `get_maintainer.pl` script. All you need to do is pass the file or directory where you wrote code.

```
$ ./scripts/get_maintainer.pl -f drivers/staging/dgap/dgap.c
Lidza Louina <lidza.louina@gmail.com> (maintainer:DIGI EPCA PCI PRODUCTS)
Mark Hounschell <markh@compro.net> (maintainer:DIGI EPCA PCI PRODUCTS)
Daeseok Youn <daeseok.youn@gmail.com> (maintainer:DIGI EPCA PCI PRODUCTS)
Greg Kroah-Hartman <gregkh@linuxfoundation.org> (supporter:STAGING SUBSYSTEM)
driverdev-devel@linuxdriverproject.org (open list:DIGI EPCA PCI PRODUCTS)
devel@driverdev.osuosl.org (open list:STAGING SUBSYSTEM)
linux-kernel@vger.kernel.org (open list)
```

You will see the set of the names and related emails. Now we can send our patch with:

```
$ git send-email --to "Lidza Louina <lidza.louina@gmail.com>" \
  --cc "Mark Hounschell <markh@compro.net>"                   \
  --cc "Daeseok Youn <daeseok.youn@gmail.com>"                \
  --cc "Greg Kroah-Hartman <gregkh@linuxfoundation.org>"      \
  --cc "driverdev-devel@linuxdriverproject.org"               \
  --cc "devel@driverdev.osuosl.org"                           \
  --cc "linux-kernel@vger.kernel.org"
```

That's all. The patch is sent and now you only have to wait for feedback from the Linux kernel developers. After you send a patch and a maintainer accepts it, you will find it in the maintainer's repository (for example [patch](https://git.kernel.org/cgit/linux/kernel/git/gregkh/staging.git/commit/?h=staging-testing&id=b9f7f1d0846f15585b8af64435b6b706b25a5c0b) that you saw in this part) and after some time the maintainer will send a pull request to Linus and you will see your patch in the mainline repository.

That's all.

Some advice
--------------------------------------------------------------------------------

In the end of this part I want to give you some advice that will describe what to do and what not to do during development of the Linux kernel:

* Think, Think, Think. And think again before you decide to send a patch.

* Each time when you have changed something in the Linux kernel source code - compile it. After any changes. Again and again. Nobody likes changes that don't even compile.

* The Linux kernel has a coding style [guide](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/Documentation/CodingStyle) and you need to comply with it. There is great script which can help to check your changes. This script is - [scripts/checkpatch.pl](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/scripts/checkpatch.pl). Just pass source code file with changes to it and you will see:

```
$ ./scripts/checkpatch.pl -f drivers/staging/dgap/dgap.c
WARNING: Block comments use * on subsequent lines
#94: FILE: drivers/staging/dgap/dgap.c:94:
+/*
+     SUPPORTED PRODUCTS

CHECK: spaces preferred around that '|' (ctx:VxV)
#143: FILE: drivers/staging/dgap/dgap.c:143:
+	{ PPCM,        PCI_DEV_XEM_NAME,     64, (T_PCXM|T_PCLITE|T_PCIBUS) },

```

Also you can see problematic places with the help of the `git diff`:

![git diff](images/git_diff.png)

* [Linus doesn't accept github pull requests](https://github.com/torvalds/linux/pull/17#issuecomment-5654674)

* If your change consists from some different and unrelated changes, you need to split the changes via separate commits. The `git format-patch` command will generate patches for each commit and the subject of each patch will contain a `vN` prefix where the `N` is the number of the patch. If you are planning to send a series of patches it will be helpful to pass the `--cover-letter` option to the `git format-patch` command. This will generate an additional file that will contain the cover letter that you can use to describe what your patchset changes. It is also a good idea to use the `--in-reply-to` option in the `git send-email` command. This option allows you to send your patch series in reply to your cover message. The structure of the your patch will look like this for a maintainer:

```
|--> cover letter
  |----> patch_1
  |----> patch_2
```

You need to pass `message-id` as an argument of the `--in-reply-to` option that you can find in the output of the `git send-email`:

It's important that your email be in the [plain text](https://en.wikipedia.org/wiki/Plain_text) format. Generally, `send-email` and `format-patch` are very useful during development, so look at the documentation for the commands and you'll find some useful options such as: [git send-email](http://git-scm.com/docs/git-send-email) and [git format-patch](http://git-scm.com/docs/git-format-patch).

* Do not be surprised if you do not get an immediate answer after you send your patch. Maintainers can be very busy.

* The [scripts](https://github.com/torvalds/linux/tree/master/scripts) directory contains many different useful scripts that are related to Linux kernel development. We already saw two scripts from this directory: the `checkpatch.pl` and the `get_maintainer.pl` scripts. Outside of those scripts, you can find the [stackusage](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/scripts/stackusage) script that will print usage of the stack, [extract-vmlinux](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/scripts/extract-vmlinux) for extracting an uncompressed kernel image, and many others. Outside of the `scripts` directory you can find some very useful [scripts](https://github.com/lorenzo-stoakes/kernel-scripts) by [Lorenzo Stoakes](https://twitter.com/ljsloz) for kernel development.

* Subscribe to the Linux kernel mailing list. There are a large number of letters every day on `lkml`, but it is very useful to read them and understand things such as the current state of the Linux kernel. Other than `lkml` there are [set](http://vger.kernel.org/vger-lists.html) mailing listings which are related to the different Linux kernel subsystems.

* If your patch is not accepted the first time and you receive feedback from Linux kernel developers, make your changes and resend the patch with the `[PATCH vN]` prefix (where `N` is the number of patch version). For example:

```
[PATCH v2] staging/dgap: Use strpbrk() instead of dgap_sindex()
```

Also it must contain a changelog that describes all changes from previous patch versions. Of course, this is not an exhaustive list of requirements for Linux kernel development, but some of the most important items were addressed.

Happy Hacking!

Conclusion
--------------------------------------------------------------------------------

I hope this will help others join the Linux kernel community!
If you have any questions or suggestions, write me at [email](kuleshovmail@gmail.com) or ping [me](https://twitter.com/0xAX) on twitter.

Please note that English is not my first language, and I am really sorry for any inconvenience. If you find any mistakes please let me know via email or send a PR.

Links
--------------------------------------------------------------------------------

* [blog posts about assembly programming for x86_64](https://0xax.github.io/categories/assembler/)
* [Assembler](https://en.wikipedia.org/wiki/Assembly_language#Assembler)
* [distro](https://en.wikipedia.org/wiki/Linux_distribution)
* [package manager](https://en.wikipedia.org/wiki/Package_manager)
* [grub](https://en.wikipedia.org/wiki/GNU_GRUB)
* [kernel.org](https://kernel.org/)
* [version control system](https://en.wikipedia.org/wiki/Version_control)
* [arm64](https://en.wikipedia.org/wiki/ARM_architecture#AArch64_features)
* [bzImage](https://en.wikipedia.org/wiki/Vmlinux#bzImage)
* [qemu](https://en.wikipedia.org/wiki/QEMU)
* [initrd](https://en.wikipedia.org/wiki/Initrd)
* [busybox](https://en.wikipedia.org/wiki/BusyBox)
* [coreutils](https://en.wikipedia.org/wiki/GNU_Core_Utilities)
* [procfs](https://en.wikipedia.org/wiki/Procfs)
* [sysfs](https://en.wikipedia.org/wiki/Sysfs)
* [Linux kernel mail listing archive](https://lkml.org/)
* [Linux kernel coding style guide](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/Documentation/CodingStyle)
* [How to Get Your Change Into the Linux Kernel](https://github.com/torvalds/linux/blob/16f73eb02d7e1765ccab3d2018e0bd98eb93d973/Documentation/SubmittingPatches)
* [Linux Kernel Newbies](http://kernelnewbies.org/)
* [plain text](https://en.wikipedia.org/wiki/Plain_text)
