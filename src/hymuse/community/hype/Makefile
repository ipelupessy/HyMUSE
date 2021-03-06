# standard amuse configuration include
# config.mk will be made after ./configure has run
AMUSE_DIR?=../../../..
-include ${AMUSE_DIR}/config.mk

MPICXX ?= mpicxx
MPICC ?= mpicc

CODE_GENERATOR = $(AMUSE_DIR)/build.py

BUILDDIR=build

CMAKE_DEPS=CMakeLists.txt ./bmi/CMakeLists.txt ./heat/CMakeLists.txt

BMICODE=hype
INTERFACE=HypeInterface

WORKER=bmi_$(BMICODE)_worker

BMIINC=-I. -I/home/inti/code/eWaterCycle/prerequisites/include 
BMILIBDIR=-L$(BUILDDIR)/bmi/ -L$(BUILDDIR)/heat/
BMILIB=-ldl 

CPPFLAGS+=$(BMIINC)

LIBS+=-lgfortran

all:  $(WORKER)

.FORCE:

$(BUILDDIR): hype-bmi
	mkdir -p $(BUILDDIR)
	cp hype-bmi/src/*  $(BUILDDIR)

clean:
	rm -f *.pyc worker_code.cc worker_code.h interface.o interface.cc *.mod
	make -C $(BUILDDIR) clean

distclean: 
	rm -f *.pyc worker_code.cc worker_code.h interface.o interface.cc *.mod
	rm -rf $(BUILDDIR) $(WORKER) hype-bmi

$(BUILDDIR)/hype_bmi.o: $(BUILDDIR)  
	make -C $(BUILDDIR) CPPFLAGS="$(CPPFLAGS)" hype hypelib.o hype_bmi.o

$(WORKER): OBJS=$(filter-out $(BUILDDIR)/hypecode/main.o, $(wildcard $(BUILDDIR)/hypecode/*.o))
$(WORKER): $(BUILDDIR)/hype_bmi.o worker_code.cc worker_code.h interface.o
	$(MPICXX) -I$(BUILDDIR) $(CXXFLAGS) $(LDFLAGS) $(BUILDDIR)/hype_bmi.o $(BUILDDIR)/hypelib.o $(OBJS) worker_code.cc interface.o  -o $@ $(BMILIBDIR) $(BMILIB) $(LIBS)

worker_code.cc: interface.py
	$(CODE_GENERATOR) --type=c interface.py $(INTERFACE) -o $@

worker_code.h: interface.py
	$(CODE_GENERATOR) --type=h interface.py $(INTERFACE) -o $@

hype-bmi:
	git clone https://github.com/eWaterCycle/hype-bmi

%.o: %.cc
	$(MPICXX) $(CFLAGS) $(BMIINC) -c -o $@ $< 

interface.cc:
	$(PYTHON) -c "import interface; interface.generate_interface_file()"
