# git tips and tricks
Кук-бук рецептов гит
## Как перенести коммиты из одной ветки в другую?
Предположим, что вы с энтузиазмом работаете над новой задачей, но недопитый кофе и рабочий чат отвлекли, новая ветка не была создана и вообще изменения уже закоммичены в master. Неплохо бы перенести их в новую ветку и продолжить работу там. Один из возможных алгоритмов такой:
```
  git branch new-feature
  git reset HEAD~ --hard
  git checkout new-feature
```
В случае если вы уже сделали больше одного коммита, то нужно во второй команде указать количество коммитов после тильды, которые нужно перенести (цифра после тильды указывает на сколько коммитов **передвинется назад** по истории указатель `HEAD`). Например, если нужно перенести три последних коммита, то команда будет выглядеть так `$ git reset HEAD~3 --hard`
## Как склеить коммиты?
Работаете над задачей и делаете много коммитов? Не хотите тащить всю эту историю с собой в основную ветку, а сделать один коммит? Любите линейную историю? Тогда мы идем к вам!
<br>Внимание все способы склейки коммитов ***переписывают историю***. 
- Правило №1: Не используйте их, если вы уже отправили коммиты в удаленный репозиторий и хотите остаться целым и симметричным. Ваша банда-команда может на понять.
- Правило №2: Используйте их, когда изменения еще только в вашем локальном репозитории.
- Правило №3: Если вы работаете с удаленной веткой один, милости просим.

