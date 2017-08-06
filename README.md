Docker image based on

Ubuntu 16.04 reference

+

arm-linux-gnueabihf


# Usage example
1) cd to the folder where sources and makefile located
2) Trigger build using 'make'
docker run -v $(pwd):/work -it alfishe/intelsoc make

## Info
-v $(pwd):/work means that current folder will be injected into Docker container as /work folder (and Docker image file already set up to operate with /work folder after start)


alfishe/intelsoc - container image name on Docker Hub (or locally if you already have one with same name)


make - command executed already in container. Can be anything
You can start bash script from current folder inside container. That will be a command:
docker run -v $(pwd):/work -it alfishe/intelsoc bash -c <your_script>.sh


# Sample Makefile for plain C project
SHELL = /bin/bash -o pipefail
BASE = arm-linux-gnueabihf

CC = $(BASE)-gcc
LD = $(CC)
STRIP = $(BASE)-strip

PRJ = <ProjectName>
SRC = $(wildcard *.c)

OBJ = $(SRC:.c=.o)
DEP = $(SRC:.c=.d)

CFLAGS = $(DFLAGS) -c -std=gnu11 -O2 -D_FILE_OFFSET_BITS=64 -D_LARGEFILE64_SOURCE -DVDATE=\"date +"%y%m%d"\"
LFLAGS = -lc

$(PRJ): $(OBJ)
@$(info $@)
@$(LD) $(LFLAGS) -o $@ $+
@$(STRIP) $@

clean:
rm -f .d .o .elf .map .lst .bak .rej .org .user ~ $(PRJ)
rm -rf obj .vs DTAR* x64

%.o: %.c
@$(info $<)
@$(CC) $(CFLAGS) -o $@ -c $< 2>&1 | sed -e 's/(.[a-zA-Z]+):([0-9]+):([0-9]+):/\1(\2,\ \3):/g'

-include $(DEP)
%.d: %.c
@$(CC) $(DFLAGS) -MM $< -MT $@ -MT $*.o -MF $@ 2>&1 | sed -e 's/(.[a-zA-Z]+):([0-9]+):([0-9]+):/\1(\2,\ \3):/g'

main.o: $(filter-out main.o, $(OBJ))
