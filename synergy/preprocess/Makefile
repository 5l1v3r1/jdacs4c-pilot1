OBJS = parse_combo.o parse_csv.o io.o dose_response_function.o

CC = g++
cc = gcc

PROFILE = #-pg -g
OPENMP = -fopenmp

FLAGS = $(PROFILE) -O3 -Wall $(OPENMP) -std=c++0x

GSL_LIB_DIR = 
GSL_INCLUDE_DIR = 

INC = -I. -I$(GSL_INCLUDE_DIR)
LIBS = -lm -lz $(GSL_LIB_DIR)/libgsl.a $(GSL_LIB_DIR)/libgslcblas.a 

.SUFFIXES : .o .cpp .c
.cpp.o:
	$(CC) $(FLAGS) $(PROFILE) $(INC) -c $<
.c.o:
	$(cc) $(FLAGS) $(PROFILE) $(INC) -c $<

all: gemini_prep synergy_search

gemini_prep : $(OBJS) gemini_prep.o
	$(CC) $(PROFILE) -o gemini_prep gemini_prep.o $(OBJS) $(LIBS) $(OPENMP)


synergy_search : $(OBJS) synergy_search.o
	$(CC) $(PROFILE) -o synergy_search synergy_search.o $(OBJS) $(LIBS) $(OPENMP)
	
clean:
	-rm -f *.o


