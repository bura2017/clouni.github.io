# clouni.github.io
******************

## Adding new/updating old pages

1. If you update old pages, go to 3 step
1. Add <FILENAME> to the  *$(CLOUNI_DOCS)/source/\<lang\>/index.rst* in order contents
2. Create file *$(CLOUNI_DOCS)/source/\<lang\>/\<FILENAME\>.rst* or *$(CLOUNI_DOCS)/source/\<lang\>/\<FILENAME\>.md*
3. Update *$(CLOUNI_DOCS)/source/\<lang\>/\<FILENAME\>.rst* or *$(CLOUNI_DOCS)/source/\<lang\>/\<FILENAME\>.md*
4.  
    ~~~shell
    cd $(CLOUNI_DOCS)/source/ru
    make dirhtml
    cd ../en
    make dirhtml
    ~~~
5. Check your site looks as expected
6. Commit and push
7. You're gorgeous!
