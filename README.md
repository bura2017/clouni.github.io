# clouni.github.io


## Добавление новых/изменение старых страниц

0. Если изменяем старый файл, goto 3
1. В файл *$(CLOUNI_DOCS)/source/<lang>/index.rst* добавляем <FILENAME> в нужное место списка страниц
2. Создаём файл *$(CLOUNI_DOCS)/source/<lang>/<FILENAME>.rst* или *$(CLOUNI_DOCS)/source/<lang>/<FILENAME>.md*
3. Изменяем *$(CLOUNI_DOCS)/source/<lang>/<FILENAME>.rst* или *$(CLOUNI_DOCS)/source/<lang>/<FILENAME>.md*
4.  
    ~~~
    cd $(CLOUNI)/source/ru
    make dirhtml
    cd ../en
    make dirhtml
    ~~~
5. Проверяем правильность сайта
6. Коммитим и пушим    
7. Вы великолепны!
