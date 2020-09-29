musl-cross-make
===============

мусл-кросс-мейк

Это второе поколение musl-cross-make, быстрого, простого, но продвинутого подхода на основе make-файлов для создания кросс-компиляторов, ориентированных на musl. Возможности включают:

Одноэтапная сборка GCC, используемая для сборки как musl libc, так и собственных общих целевых библиотек в зависимости от libc.

Нет жестко запрограммированных абсолютных путей; полученные кросс-компиляторы можно копировать / перемещать куда угодно.

Возможность создавать несколько кросс-компиляторов для разных целей, используя один набор исправленных исходных деревьев.

Ничего не устанавливается до тех пор make install, пока не запустится , и место установки можно выбрать во время установки.

Автоматическая загрузка исходных пакетов, включая необходимые компоненты GCC (GMP, MPC, MPFR), с использованием https и проверки хэшей.

Автоматическое внесение исправлений с использованием канонических исправлений поддержки musl и исправлений, которые обеспечивают исправления ошибок и функции, от которых зависит musl для некоторых целей Arch.

Применение
Систему сборки можно настроить, предоставив config.makфайл в каталоге верхнего уровня. Единственная обязательная переменная TARGET, которая должна содержать целевой кортеж gcc (например, i486-linux-musl), но доступно гораздо больше параметров. См. Предоставленные config.mak.distи presets/*примеры.

Для компиляции запустите make. Для установки $(OUTPUT)запустите make install.

Значение по умолчанию для $(OUTPUT)- выводится; после установки здесь вы можете переместить цепочку инструментов кросс-компилятора в другое место по желанию.

Поддерживается TARGETs
Ниже приводится неисчерпывающий список $(TARGET)кортежей, которые считаются работоспособными:

aarch64[_be]-linux-musl
arm[eb]-linux-musleabi[hf]
i*86-linux-musl
microblaze[el]-linux-musl
mips-linux-musl
mips[el]-linux-musl[sf]
mips64[el]-linux-musl[n32][sf]
powerpc-linux-musl[sf]
powerpc64[le]-linux-musl
riscv64-linux-musl
s390x-linux-musl
sh*[eb]-linux-musl[fdpic][sf]
x86_64-linux-musl[x32]
Как это устроено
Текущий мусульманский кросс-мейк делится на два слоя:

Makefile верхнего уровня, который отвечает за загрузку, проверку, извлечение и исправление источников, а также за настройку каталога сборки, а также

Litecross, система сборки кросс-компилятора, которая сама по себе является Makefile, привязанным к каталогу сборки.

Большая часть настоящего волшебства происходит в лайткроссе. Он начинается с установки символических ссылок на все исходные деревья, предоставленные ему вызывающей стороной, затем создает объединенный src_toolchainкаталог символических ссылок, который объединяет содержимое исходных деревьев gcc и binutils верхнего уровня и символические ссылки на gmp, mpc и mpfr. Один сконфигурированный вызов их настраивает все компоненты инструментальной цепочки GNU вместе таким образом, что не требуется их установка, чтобы другие могли их использовать.

Однако вместо того, чтобы строить все дерево инструментальной цепочки сразу, litecross начинает с создания только каталога gcc и его предварительных условий, чтобы получить файл, xgccкоторый можно использовать для настройки musl. Затем он настраивает musl, устанавливает заголовки musl в промежуточный «build sysroot» и выполняет сборку libgcc.aс использованием этих заголовков. На данный момент у него есть все предпосылки для сборки musl libc.aи libc.so, от которых зависят остальные целевые библиотеки gcc; как только они будут построены, вся цепочка инструментов make allможет продолжаться.

Litecross фактически не зависит от системы сборки верхнего уровня musl-cross-make; его можно использовать с любым предварительно извлеченным, правильно пропатченным набором исходных деревьев.

Объем и цели проекта
Основные цели этого проекта:

Предоставьте канонические патчи поддержки musl для GCC и binutils.

Служить каноническим примером того, как GCC должен быть построен для работы с musl.

Оптимизируйте производство кросс-компиляторов, ориентированных на musl, чтобы пользователи musl могли легко создавать приложения, связанные с musl, или загружать новые системы с помощью musl.

Помогите разработчикам musl и toolchain в разработке и тестировании.

Хотя все патчи, применяемые к GCC и binutils, предназначены для того, чтобы не нарушать конфигурации, отличные от musl, musl-cross-make сам по себе специфичен для musl. Изменения для добавления поддержки экзотических целевых вариантов, выходящие за рамки поддержки musl, вероятно, выходят за рамки, если они не являются неинвазивными. Изменения для исправления проблем при сборке musl-cross-make для работы в системах, отличных от Linux, вполне допустимы, если они чистые.

Самое главное, что это побочный проект на благо musl и его пользователей. Он не предназначен для того, чтобы требовать особого ухода или отвлекать усилия по разработке от самого musl.

Патчи включены
В дополнение к каноническим патчам поддержки musl для GCC, набор патчей musl-cross-make предоставляет:

Поддержка статической связи PIE
Добавление --enable-default-pie
Исправления ошибок, специфичных для SH, и битротов в GCC
Поддержка целевого процессора J2 Core в GCC и binutils
Поддержка SH / FDPIC ABI
Большинство этих патчей интегрированы в мастер gcc trunk / binutils. Их также можно использовать с оригинальным мусл-крестом Грегора или другими системами сборки, если это необходимо.

Некоторые функции (SH / FDPIC и поддержка специфических функций J2) в настоящее время доступны только в gcc 5.2.0 и новее и binutils 2.25.1 и новее.

Лицензия
Инструменты сборки и документация musl-cross-make под лицензией MIT / Expat, как указано в LICENSEфайле.

Обратите внимание, что эта лицензия не распространяется на патчи ( patches/) или полученные двоичные артефакты.

Каждый патч ( patches/) распространяется в соответствии с условиями лицензии вышестоящего проекта, к которому он применяется.

Точно так же любые результирующие двоичные артефакты, созданные с помощью этого инструментария сборки, сохраняют исходную лицензию от исходных проектов. Авторы musl-cross-make не предъявляют никаких дополнительных авторских прав на эти артефакты.

Вклад
Если вы явно не указали иное, любой вклад, представленный вами для включения в musl-cross-make, должен быть лицензирован, как указано выше, без каких-либо дополнительных условий.

