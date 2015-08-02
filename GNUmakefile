needed_programs = s3cmd trickle curl

ifeq ($(shell which $(needed_programs) >/dev/null 2>&1 && echo yes),)
$(error "для работы нужны программы $(needed_programs)")
endif

ifeq ($(n),)
$(error "не определён номер диска в переменной n")
endif

# 180-й диск был «первым блином»: выложен в bucket с «неправильным» именем
# так как удалить bucket на archive.org «просто так» нельзя, «идём в обход»
ifeq ($(n),180)
item = lxfrudvd$(n)
else
item = rulxfdvd$(n)
endif

bucket = s3://$(item)

local_path = /home/www/dvd
files = $(local_path)/LXFDVD$(n)-1.iso $(local_path)/LXFDVD$(n)-2.iso

thumb = logo_LXF.jpg

creator = linuxformat.ru

ifeq ($(wildcard $(thumb)),)
$(error "файл $(thumb) не найден")
endif

# ограничение скорости для trickle
speed = 200

# извлекаем ключи из ~/.s3cfg для curl-а
accesskey = $(shell grep access_key ~/.s3cfg | sed 's/.*= *//')
secretkey = $(shell grep secret_key ~/.s3cfg | sed 's/.*= *//')

all: $(item).meta

%.put:
	@echo "делаю $@"
	s3cmd -c -v mb $(bucket); sleep 20
	trickle -s -u $(speed) s3cmd -v put $(files) $(bucket)
	touch $@

%.meta: %.put %.logo
	@echo "делаю $@"
	touch empty
	curl --location \
	--header 'x-archive-ignore-preexisting-bucket:1' \
	--header 'x-archive-meta01-collection:opensource' \
	--header 'x-archive-meta-mediatype:software' \
	--header 'x-archive-meta-creator:$(creator)' \
	--header 'authorization: LOW $(accesskey):$(secretkey)' \
	--upload-file empty \
	http://s3.us.archive.org/$(item)
	touch $@

%.logo: $(thumb)
	@echo "делаю $@"
	s3cmd put $< $(path)/$(item).jpg
	touch $@

.PRECIOUS: %.put %.logo