### Способ №1. rebase squash
Первый способ это использование `rebase` в ~~интуитивном~~ интерактивном режиме. Для этого считаем количество коммитов который мы хотим склеить (важно, чтобы они шли друг за другом), далее вводим `$ git rebase -i HEAD~<количество коммитов>`. И в интерактивном режиме заменяем `pick` на `squash` (или `fixup`) для каждого коммита 
### Способ №2. merge squash
```
$ git checkout master
$ git merge --squash branch_with_feature
$ gir commit -m 'best commit message'
```
### Способ №3. soft reset
Тоже самое можно сделать и с помощью reset с опцией `--soft`
```
$ git reset HEAD~3 --soft
$ git commit
```
### Способ №4. amend
Допустим вы сделали один коммит, внесли небольшое изменение и хотите склеить его с предыдущим коммитом *не меняя его сообщение*, вместо того чтобы сделать новый. Используйте: ```$ git commit --ammend --no-edit```.
## Как поменять сообщение в последнем коммите?
И вот на стал час коммита, вы набираете `$ git commit -m 'Wingarduim Leviosa'`, и вы понимаете, что описание коммита получилось очень абракадабристым.
Используйте: ```$ git commit --ammend```
## Как отменить git commit --amend?
Amend это круто, но вдруг нужно откатится назад и отменить amend? Просто сделать `reset` не получится, потому что коммит уже склеен с предыдущим. Тут на сцену выходит `git reflog`. Мощная команда для путешественников во времени. [Ссылка](http://git-scm.com/docs/git-reflog) на официальную документацию.
Приведем небольшой пример. Инициализируем новый репозиторий и сделаем в нем один коммит:
```
$ git init
Initialized empty Git repository in reflog/.git/
$ echo 'awesome changes' > awesome_file.txt
$ git add .
$ git commit -m 'awesome message'
[master (root-commit) b8e5fdd] awesome message
 1 file changed, 1 insertion(+)
 create mode 100644 awesome_file.txt
```
Прекрасно. Теперь добавим еще один коммит и склеим его с предыдущим:

```
$ echo 'great changes' > awesome_file.txt
$ git add .
$ git commit --amend
[master 8ee9546] awesome message
 Date: Fri Jan 7 22:57:31 2022 +0300
 1 file changed, 1 insertion(+)
 create mode 100644 awesome_file.txt
 ```

Текс, посмотрим, что там с историей коммитов, а заодно и глянем в `reflog`
```
$ git log --oneline
8ee9546 awesome message

$ git reflog
8ee9546 HEAD@{0}: commit (amend): awesome message
b8e5fdd HEAD@{1}: commit (initial): awesome message
```

Супер, с одной стороны оба коммита **склеились в один** коммит, с другой стороны мы имеем **две записи** в `reflog`. Теперь нам нужно просто скопировать хэш того коммита до которого мы хотим отменить изменения (в нашем случае хэш будет равен `b8e5fdd`), сделать reset и закоммитить изменения:

```
$ git reset --soft b8e5fdd
$ git commit -m 'undo commit'
[master 63c3d40] undo commit
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git log --oneline
63c3d40 (HEAD -> master) undo commit
b8e5fdd awesome message
```
Пабам, мы расклеили склеенный коммит
## Как отменить hard reset?
Все шло прекрасно, пока я не сделал `git reset --hard`... Возьмем пример отсюда [тыц](#как-отменить-git-commit---amend) и выполним команду `git reset b8e5fdd --hard` :

```
$ git log --oneline
63c3d40 (HEAD -> master) undo commit
b8e5fdd awesome message

$ git reset b8e5fdd --hard
HEAD is now at b8e5fdd awesome message

$ git log --oneline
b8e5fdd (HEAD -> master) awesome message
```
Таким образом, мы отменили последний коммит. И передумали... ШтошТеперьДелатьТо? А поступить можно так.  Тут нам тоже может помочь `git reflog` . Смотрим, что там есть:

```
$ git reflog
b8e5fdd (HEAD -> master) HEAD@{0}: reset: moving to b8e5fdd
63c3d40 HEAD@{1}: commit: undo commit
b8e5fdd (HEAD -> master) HEAD@{2}: reset: moving to b8e5fdd
8ee9546 HEAD@{3}: commit (amend): awesome message
b8e5fdd (HEAD -> master) HEAD@{4}: commit (initial): awesome message
```
В нашем случае нужно вернуться на один шаг назад. Лучший кандидат для этого - коммит с хэшем `63c3d40` :

```
$ git reset --hard 63c3d40
HEAD is now at 63c3d40 undo commit

$ git log --oneline
63c3d40 (HEAD -> master) undo commit
b8e5fdd awesome message
```

Мама, я только что, отменил `git reset --hard` !
## Как пользоваться .gitignore
Перевод официальной спецификации [тыц](http://git-scm.com/docs/gitignore).
<br>Шаблоны файла gitignore

| Шаблон | Поведение | Пример |
|--------|----------|--------|
|        | Пустые строки игнорируются||
|`#`     | Комментарий | `# this is comment` |
|`name`  | Будут проигнорированы все файлы и папки с заданным именем, а также файлы и подпапки в папке с именем `name` |`/name.log`<br/>`/name/file.txt`<br/>`/lib/name.log`|
|`name/` | Если шаблон оканчивается на `/`, то будут проигнорированы все файлы и подпапки в папке с именем `name` |`/name/file.txt`<br/>`/name/log/name.log`<br/><br/>**не будет проигнорирован:**<br/>`/name.log`|
|`name.file`| Будет проигнорирован файл с именем `name.file` |`/name.file`<br/>`/lib/name.file`|
|`/name.file`| Будет проигнорирован файл с именем `name.file`, находящийся в корневом каталоге | `/name.file`<br/><br/>**не будет проигнорирован:**<br/>`/lib/name.file`|
|`lib/name.file`| Будет проигнирован файл с именем `name.file` в папке `lib` корневого каталога (путь всегда соответствует корневому, даже если в начале не символа `/`) | `/lib/name.file`<br/><br/>**не будет проигнорирован:**<br/>`name.file`<br/>`/test/lib/name.file` |
|`**/lib/name.file` | Если шаблон начинается на `**` то, будет проигнорирова файл с именем `name.file` в любой папке `lib` репозитория | `/lib/name.file`<br/>`/test/lib/name.file` |
|`**/name`|Будут проигнорированы все заданные папки и файлы, а также подпапки внутри папки с заданным именем|`/name/log.file`<br/>`/lib/name/log.file`<br/>`/name/lib/log.file`|
|`/lib/**/name`|Будут проигнорированы все папки и файлы, а также подпапки в папке с именем `name`, находящейся внутри папки с именем `lib`|`/lib/name/log.file`<br/>`/lib/test/name/log.file`<br/>`/lib/test/ver1/name/log.file`<br/><br/>**не будет проигнорирован:**<br/>`/name/log.file`|
|`*.file`| Будут проигнорированы все файлы с расширением `.file`|	`/name.file`<br/>`/lib/name.file`|
|`*name/`| Будут проигнорированы все папки оканчивающиеся на `name`|`/lastname/log.file`<br/>`/firstname/log.file`|
|`name?.file`|`?` указывает на единственный не зарезервированный символ|`/names.file`<br/>`/name1.file`<br/><br/>**не будет проигнорирован:**<br/>`/names1.file`
|`name[a-z].file`| Единственный символ должен находится в указанном диапазоне символов|`/names.file`<br/>`/nameb.file`<br/><br/>**не будет проигнорирован:**<br/>`/name1.file`|
|`name[abc].file`| Единственный символ должен находится в указанном множестве символов|`/namea.file`<br/>`/nameb.file`<br/><br/>**не будет проигнорирован:**<br/>`/names.file`|
|`name[!abc].file`| Единственный символ **не должен** находится в указанном множестве символов|`/names.file`<br/>`/namex.file`<br/><br/>**не будет проигнорирован:**<br/>`/namesb.file`
|`name/`<br/>`!name/secret.log`|`!` инвертирует шаблон. Будут проигнорированы все файлы и подпапки в папке с именем `name`, исключая `name/secret.log`|`/name/file.txt`<br/>`/name/log/name.log`<br/><br/>**не будет проигнорирован:**<br/>`/name/secret.log`
|`*.file`<br/>`!name.file`|`!` инвертирует шаблон. Будут проигнорированы все файлы с расширением `.file`, кроме файла `name.file`|`/log.file`<br/>`/lastname.file`<br/><br/>**не будет проигнорирован:**<br/>`/name.file`|
## Как удалить ветку из локального и удаленного репозитория
- Чтобы удалить ветку из **локального** репозитория нужно выполнить `$ git branch -d <branch_name>`. Если ветка, которую вы хотите удалить еще не была смержена, то команду нужно выполнить с опцией `-D` : `$ git branch -D <bransh_name>`
- Чтобы удалить ветку из **удаленного** репозитория нужно выполнить `$ git branch origin --delete <branch_name>` , а затем `$ git fetch --all --prune`