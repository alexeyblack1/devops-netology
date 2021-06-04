# devops-netology
ДЗ - 6
1.Ресурсы по умолчанию:
ОЗУ - 1024 Мб, видеопамять - 4 Мб, объем диска - 64 Гб
Добавлена память до 4096 с помощью настроек Oracle VirtualBox Manager. 

2.Какой переменной можно задать длину журнала history:
строка 2656 The value of the HISTSIZE variable is used as the number of commands to save in a history list.

3. директива ignoreboth в bash:
Значение ignoreboth является сокращением для ignorespace и ignoredups. Значение 
 заставляет все предыдущие строки, соответствующие текущей строке, удаляться из списка истории до того, как эта строка будет сохранена. Любое значение, не входящее в 
приведенный выше список игнорируется. Если HISTCONTROL не установлен или не содержит допустимого значения, все строки, прочитанные синтаксическим анализатором оболочки, сохраняются.
в списке истории при условии значения HISTIGNORE. Вторая и последующие строки многострочной составной команды не 
протестированы и добавляются в историю независимо от значения HISTCONTROL.


4. В каких сценариях использования применимы скобки {} и на какой строчке man bash это описано?:
строка 179 Bash также интерпретирует ряд многосимвольных параметров. Эти параметры должны появиться в командной строке перед односимвольным оператором чтобы быть распознанными.
5. Основываясь на предыдущем вопросе, как создать однократным вызовом touch 100000 файлов? А получилось ли создать 300000?

touch text/file-{000..100000}
Удалось создать max 104736 файлов
Создать 300000 не получилось, ошибка :'Argument list too long'

6.Что делает конструкция [[ -d /tmp ]]?
проверяет существование директории /tmp, когда директория существует, возвращает 1. Это расширенный вариант [] в части использования && и ||, внутри [] приведет к ошибке
7. Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке

         PATH=/tmp/new_bash:$PATH
         echo $PATH
         type -a bash
         
      
         bash is /tmp/new_bash/bash
         bash is /usr/bin/bash
         bash is /bin/bash
         

8. Чем отличается планирование команд с помощью batch и at?
команда at используется для назначения одноразового задания на заданное время, а команда batch — для назначения одноразовых задач, которые должны выполняться, когда загрузка системы становится меньше указанного
batch выполняет команды, когда это позволяют уровни загрузки системы; другими словами, когда средняя нагрузка падает ниже 1.5
