AR        := ar
CC        := gcc
INCLUDES  := -I. -Icli 
OPTIONS   := -Wno-macro-redefined -Wno-shift-negative-value -Os


ifeq ($(OS),Windows_NT)
	# Windows
else
	UNAME_S := $(shell uname -s)
	ifeq ($(UNAME_S),Darwin)
		# MacOS
		LDFLAGS = -lpthread -ltls
		ifeq ($(TLS_STATIC), 1)
			LIBFLAGS = -L./SSL/macos_fat
		else
			LIBFLAGS = -L/opt/homebrew/opt/libressl/lib
		endif
	else
		# Linux
		LDFLAGS = -lpthread -ltls
		ifeq ($(TLS_STATIC), 1)
			LIBFLAGS = -L./SSL/linux_64bit
		else
			LIBFLAGS = -L/usr/local/libressl/lib/ -Wl,-rpath=/usr/local/libressl/lib/
		endif
	endif
endif

lz4.o:	lz4.c lz4.h
	$(CC) $(OPTIONS) $(INCLUDES) lz4.c -c -o lz4.o
	
sqcloud.o:	sqcloud.c sqcloud.h
	$(CC) -fPIC -shared *.o -o libsqcloud.so  ${LIBFLAGS} ${LDFLAGS}

libsqcloud.a: lz4.o sqcloud.o
	$(AR) rcs libsqcloud.a *.o

libsqcloud.so: lz4.o sqcloud.o
	$(CC) -fPIC -shared *.o -o libsqcloud.so
	
libsqcloud.dylib: lz4.o sqcloud.o
	$(CC) -dynamiclib -install_name -flat_namespace *.o -o libsqcloud.dylib

all:	lz4.o

cli: libsqcloud.a cli/linenoise.c cli/linenoise.h cli/main.c
	$(CC) $(OPTIONS) $(INCLUDES) cli/*.c libsqcloud.a -o sqlitecloud-cli ${LIBFLAGS} ${LDFLAGS}

clean:
	rm -rf *.o *.a *.so *.dylib
	
test: cli
	./sqlitecloud-cli -h localhost
