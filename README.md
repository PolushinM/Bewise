### Тестовое задание Bewise

#### 1. Извлекать реплики с приветствием – где менеджер поздоровался.
Если я правильно понял задачу, извлечение реплик приветствия выполняется для проверки требования о том, что менеджер должен обязательно здороваться и представляться.
Я полагаю, что перечень допустимых приветствий менеджера очень оганичен, и это должно быть записано в стандарте коммуникации в организации, чтобы менеджеры избегали недопустимых форм приветствия, кроме того, менеджеру следует представляться в первой или второй реплике разговора.
В таком случае, извлекать подобные реплики лучше всего при помощи регулярных выражений.

Допустимые приветствия (вариант): ["Здравствуйте", "Доброе утро", "Добрый день", "Добрый вечер"]

Получаем простую схему валидации приветствия:
1. Находим первую и вторую реплики менеджера.
2. В первой реплике менеджера ищем при помощи регулярных выражений допустимые приветствия.
3. Если допустимое приветствие присутствует (минимум одно), приветствие считаем валидным, в противном случае - нет.

#### 2. Извлекать реплики, где менеджер представил себя.
Я вижу 2 способа решить задачу:
1. Имя менеджера известно, известно, что он должен представиться в первой или второй реплике разговора. В таком случае, проще всего извлекать имя менеджера (или фразу "меня зовут Илья", например) при помощи регулярного выражения как в п. 1. Просто, надёжно, требует минимум вычислительных ресурсов, быстро можно разобраться с ошибками.
2. Предполагаем, что имя неизвестно, будем проверять первые 2 реплики, и пытаться извлечь из них имя (методом NER библиотеки natasha или моделью LaBSE_ner_nerel, например). В этом случае, сложно разбираться, чьё именно имя было названо: менеджера, клиента, другого сотрудника, другого клиента и пр. Поэтому первый вариант более предпочтительный.

#### 3. Извлекать имя менеджера.
Также есть 2 способа:
1. У нас есть список имен менеджеров, находим при помощи регулярного выражения имя одного из менеджеров. Думаю, это предпочтительный вариант с точки зрения надежности и производительности.
2. Набора имён у нас нет, используем инструмент для NER из библиотеки natasha или модель LaBSE_ner_nerel, в паре с truecaser работает неплохо и не требует списка имен.\
Проблемой является то, что имя написано не с заглавной буквы, это осложняет работу NER. Кроме того, в реплике может прозвучать несколько имен, наиболее вероятно, что менеджер назовет свое имя в начале, поэтому, возьмем первое найденное имя.
Ситуацию с заглавными буквами может поправить truecaser.

#### 4. Извлекать название компании.
Задача NER.
Большая проблема, что в текстах нет знаков препинания и заглавных букв, а инструменты для ивлечения именованных сущностей (Natasha, Spacy и пр.) их используют, причем, используют как самые значимые признаки.
Задача выделения наименования компании в тексте, приведенном к нижнему регистру, с отсутствующими знаками препинания, сложная сама по себе, для её решения необходимо достаточно точно учитывать контекст слов, сейчас это, чаще всего, делается при помощи BERT.
Так как готовые инструменты на подобных текстах не работают, придется использовать регулярные выражения, но доля ошибок такого решения будет велика.
Остается 2 варианта:
1. Регулярные выражения: считать наименованием компании слово, идущее после слов "ООО", "компания" и пр. Работает плохо.
2. Использовать truecaser для нормализации регистра, потом подавать строку в natasha(slovnet) NER. Работает ещё хуже.
3. Использовать модель LaBSE_ner_nerel, дообученную на текстах без пунктуации и регистра. Работает намного лучше.

#### 5. Извлекать реплики, где менеджер попрощался.
Задача похожа на первую. Решаем аналогично. Составляем список допустимых реплик: ["До свидания", "Всего доброго", "Приятного вечера"], проверяем последние 2 реплики менеджера на их наличие.


#### 6. Проверять требование к менеджеру: «В каждом диалоге обязательно необходимо поздороваться и попрощаться с клиентом».
Проверяем 2 условия, группируем датафрейм по диалогам, добавляем в датафрейм колонку "greetings and farewell", где указываем выполнение условий (True/False).

#### Использование машинного обучения.
Датасет очень маленький (всего 5 телефонных звонков), разметки, которая нас интересует нет. Обучать модели на этом датасете нет смысла, но если собрать больше данных - можно пробовать решать задачи с применением ML.

### Результат работы:
1. Решение с применением регулярных выражений, truecaser и natasha (см. каталог No_BERT).
Для извлечения имен менеджеров сделаны 2 альтернативных метода: извлечение имен менеджеров из списка и извлечение всех имен при помощи truecaser + natasha.
Для извлечения наименований компаний аналогично: извлечение наименований известных компаний из списка и извлечение всех организаций помощи truecaser + natasha.

2. Решение с применением регулярных выражений и BERT (см. каталог With_BERT).
Для извлечения имен менеджеров сделаны 2 альтернативных метода: извлечение имен менеджеров из списка и извлечение всех имен при помощи LaBSE_ner_nerel.
Для извлечения наименований компаний аналогично: извлечение наименований известных компаний из списка и извлечение всех организаций помощи LaBSE_ner_nerel.
Для того чтобы модель могла извлекать имена из текста без пунктуации и в нижнем регистре, выполнено дообучене модели LaBSE_ner_nerel на одной эпохе с lr=9e-5 и одной эпохе с lr=1e-5 на датасете "Nerel short" без знаков препинания, в lowercase режиме. Полученные результаты - выше, чем при использовании регулярных выражений, но производительность значительно ниже (на несколько порядков).\
В каталоге присутствует также файл training.ipynb, где показан процесс файнтюнинга модели.\
Что можно попробовать ещё:\
1. Использовать Roberta. BERT не очень хорошо работает с совсем незнакомыми токенами, какими часто являются наименования компаний, roberta работает с чарграммами, и если распределение чарграмм в наименованиях компаний отличается от распределения чарграмм в остальной части текста, это может значительно увеличить точность.
2. Попробовать повысить производительность: использовать небольшую и высокопроизводительную модель BERT, обучить её можно либо с нуля на большом датасете, либо путем дистилляции большой и точной модели.\

Модель LaBSE_ner_nerel занимает около 500Мб, поэтому её не удалось рагрузить на github. Архив со всеми данными:
По вопросам: https://t.me/polushinmc