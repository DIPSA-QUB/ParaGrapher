ifndef PARAGRAPHER_LIB_FOLDER
	PARAGRAPHER_LIB_FOLDER := $(shell realpath  ..)/lib64
endif 

GCC := gcc
GXX := g++
LIB := $(PARAGRAPHER_LIB_FOLDER):$(LD_LIBRARY_PATH)
SHELL := /bin/bash

UP_GCC_DIR := ~/gcc9.2
ifneq ("$(wildcard $(UP_GCC_DIR)/bin/gcc)","")
	GCC := $(UP_GCC_DIR)/bin/gcc
	GXX := $(UP_GCC_DIR)/bin/g++
	LIB := $(UP_GCC_DIR)/lib64:$(LIB)
endif

OBJ := obj
INCLUDE_LIBS := $(addprefix -L , $(subst :, ,$(LIB)))
INCLUDE_HEADER := $(addprefix -I , $(subst :,/../include ,$(LIB))) 
FLAGS :=  -Wfatal-errors -lm -fopenmp -lparagrapher

_threads_per_core := $(shell lscpu | grep "Thread(s) per core" | head -n1 | cut -f2 -d":"|xargs)
_total_threads := $(shell nproc --all) #getconf _NPROCESSORS_ONLN
_available_threads := $(shell nproc)
_total_cores := $(shell echo "$(_total_threads)/$(_threads_per_core)"|bc)
_available_cores := $(shell echo "$(_available_threads)/$(_threads_per_core)"|bc)
OMP_VARS :=  OMP_PLACES={0}:$(_total_cores):1 OMP_PROC_BIND=close OMP_DYNAMIC=false OMP_WAIT_POLICY=passive OMP_NUM_THREADS=$(_available_cores)

ifdef debug
	COMPILE_TYPE := -g
else
	COMPILE_TYPE := -O3 # -DNDEBUG
endif

all:
	@if [ `$(GCC) -dumpversion | cut -f1 -d.` -le 8 ]; then\
		$(GCC) -dumpversion; \
		echo -e "\033[0;33mError:\033[0;37m Version 9 or newer is required for gcc.\n\n";\
		exit -1;\
	fi 
	@echo -e "\n\033[1;34mRunning Tests\033[0;37m"
	@echo -e "#total_cores: "$(_total_cores)
	@echo -e "#available_cores: "$(_available_cores)
	@echo -e ""
	make $(subst .c,,$(shell ls *.c))

$(OBJ)/%.o: %.c Makefile ../include/*.h
	@echo -e "Compiling \033[1;33m"$<"\033[0;37m"
	mkdir -p $(OBJ)
	$(GCC) $< $(INCLUDE_LIBS) $(INCLUDE_HEADER) $(FLAGS) $(COMPILE_TYPE)  -o $(OBJ)/$*.o
	@echo -e ""

test%_WG400: $(OBJ)/test%_WG400.o
	@if [ $(dataset) == "" ]; then\
		dataset:="datasets/cnr-2000";\
	fi 
	@echo -e "Executing \033[1;33m"$@"\033[0;37m"	
	PARAGRAPHER_LIB_FOLDER=$(PARAGRAPHER_LIB_FOLDER) LD_LIBRARY_PATH=$(LIB) $(OMP_VARS) $(OBJ)/$@.o $(dataset)
	@echo -e ""

test%_WG404: $(OBJ)/test%_WG404.o
	@if [ $(dataset) == "" ]; then\
		dataset:="datasets/MS1-weights";\
	fi 
	@echo -e "Executing \033[1;33m"$@"\033[0;37m"	
	PARAGRAPHER_LIB_FOLDER=$(PARAGRAPHER_LIB_FOLDER) LD_LIBRARY_PATH=$(LIB) $(OMP_VARS) $(OBJ)/$@.o $(dataset)
	@echo -e ""

test%_WG800: $(OBJ)/test%_WG800.o
	@if [ $(dataset) == "" ]; then\
		dataset:="datasets/WDC-2012";\
	fi 
	@echo -e "Executing \033[1;33m"$@"\033[0;37m"	
	PARAGRAPHER_LIB_FOLDER=$(PARAGRAPHER_LIB_FOLDER) LD_LIBRARY_PATH=$(LIB) $(OMP_VARS) $(OBJ)/$@.o $(dataset)
	@echo -e ""

download_WG400:
	@if [ ! -f datasets/cnr-2000.graph ]; then \
		mkdir -p datasets; \
		echo -e "--------------------\n\033[1;34mDownloading cnr-2000\033[0;37m"; \
		wget -P datasets "http://data.law.di.unimi.it/webdata/cnr-2000/cnr-2000.graph"; \
		wget -P datasets "http://data.law.di.unimi.it/webdata/cnr-2000/cnr-2000.properties"; \
		echo -e "--------------------\n";\
	fi

download_WG404:
	@if [ ! -f datasets/MS1-weights.labels ]; then \
		mkdir -p datasets; \
		echo -e "--------------------\n\033[1;34mDownloading MS1\033[0;37m"; \
		wget -P datasets "http://78.46.92.120/MS-BioGraphs/MS1/MS1-underlying.graph"; \
		wget -P datasets "http://78.46.92.120/MS-BioGraphs/MS1/MS1-underlying.properties"; \
		wget -P datasets "http://78.46.92.120/MS-BioGraphs/MS1/MS1-weights.labels"; \
		wget -P datasets "http://78.46.92.120/MS-BioGraphs/MS1/MS1-weights.labeloffsets"; \
		wget -P datasets "http://78.46.92.120/MS-BioGraphs/MS1/MS1-weights.properties"; \
		echo -e "--------------------\n";\
	fi

download_WG800:
	@if [ ! -f datasets/WDC-2012.graph ]; then \
		mkdir -p datasets; \
		echo -e "--------------------\n\033[1;34mDownloading MS1\033[0;37m"; \
		wget "https://data.dws.informatik.uni-mannheim.de/hyperlinkgraph/2012-08/webgraph/network.graph" -O datasets/WDC-2012.graph; \
		wget "https://data.dws.informatik.uni-mannheim.de/hyperlinkgraph/2012-08/webgraph/network.properties" -O datasets/WDC-2012.properties ; \
		echo -e "--------------------\n";\
	fi
	

clean:
	rm -r $(OBJ)
	touch * 
	
touch:
	touch *

.SUFFIXES:

.keep_exec_files: $(addprefix $(OBJ)/,$(subst .c,.o, $(shell ls *.c)))
