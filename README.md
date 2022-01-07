# git tips and tricks
Кук-бук рецептов гит
## Как перенести коммиты из одной ветки в другую?
Предположим, что вы с энтузиазмом работаете над новой задачей, но недопитый кофе и рабочий чат отвлекли, новая ветка не была создана и вообще изменения уже закоммичены в master. Неплохо бы перенести их в новую ветку и продолжить работу там. Один из возможных алгоритмов такой:
```
  git branch new-feature
  git reset HEAD~ --hard
  git checkout new-feature
```
В случае если вы уже сделали больше одного коммита, то нужно во второй команде указать количество коммитов после тильды, которые нужно перенести (цифра после тильды указывает на сколько коммитов **передвинется назад** по истории указатель `HEAD`). Например, если нужно перенести три последних коммита, то команда будет выглядеть так `git reset HEAD~3 --hard`
## Как склеить коммиты?
Работаете над задачей и делаете много коммитов? Не хотите тащить всю эту историю с собой в основную ветку, а сделать один коммит? Тогда мы идем к вам!
### Способ №1. Squash
Внимание этот способ ***меняет историю***. Не используйте его, если вы уже отправили коммит в удаленный репозиторий и хотите остаться целым и симметричным. Испольуйте его когда изменения еще только в вашем локальном репозитории
### Способ №2. Soft reset
### Способ №3. Amend
Допустим вы сделали один коммит, внесли небольшое изменение и хотите склеить его с предыдущим коммитом, вместо того чтобы сделать новый. Используйте:
```git commit --ammend```. 

Вообще говоря, 
